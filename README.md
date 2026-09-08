**We design and implement trustworthy software tools for Erlang, by using formal methods.**

Our key asset is a formal semantics for Core Erlang, mechanized in Rocq theorem prover. Besides developing this formal foundation, our team is working on creating applications on the top of this mechanized formal semantics, because we want to build formally based, high-assurance tools to the entire BEAM community.

# A mechanized formal semantics for Core Erlang

Formal semantics describe programming languages with mathematical precision. With the semantics, one can examine program behaviour, define program equivalence, and formally verify program correctness or program safety. We defined formal semantics for Core Erlang, a standard intermediate language in Erlang/OTP; this allows us to develop high assurance tools not only for Erlang but also for other BEAM-based languages such as Elixir, Gleam and LFE.

[Our mechanized formal semantics](https://github.com/harp-project/Core-Erlang-Formalization) includes a fairly complete, mathematically precise definition of the behaviour of sequential and concurrent features of (Core) Erlang, including the actor model and its fault tolerance features. Furthermore, we defined program equivalence so we can mathematically prove or disprove if two programs behaviours are indistinguishable, enabling the verification of refactoring and optimisation.

Our formal definition has been [validated against the reference implementation](https://github.com/harp-project/erlang-semantics-testing), that is, it is assured by automated testing that the behaviour we defined for programs matches their behaviour when run in the interpreter.

Technical highlights:
 - representative coverage of Core Erlang including semantics of the Erlang actor model
 - deep embedding with de Bruijn variables and substitution theory
 - frame stack style small-step semantics with corresponding metatheory
 - contextual and CIU equivalence, logical relations, barbed bisimulation

# Security verification

The formal semantics defines every possible behaviour of programs, finite or infinite, deterministic or nondeterministic. Vulnerable behaviour can be mathematically defined, and formally verified and machine-checked in our implementation. As a first case study, our team has demonstrated [proving atom exhaustion vulnerabilities](https://github.com/harp-project/Core-Erlang-Formalization/blob/master/src/FrameStack/Vulnerabilities/AtomExhaustion.v) by using a calculus tailored for this proof domain. We are looking forward to extend this work by formally defining security guidelines and certifying compliance, as well as formally verifying CVE affectedness (pending grant application to HORIZON-CL3-2026-02-CS-ECCC-01).

# Property verification

We can state correctness properties about the programs and we can verify them against all possible behaviours identified by the semantics. Work-in-progress in this topic is demonstrated by automatically proving basic properties of [integer-processing recursive functions](https://github.com/harp-project/Core-Erlang-Formalization/pull/58) and [list-processing recursive functions](https://github.com/harp-project/Core-Erlang-Formalization/pull/66). These prototypes formalise Hoare-triples that were manually constructed from QuickCheck properties, but automatic translation for these is a straightforward, technical step.

*Note that as far as we understand, our method for property verification is fundamentally different from the approach used in [Lynx](https://github.com/josevalim/lynx).* Although both are based on the idea of turning contracts and program properties into proof obligations, Lynx translates Elixir programs into Lean by using a shallow embedding of functions (Elixir functions become Lean functions and their behaviour is defined by host language semantics). That is, this approach incorporates unverified, trusted code base (which brings programs and their specifications into the verification tool), and necessitates further (nontrivial) validation of whether the property proved formally is reflected in the program's actual behaviour. **In contrast**, our approach uses deep embedding and defines the behaviour *explicitly* over the abstract syntax deeply embedded in the theorem prover. Thus, programs can be processed by the semantics without translation, but by a trivial mapping from the abstract format to the abstract syntax used in the theorem prover. This allows us to bring programs into the verification pipeline effortlessly.

# Certified compilation

Our team carries out a 4-year (2025-2028) project that develops a certified optimising compiler for Core Erlang. We will implement a compilation from Core Erlang to BEAM via ANF and SSA, and will implement and verify many of the standard optimisation steps also available in the Erlang/OTP compiler. Currently, the intermediate languages are under development, investigating a common language metatheory that facilities the verification of translation and optimisation steps.

# Featured publications

- Benjamin Rosta, "Property-based verification of Erlang functions using formal methods." M.S.c. thesis, Dept. Prog. Lang. and Compilers, Eötvös Loránd Univ., Budapest, Hungary, 2026. [PDF](https://beng.web.elte.hu/works/Benjamin_Rosta_MSc_thesis.pdf)
- Péter Bereczky, Dániel Horpácsi, "Formally Based Tools for Safer Erlang" CODE BEAM Europe 2025. [URL](https://www.youtube.com/watch?v=_GygjVGrrzs)
- Péter Bereczky, "Proof Assistant-Based Formalisation of Core Erlang," Ph.D. dissertation, Dept. Program. Lang. and Compilers, Eötvös Loránd Univ., Budapest, Hungary, 2025. [DOI](https://doi.org/10.15476/ELTE.2025.413)
- Arsenii Fomin, Péter Bereczky, Dániel Horpácsi, and Gergő Lajos Turán, "Mechanised Proofs of Atom Exhaustion in Erlang." In Proceedings of the 24th ACM SIGPLAN International Workshop on Erlang (Erlang '25). Association for Computing Machinery, New York, NY, USA, 14–25. 2025. [DOI](https://doi.org/10.1145/3759161.3763045)
- Gergő Lajos Turán, Arsenii Fomin, Péter Bereczky, Dániel Horpácsi, and Simon Thompson, "Deriving an Erlang Interpreter from a Mechanised Formal Semantics of Core Erlang." In Proceedings of the 24th ACM SIGPLAN International Workshop on Erlang (Erlang '25). Association for Computing Machinery, New York, NY, USA, 26–39. 2025. [DOI](https://doi.org/10.1145/3759161.3763046)
- Péter Bereczky, Dániel Horpácsi, Judit Kőszegi, Soma Szeier, and Simon Thompson, "Validating Formal Semantics by Property-Based Cross-Testing." In Proceedings of the 32nd Symposium on Implementation and Application of Functional Languages (IFL '20). Association for Computing Machinery, New York, NY, USA, 150–161. 2021. [DOI](https://doi.org/10.1145/3462172.3462200)

# Contact

- Dániel Horpácsi (daniel-h AT elte.hu)
- Péter Bereczky (berpeti AT inf.elte.hu)
- Simon Thompson (s.j.thompson AT kent.ac.uk)
