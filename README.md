*We design and implement trustworthy software tools (such as refactorings, compilers, and program verifiers) for Erlang, primarily by using formal methods.*

In the past few years, we have successfully mechanized the Core Erlang language in the Rocq interactive theorem prover, and validated it against the reference implementation. Besides the definition of the sequential and the concurrent features of the language, we have defined program equivalence (contextual, CIU, log.rel., and bisimulations) and applied it in various examples.

Currently, our team is working on building a wide range of applications on the top of the formal semantics. We want to bring high assurance and formal methods to the entire BEAM community.

# A mechanized formal semantics for Core Erlang

Formal semantics describe programming languages with mathematical precision. With the semantics, one can examine program behaviour, define program equivalence, and formally verify program correctness or program safety. We defined formal semantics for Core Erlang, a standard intermediate language in Erlang/OTP. This allows us to develop high assurance tools not only for Erlang but also for other BEAM-based languages such as Elixir, Gleam and LFE.

# Featured repositories

 - [Core Erlang mechanized in Rocq](https://github.com/harp-project/Core-Erlang-Formalization)
 - [Property-based testing of Erlang formal semantics](https://github.com/harp-project/erlang-semantics-testing)
 - [Applicative Matching Logic mechanized in Rocq](https://github.com/harp-project/AML-Formalization)

# Featured publications

 - P. Bereczky, "Proof Assistant-Based Formalisation of Core Erlang," Ph.D. dissertation, Dept. Program. Lang. and Compilers, Eötvös Loránd Univ., Budapest, Hungary, 2025. https://doi.org/10.15476/ELTE.2025.413
 - P. Bereczky, D. Horpácsi, "Formally Based Tools for Safer Erlang" CODE BEAM Europe 2025. https://www.youtube.com/watch?v=_GygjVGrrzs
 - Arsenii Fomin, Péter Bereczky, Dániel Horpácsi, and Gergő Lajos Turán, "Mechanised Proofs of Atom Exhaustion in Erlang." In Proceedings of the 24th ACM SIGPLAN International Workshop on Erlang (Erlang '25). Association for Computing Machinery, New York, NY, USA, 14–25. 2025. https://doi.org/10.1145/3759161.3763045
- Gergő Lajos Turán, Arsenii Fomin, Péter Bereczky, Dániel Horpácsi, and Simon Thompson, "Deriving an Erlang Interpreter from a Mechanised Formal Semantics of Core Erlang." In Proceedings of the 24th ACM SIGPLAN International Workshop on Erlang (Erlang '25). Association for Computing Machinery, New York, NY, USA, 26–39. 2025. https://doi.org/10.1145/3759161.3763046
- Benjamin Rosta, "." M.S.c. thesis, Dept. Prog. Lang. and Compilers, Eötvös Loránd Univ., Budapest, Hungary, 2026.
