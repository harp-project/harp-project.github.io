---
layout: default
title: Semantics
description: Formal semantics for Core Erlang
permalink: /semantics/
---

# A short introduction of Core Erlang

Core Erlang is an intermediate language in the compilation pipeline of Erlang (and of other popular languages, such as Elixir and Gleam). It is the target of numerous optimization steps of the compiler (such as [constant folding](https://www.erlang.org/blog/core-erlang-optimizations/), or dead code elimination). Consider the following simple function computing the factorial of a number:

```erlang
fact(N) when N =:= 0 -> 1;
fact(N) -> N * fact(N - 1).
```

Compiling this function to Core Erlang with the options `[to_core, no_copt]` results in the following code:

```
'fact'/1 =
    %% Line 4
    ( fun (_0) ->
	  ( case ( _0 -| [{'function',{'fact',1}}] ) of
	      <N> when call 'erlang':'=:='(N, 0) -> 1
	      %% Line 5
	      <N> when 'true' ->
		  let <_1> = call 'erlang':'-'(N, 1)
		  in let <_2> = apply 'fact'/1(_1)
		     in call 'erlang':'*'(N, _2)
	      ( <_3> when 'true' -> ( primop 'match_fail'
			  (( {'function_clause',_3}
			     -| [{'function',{'fact',1}}] ))
		      -| [{'function',{'fact',1}}] )
		-| ['compiler_generated'] )
	    end
	    -| [{'function',{'fact',1}}] )
      -| [{'function',{'fact',1}}] )
```

While with only `[to_core]`, we get the following, optimized variant (which does not contain the redundant 3rd clause of the `case` expression):

```
'fact'/1 =
    %% Line 4
    ( fun (_0) ->
	  ( case ( _0 -| [{'function',{'fact',1}}] ) of
	      <N> when call 'erlang':'=:='(( _0 -| [{'function',{'fact',1}}]), 0) -> 1
	      %% Line 5
	      <N> when 'true' ->
		  let <_1> =
		      call 'erlang':'-'(N, 1)
		  in  let <_2> = apply 'fact'/1(_1)
		      in  call 'erlang':'*'(N, _2)
	    end
	    -| [{'function',{'fact',1}}] )
      -| [{'function',{'fact',1}}] )
```

Even from these code examples, a number of notable features of Core Erlang appear that do not exist, or behave in a different way than in Erlang. Just to mention a few features (visible in the code above):

- There is no multiclause function definition. Multiclause Erlang functions get translated to explicit `case` expressions with guards.
- `case` expression clauses always have explicit guards. If the guard evaluates to an exception, then the entire case expression evaluates to that exception.
- There are multiple ways to apply functions: `call` is used for inter-module function application (with explicit module and function name arguments), while `apply` is reserved for local functions.
- `let` expressions are used to assign names to subexpressions; however, `let` expressions do not contain pattern matching (in contrast with match expressions of Erlang).
- The compiled code is annotated with `-|` containing compilation information.

Furthermore, numerous other Erlang language features get translated to their more primitive Core Erlang counterpart (or a combination of more primitive Core Erlang expressions). For example, `receive` expressions are expressed with primitive operations and recursion, list comprehensions are unfolded as an explicit recursive function, and `if` expressions get translated to `case` expressions. This means that Core Erlang has far fewer language elements than Erlang, and therefore, it is simpler to define its semantics, and simpler to reason about program behavior in general. On the other hand, Core Erlang is more verbose and significantly less readable than Erlang---however, this was not its purpose.

For the reasons above, defining a semantics of Core Erlang is more advantageous than targeting Erlang, Elixir or Gleam: 1) we can exploit trusted translation of Erlang, Elixir and Gleam to Core Erlang to indirectly reason about programs written in the original language, 2) the language of Core Erlang is much smaller and simpler to define formally than the higher-level languages.

# What is a formal semantics? What is a mechanized semantics?

A *formal semantics* is a mathematically precise description of program behavior: it defines exactly what program execution does, using mathematical objects and rules rather than prose in a manual or the behavior of a particular compiler. Because the definition is unambiguous, it can settle questions informal descriptions leave open, such as whether two programs always behave the same or whether an optimization preserves meaning.

