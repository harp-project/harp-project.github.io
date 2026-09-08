**We design and implement trustworthy software tools (such as refactorings, compilers, and program verifiers) for Erlang, using formal methods.**

In the past few years, we have mechanized the Core Erlang language in the Rocq interactive theorem prover. Currently, our team is working on building a wide range of applications on the top of this mechanized formal semantics. We want to bring high-assurance to the entire BEAM community by building formally based tools.

# 🏛️ A mechanized formal semantics for Core Erlang

Formal semantics describe programming languages with mathematical precision. With the semantics, one can examine program behaviour, define program equivalence, and formally verify program correctness or program safety. We defined formal semantics for Core Erlang, a standard intermediate language in Erlang/OTP; this allows us to develop high assurance tools not only for Erlang but also for other BEAM-based languages such as Elixir, Gleam and LFE.

[Our mechanized formal semantics](https://github.com/harp-project/Core-Erlang-Formalization) includes the complete, mathematically precise definition of the behaviour of both the sequential and the concurrent features of Core Erlang. In particular, we have a fairly complete mechanization of Erlang's actor model. Furthermore, we have defined program equivalence (contextual, CIU, log.rel., and bisimulations) so we can mathematically prove if two programs behaviours are indistinguishable.

Our formal definition has been [validated against the reference implementation](https://github.com/harp-project/erlang-semantics-testing) (i.e. we tested if the behaviour we defined for programs matches their behaviour when run in the interpreter).

Technical highlights:
 - representative coverage of Core Erlang including semantics of the Erlang actor model
 - deep embedding with de Bruijn variables and substitution theory
 - frame stack style small-step semantics with corresponding metatheory
 - contextual and CIU equivalence, logical relations, barbed bisimulation

# Security verification

# Property verification

# Certified compilation

# Featured publications

- Benjamin Rosta, "Property-based verification of Erlang functions using formal methods." M.S.c. thesis, Dept. Prog. Lang. and Compilers, Eötvös Loránd Univ., Budapest, Hungary, 2026. https://beng.web.elte.hu/works/Benjamin_Rosta_MSc_thesis.pdf
- P. Bereczky, D. Horpácsi, "Formally Based Tools for Safer Erlang" CODE BEAM Europe 2025. https://www.youtube.com/watch?v=_GygjVGrrzs
- P. Bereczky, "Proof Assistant-Based Formalisation of Core Erlang," Ph.D. dissertation, Dept. Program. Lang. and Compilers, Eötvös Loránd Univ., Budapest, Hungary, 2025. https://doi.org/10.15476/ELTE.2025.413
- Arsenii Fomin, Péter Bereczky, Dániel Horpácsi, and Gergő Lajos Turán, "Mechanised Proofs of Atom Exhaustion in Erlang." In Proceedings of the 24th ACM SIGPLAN International Workshop on Erlang (Erlang '25). Association for Computing Machinery, New York, NY, USA, 14–25. 2025. https://doi.org/10.1145/3759161.3763045
- Gergő Lajos Turán, Arsenii Fomin, Péter Bereczky, Dániel Horpácsi, and Simon Thompson, "Deriving an Erlang Interpreter from a Mechanised Formal Semantics of Core Erlang." In Proceedings of the 24th ACM SIGPLAN International Workshop on Erlang (Erlang '25). Association for Computing Machinery, New York, NY, USA, 26–39. 2025. https://doi.org/10.1145/3759161.3763046
- Péter Bereczky, Dániel Horpácsi, Judit Kőszegi, Soma Szeier, and Simon Thompson, "Validating Formal Semantics by Property-Based Cross-Testing." In Proceedings of the 32nd Symposium on Implementation and Application of Functional Languages (IFL '20). Association for Computing Machinery, New York, NY, USA, 150–161. 2021. https://doi.org/10.1145/3462172.3462200