A *mechanized semantics* is a formal semantics encoded in a proof assistant (in our case, the [Rocq prover](https://rocq-prover.org/)) rather than existing only on paper. The syntax, the evaluation rules, and every proof about them---for instance that evaluation is deterministic, or that a transformation preserves the meaning---are checked by the machine down to primitive inference steps. Mechanization rules out the gaps and unstated assumptions that hand-written proofs often hide, and it gives us a solid foundation for larger verified work. Furthermore---since Rocq is essentially a functional programming language---definitions and functions are executable, and therefore, a function-based semantics can also serve as an [interpreter for the programming language](https://github.com/harp-project/Core-Erlang-Formalization/tree/master/src/Interpreter).

# The syntax and formal semantics of Core Erlang

In this section, we discuss what features of Core Erlang have been implemented in our formal semantics, and how they behave. Core Erlang has an [official language specification](https://www.diva-portal.org/smash/record.jsf?pid=diva2%3A1695554&dswid=7248) written in 2000 which is now outdated. Therefore, our semantics has been based not only on this specification, but also on other related research, online materials (some are highlighted under "Further reading"), and the official implementation of the Erlang compiler. Here, we summarize the syntax and behavior of Core Erlang informally; however, everything presented here is expressed in mathematics and logic, and also implemented in the [Rocq prover](https://rocq-prover.org/).

## Sequential features

We will use `v` (and its indexed variants) to denote values, `e` for expressions, and `p` for patterns. 

### Values (denoted with `v`)

Values represent the result of computation, and not all values can be explicitly written in as Core Erlang code. For example, closures cannot be constructed directly, but they are the result of evaluating a function expression.

- Literals: `[]` denotes the empty list, `i` is used for integers, `a` is used for atoms. Concrete atoms are always enclosed with apostrophes `'` in Core Erlang.
- Lists (constructed as `[v_1|v_2]`) consist of a head and a tail value.
- Tuples (constructed as `{v_1, v_2, ..., v_n}`) consist of a number of values, and are enclosed in curly braces.
- Maps (constructed as `~{v_1 => v_1', v_2 => v_2', ..., v_n => v_n'}~`) are essentially tilde-enclosed tuples containing key-value pairs. A map value cannot contain duplicate keys.
- Variables: we use `X, Y, Z, ...` to denote variables (`_0`, `_1`, `_3` are also variables in the code above). Variables are special because they are not valid final results: they get their meaning by substituting them with another value.
- Function identifiers (constructed as `a/i`, with an atom and an integer) are similar to variables, their meaning is defined by substituting them with a function expression.
- Function closures do not have an explicit syntax. They denote the values of function expressions, and contain the necessary data to apply them: the formal parameter list, body expression, and the list of simultaneously defined functions (which can be done with a `letrec` expression).

### Patterns (denoted with `p`) and pattern matching

Patterns have the same structure as values (except for closures and function identifiers). They can only be used in `case` expressions (cf. Erlang) to match against values. A pattern matching is either successful or unsuccessful. A successful match contains (sub)values for the variables occurring in the pattern.

- Literal patterns only match the same literal values (e.g., `1` matches `1`, but not `[]`).
- Lists patterns (`[p_1|p_2]`) only match list values (`[v_1|v_2]`), and only if `p_1` matches `v_1`, and `p_2` matches `v_2`.
- Tuple patterns (`{p_1, p_2, ..., p_n}`) only match tuple values (`{v_1, v_2, ..., v_n}`), and only if their components pairwise match (i.e. for all `i`, `p_i` matches `v_i`).
- Map patterns (`{p_1 := p_1', p_2 := p_2', ..., p_n := p_n'}`) only match map values (`{v_1 => v_1', v_2 => v_2', ..., v_n => v_n'}`), and only if their components pairwise match.
- Variable patterns match any value, and this value will be bound in the corresponding clause (guard and body expressions) of the enclosing `case` expression. Nonlinear patterns are not allowed in Core Erlang (cf. Erlang), which means that including a bound variable in a pattern shadows the outer occurrence. For example, the following expression will evaluate to `2` without raising an exception.

   ```
   let X = 1 in
     case 2 of
	   X when 'true' -> X
     end
   ```

### Expressions (denoted with `e`) and their evaluation

An expression of Core Erlang evaluates either to a *value sequence* or to an exception. An exception is a triple of values (an exception class, exception reason, and further details about the error). Whenever an exception is raised, the evaluation stops, and the exception gets propagated, unless it is handled by a `try` expression.

A value sequence is essentially a list of values (denoted as `<v_1, v_2, ..., v_n>`). Most Core Erlang expressions evaluate to a singleton value sequence (i.e. a single value), and non-singleton value sequences can only be used in binding expressions (i.e. in `let`, `try`, and `case`) to express simultaneous bindings. The only expression that constructs a non-singleton value sequence is the value list (denoted as `<e_1, e_2, ..., e_n>`).

<style>
  details > summary { cursor: pointer; font-weight: 500; list-style: none; margin-bottom: 0.6em; }
  details > summary::-webkit-details-marker { display: none; }
  details > summary::before { content: "▶"; display: inline-block; margin-right: 0.4em; transition: transform 0.15s ease; }
  details[open] > summary::before { transform: rotate(90deg); }
  details > summary:hover { text-decoration: underline; }
</style>

Values are the simplest expressions, and they evaluate to themselves enclosed in a singleton value sequence.

**Value lists** are constructed as

  ```
  <e_1, e_2, ..., e_n>
  ```

  <details markdown="1">
  <summary>Evaluation</summary>

  Value lists evaluate in a left-to-right order. It is expected that all subexpressions evaluate to a singleton value sequence (i.e. `e_i` evaluates to `<v_i>`), and then the result is a value sequence containing values from these singletons (i.e. `<v_1, v_2, ..., v_n>`).

  </details>

**Functions** are constructed as

  ```
  fun(X_1, X_2, ..., X_n) -> e
  ```

  <details markdown="1">
  <summary>Evaluation</summary>

  A function expression evaluates to a closure which contains the formal parameter list and the body expression.

  </details>

**Lists** are constructed as 
  
  ```
  [e_1|e_2]
  ```

  <details markdown="1">
  <summary>Evaluation</summary>

  Lists evaluate in a right-to-left order (cf. Erlang), so first `e_2` is evaluated to a singleton value sequence (`<v_1>`), then `e_1` (to `<v_2>`), and the result `[v_1|v_2]` is constructed from the values of these singleton sequences.

  </details>

**Tuples** are constructed as

  ```
  {e_1, e_2, ..., e_n}
  ```

  <details markdown="1">
  <summary>Evaluation</summary>

  Tuples are evaluated in a left-to-right order. All subexpressions are expected to evaluate to singleton value sequences---the result tuple is constructed from the values of these singletons.

  </details>

**Maps** are constructed as
  
  ```
  ~{e_1 => e_1', e_2 => e_2', ..., e_n => e_n'}~
  ```

  <details markdown="1">
  <summary>Evaluation</summary>

  Maps are evaluated in a left-to-right order (i.e. first key, first value, second key, second value, etc.). All subexpressions are expected to evaluate to singleton value sequences, and the result map is constructed from the values of these sequences, after duplicate keys are eliminated.

  </details>

**Inter-module call** expressions are constructed as
  
  ```
  call e_1:e_2(e_3, e_4, ..., e_n)
  ```

  <details markdown="1">
  <summary>Evaluation</summary>

  The subexpressions of inter-module calls are evaluated in a left-to-right order. All subexpressions are expected to evaluate to singleton value sequences (i.e., `<v_1>`, `<v_2>`, `<v_3>`, ...), furthermore, `e_1` and `e_2` should evaluate to atoms (i.e. `v_1 = a_1`, and `v_2 = a_2`), which specify the module and function name used in the call. The semantics then simulates the function called `a_2` in module `a_1` with the actual arguments `v_3, ..., v_n`.

  </details>

**Primitive operations** are constructed as

  ```
  primop a(e_1, e_2, ..., e_n)
  ```

  <details markdown="1">
  <summary>Evaluation</summary>

  Note that the name of primitive operations is expected to be an atom (it is not an expression). The subexpressions of primitive operations are evaluated in a left-to-right order. All subexpressions are expected to evaluate to singleton value sequences (i.e., `<v_1>`, `<v_2>`, ...). The semantics then simulates the primitive operation called `a` with the actual arguments `v_1, v_2, ...`. If the function expression does not evaluate to a closure, or the argument counts mismatch, then an exception is raised.

  </details>

**Function applications** are constructed as
  
  ```
  apply e_1(e_2, e_3, ..., e_n)
  ```

  <details markdown="1">
  <summary>Evaluation</summary>

  The subexpressions of applications are evaluated in a left-to-right order. All subexpressions are expected to evaluate to singleton value sequences (i.e., `<v_1>`, `<v_2>`, ...). The function expression (`e_1`) is expected to evaluate to a closure (i.e. `v_1` is a closure), which is applied to the actual parameters. This means that the closure's body is evaluated next, after substituting the formal parameters by `v_2, ..., v_n`. If `v_1` is not a closure, or the argument numbers mismatch, an exception is raised.

  </details>

**Pattern matching** expressions are constructed as
  
  ```
  case e of
    <p_1_1, p_1_2, ..., p_1_n> when e_1_1 -> e_1_2
    <p_2_1, p_2_2, ..., p_2_n> when e_2_1 -> e_2_2
    ...
    <p_m_1, p_m_2, ..., p_m_n> when e_m_1 -> e_m_2
  end
  ```

  <details markdown="1">
  <summary>Evaluation</summary>

  Note that pattern matching is only done with `case` in Core Erlang (`try` does not include patterns, [`receive` is syntax sugar](https://www.erlang.org/eeps/eep-0052), and there is no match operator). First `e` is evaluated to a value sequence `<v_1, v_2, ..., v_n>`. Each clause is expected to contain exactly `n` patterns, which are pairwise matched against the elements of the value sequence. If all patterns match the values of the sequence, the variables occurring in the patterns will be bound to subvalues, and these bindings are substituted in the guard (after the keyword `when`) and body expressions (after `->`) of the current clause. The guard is expected to evaluate to either `<true>` (i.e. it succeeds), `<false>` (i.e. it fails), or an exception, which is propagated (cf. Erlang). If the guard succeeds, the body expression of the current clause is evaluated. If the guard fails, or the patterns of the clause do not match, the next clause is tried. If there are no more clauses, an exception is raised.

  </details>

**Let bindings** are constructed as
  
  ```
  let <X_1, X_2, ..., X_n> = e_1 in e_2
  ```

  <details markdown="1">
  <summary>Evaluation</summary>

  First, `e_1` is evaluated to a value sequence of length `n` (which is exactly the number of variables to be bound). Then `e_2` is evaluated after substituting the variables by the values of the sequence. If the value sequence has a different length, the behavior is not defined.

  </details>

**Letrec bindings** are constructed as

  ```
  letrec
    a_1/i_1 = fun(X_1_1, X_1_2, ..., X_1_n) -> e_1
    a_2/i_2 = fun(X_2_1, X_2_2, ..., X_2_n) -> e_2
    ...
    a_m/i_m = fun(X_m_1, X_m_2, ..., X_m_n) -> e_m
  in e
  ```

  <details markdown="1">
  <summary>Evaluation</summary>

  A `letrec` expression defines recursive functions. First, closures are constructed for each function, which are substituted for their respective function identifiers in `e`. Note that in this case, the closures also contain a list of all functions defined simultaneously in the same `letrec`---these functions can be mutually recursive.

  </details>

**Sequencing** is expressed as
  
  ```
  do e_1 e_2
  ```

  <details markdown="1">
  <summary>Evaluation</summary>

  Sequencing behaves the same way as a `let` expression with a single, unused variable. First `e_1` is evaluated to a singleton value sequence, then this result is discarded, and the evaluation continues with `e_2`.

  </details>

**Exception handler expressions** are constructed as
  
  ```
  try e_1 of
    <X_1, X_2, ..., X_n> -> e_2
	catch <X_1', X_2', ..., X_m'> -> e_3
  ```

  <details markdown="1">
  <summary>Evaluation</summary>

  First, `e_1` is evaluated to a value sequence of length `n` (if the length is different, the behavior is not defined). Then, `e_2` is evaluated after substituting the variables `X_1, X_2, ..., X_n` by the values of the sequence. However, if `e_1` raises an exception, it is handled by `catch`. In practice, there are three variables in the `catch` clause that are bound to the three parts of the exception, and `e_3` is evaluated after substituting these variables.

  </details>

The informal description provided here is also expressed in Rocq with a *frame-stack semantics* (a relation that expresses step-by-step evaluation, and its reflexive, transitive closure).

### What's not formalized from the complete Core Erlang syntax?

- References (however, we already have a proof-of-concept outlined in [this paper](https://doi.org/10.1145/3830434.3830944))
- Floating point numbers
- Binaries, bitstrings ([they are work-in-progress](https://github.com/harp-project/Core-Erlang-Formalization/tree/binaries))
- The module system and non-standard library inter-module calls (supported functions are implemented [here, in the formalization](https://github.com/harp-project/Core-Erlang-Formalization/blob/master/src/Auxiliaries.v))

## Concurrent features

Our semantics of Core Erlang also formalizes a subset of Erlang's actor model. Here, we enumerate the concurrent features we support.

- **PIDs** are used to identify processes in an Erlang node. PIDs are values, and therefore, they can be used at any position where a value can be used (even in the sequential setup). Besides PIDs, there is no other new value or expression that are introduced for the concurrent semantics, because the remaining concurrency features are expressed with primitive operations (`primop`) and inter-module function calls (`call`).
- **Primitive operations for message receipts.** Erlang's `receive` expression is expressed as syntax sugar with recursion, pattern matching, and primitive operations. The desugaring uses four primops that operate on the process mailbox and its mailbox pointer:
  - `recv_peek_message` reads the message currently under the mailbox pointer, returning `<'true', Msg>`, or `<'false', v>` (the exact value `v` is unspecified) when there is none---i.e. all messages are already checked.
  - `recv_next` advances the mailbox pointer to the next message.
  - `remove_message` deletes the message under the mailbox pointer, and resets the pointer to the front of the mailbox.
  - `recv_wait_timeout` handles the `after` part of Erlang's `receive`, returning `'true'` when the given timeout expires and `'false'` when a new message arrives before then. Currently in the semantics only `0` and `'infinity'` timeouts are expressed.
- **Inter-module calls the express concurrent behavior.** The following functions of the `erlang` module are formalized:
  - `erlang:!/2` sends a value to the mailbox of the process identified by the given PID.
  - `erlang:self/0` returns the PID of the evaluating process.
  - `erlang:spawn/2` creates a new process that applies the given closure to the given argument list, and returns its PID. Currently, this is not exactly modeled, since `spawn/2` of the standard library works with a parameterless function.
  - `erlang:spawn_link/2` behaves like `erlang:spawn/2`, but additionally establishes a link between the spawning and the spawned process.
  - `erlang:link/1` and `erlang:unlink/1` create and remove a link between the evaluating process and the process given by its PID.
  - `erlang:exit/2` sends an exit signal carrying the given reason to the process given by its PID.
  - `erlang:process_flag/2` with the `'trap_exit'` flag controls whether incoming exit signals are converted into messages instead of terminating the process.

# Mechanization of the semantics of Core Erlang

The entire description above is mechanized in the [Rocq prover](https://rocq-prover.org/), in the [Core-Erlang-Formalization](https://github.com/harp-project/Core-Erlang-Formalization) repository. The abstract syntax is represented in a nameless style: variables and function identifiers are de Bruijn indices (`Var := nat`), and well-formedness is captured by inductive scoping predicates that track how many binders are in scope. The sequential language is given a substitution-based frame-stack semantics: a small-step relation that uses explicit continuation frames, together with its reflexive-transitive closure describing complete evaluations. The concurrent layer extends this with a process-local relation (message receipts, spawning, links, exit signals for a single process) and an inter-process relation that routes signals between processes running on a node.

Alongside the relational semantics, the development provides an [executable interpreter](https://github.com/harp-project/Core-Erlang-Formalization/tree/master/src/Interpreter), written as a Rocq function, that is proved equivalent to the frame-stack semantics. Being a plain function, it can be run inside Rocq and extracted to OCaml and Haskell. Furthermore, we also applied the semantics in several areas, the most important of which is program equivalence.

# Program equivalence in Core Erlang

Program equivalence is used to express when two programs behave the same way, in all possible circumstances. Program equivalence proofs can be used to verify that refactorings or optimizations preserve the behavior, the latter of which is crucial for proving compiler correctness.

For the sequential semantics, we express program equivalence in terms of contextual equivalence (i.e. two equivalent programs should behave the same way in every syntactical context). However, proving contextual equivalence of concrete programs is challenging, and therefore, we also defined a simpler concept (CIU-equivalence) which coincides with contextual equivalence. As an example, we partially verified the clause for `do` expressions [of the constant folding optimization in the Erlang compiler](https://github.com/erlang/otp/blob/97dc0d2d190cc3710e870fed604bdae144227bd4/lib/compiler/src/sys_core_fold.erl#L268). This means that Core Erlang programs before and after the transformation show the same behavior.

<figure style="text-align: center; margin: 1.5em 0;">
  <img src="{{ '/assets/optim.png' | relative_url }}" alt="The c_seq clause of the constant-folding pass in sys_core_fold.erl, with part of the code covered by a dashed box labeled 'Not formalized yet'." style="max-width: 34rem; width: 100%; height: auto;">
  <figcaption style="font-size: 0.9em;">The <code>c_seq</code> (sequencing) clause of the constant-folding pass in <a href="https://github.com/erlang/otp/blob/97dc0d2d190cc3710e870fed604bdae144227bd4/lib/compiler/src/sys_core_fold.erl#L268"><code>sys_core_fold.erl</code></a>. The dashed box marks the sub-case that our equivalence proof does not yet cover.</figcaption>
</figure>

In the concurrent setup, we rely on barbed bisimulation to prove program equivalence. With these concepts we managed to prove that a sequential and a concurrent variant of list mapping is equivalent (if no exit signals arrive from other processes). This equivalence holds for all function expressions <span class="fv">f</span> (with some restrictions), proper lists <span class="fv">el</span>, observed PID <span class="fv">i</span>, and natural numbers <span class="fv">n</span>. (Note that in Core Erlang, [`receive` is syntax sugar](https://www.erlang.org/eeps/eep-0052).)

<style>
  .cerl-code { background-color: #f3f6fa; border-radius: 0.3rem; padding: 0.8rem; overflow-x: auto; font-size: 0.85em; line-height: 1.45; }
  .fv { color: #c92a2a; font-weight: 700; }
</style>

<div style="display: flex; flex-wrap: wrap; gap: 1.5em; align-items: flex-start;">
<div style="flex: 1 1 20rem; min-width: 0;">

<p><em>Sequential variant</em></p>

<pre class="cerl-code"><code>letrec 'map'/2 = fun(F,L) -&gt;
  case L of
    [] when 'true' -&gt; []
    [H|T] when 'true' -&gt;
      [apply F(H)|apply 'map'/2(F,T)]
  end
in
  call 'erlang':'!'(<span class="fv">i</span>,apply 'map'/2(<span class="fv">f</span>,<span class="fv">el</span>))</code></pre>

</div>
<div style="flex: 1 1 20rem; min-width: 0;">

<p><em>Concurrent variant</em></p>

<pre class="cerl-code"><code>letrec 'map'/2 = fun(F,L) -&gt;
  case L of
    [] when 'true' -&gt; []
    [H|T] when 'true' -&gt;
      [apply F(H)|apply 'map'/2(F,T)]
  end
in
case call 'lists':'split'(<span class="fv">n</span>,<span class="fv">el</span>) of
 {L1, L2} when 'true' -&gt;
  let S = call 'erlang':'self'() in
   do call 'erlang':'spawn'(fun() -&gt;
        call 'erlang':'!'(S,
                    apply 'map'/2(<span class="fv">f</span>, L1)))
      let M2 = apply 'map'/2(<span class="fv">f</span>,L2) in
       receive
         M1 when 'true' -&gt;
           call 'erlang':'!'(<span class="fv">i</span>,
              call 'erlang':'++'(M1,M2))
       after 'infinity' -&gt; []
end</code></pre>

</div>
</div>

These snippets roughly correspond to the following Erlang functions.

```erlang
map(F, L) ->
    case L of
        []      -> [];
        [H | T] -> [F(H) | map(F, T)]
    end.

%% Sequential variant
seq(F, El, I) ->
    I ! map(F, El).

%% Concurrent variant
conc(F, El, I, N) ->
    {L1, L2} = lists:split(N, El),
    S = self(),
    spawn(fun() -> S ! map(F, L1) end),
    M2 = map(F, L2),
    receive
        M1 -> I ! M1 ++ M2
    end.
```

# Further reading

- D. Lukács, P. Bereczky, D. Horpácsi, "A Mechanised Semantics of Erlang's References." Erlang Workshop, 2026. [DOI](https://doi.org/10.1145/3830434.3830944)
- P. Bereczky, "Proof Assistant-Based Formalisation of Core Erlang." Ph.D. dissertation, Eötvös Loránd Univ., Budapest, Hungary, 2025. [DOI](https://doi.org/10.15476/ELTE.2025.413)
- G. L. Turán, A. Fomin, P. Bereczky, D. Horpácsi, S. Thompson, "Deriving an Erlang Interpreter from a Mechanised Formal Semantics of Core Erlang." Erlang Workshop, 2025. [DOI](https://doi.org/10.1145/3759161.3763046)
- A. Fomin, P. Bereczky, D. Horpácsi, G. L. Turán, "Mechanised Proofs of Atom Exhaustion in Erlang." Erlang Workshop, 2025. [DOI](https://doi.org/10.1145/3759161.3763045)
- P. Bereczky, D. Horpácsi, S. Thompson, "A Frame Stack Semantics for Sequential Core Erlang." IFL, 2023. [DOI](https://doi.org/10.1145/3652561.3652566)
- D. Horpácsi, P. Bereczky, S. Thompson, "Program Equivalence in an Untyped, Call-by-value Functional Language with Uncurried Functions." JLAMP, 2023. [DOI](https://doi.org/10.1016/j.jlamp.2023.100857)
- P. Bereczky, D. Horpácsi, J. Kőszegi, S. Szeier, S. Thompson, "Validating Formal Semantics by Property-Based Cross-Testing." IFL, 2020. [DOI](https://doi.org/10.1145/3462172.3462200)
- I. Lanese, D. Sangiorgi, G. Zavattaro, "Playing with Bisimulation in Erlang." LNCS, 2019. [DOI](https://doi.org/10.1007/978-3-030-21485-2_6)
- B. Gustavsson, "Core Erlang Optimizations." erlang.org blog, 2018. [URL](https://www.erlang.org/blog/core-erlang-optimizations/)
- B. Gustavsson, "Core Erlang by Example." erlang.org blog, 2018. [URL](https://www.erlang.org/blog/core-erlang-by-example/)
- I. Lanese, N. Nishida, A. Palacios, G. Vidal, "A Theory of Reversibility for Erlang." JLAMP, 2018. [DOI](https://doi.org/10.1016/j.jlamp.2018.06.004)
- L.-Å. Fredlund, "A Framework for Reasoning about Erlang Code." Ph.D. dissertation, KTH Royal Institute of Technology, Stockholm, Sweden, 2001. [URL](https://urn.kb.se/resolve?urn=urn%3Anbn%3Ase%3Akth%3Adiva-3210)
- R. Carlsson et al., "Core Erlang 1.0 Language Specification." Technical report, Uppsala University, 2000. [URL](https://www.diva-portal.org/smash/record.jsf?pid=diva2%3A1695554)

