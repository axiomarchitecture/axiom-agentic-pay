# Agentic Pay

A conceptual reference implementation showing how the protocol-independent
**Axiom Reference Model** can be expressed using Kaspa's native **Argent**
language.

This repository is **not** an official Kaspa, Argent or x402 project.

Its purpose is educational:

- demonstrate how semantic responsibilities map to executable actors
- explore native covenant-based commerce on Kaspa
- provide a concrete reference implementation for discussion and learning

---

# Relationship to Axiom

Axiom is a protocol-independent semantic reference model.

Argent is an implementation language for native Kaspa covenants.

This repository explores **one possible mapping** between the two.

It should not be interpreted as an official implementation of Axiom,
nor as guidance from the Argent or Kaspa teams.

---

# Conceptual Architecture

```
Buyer Agent
      │
      ▼
    Escrow
      │
      ▼
Provider Agent
```

The example intentionally focuses on the minimal execution flow rather than
implementing a complete commerce protocol.

---

# Semantic Mapping

| Axiom Responsibility | Argent Mapping |
|----------------------|----------------|
| Intent | BuyerAgent validation |
| Authorization | `require()` conditions |
| Information | Transaction parameters |
| Escrow | Escrow covenant |
| Payment | Native KAS transfer |
| Settlement | Provider release |
| Audit | Kaspa UTXO history |

This mapping is conceptual.

Axiom describes **economic meaning**.

Argent describes **executable behavior**.

---

# Current Status

✅ Compiles successfully using the real Argent compiler

Tested against:

- Argent alpha.9
- Native `argentc`
- Local compilation
- Generated Silverscript output

Generated artifacts:

- BuyerAgent.sil
- Escrow.sil
- ProviderAgent.sil

---

# Current Limitations

This repository intentionally demonstrates concepts rather than production-ready contracts.

Known limitations include:

- simplified escrow logic
- illustrative payment flow
- no complete type-level verification
- no formal security audit

The generated contracts should therefore be viewed as educational reference material.

---

# Why this repository exists

Most protocol documentation explains **how** something works.

This repository explores **why** different components exist by connecting
them to a protocol-independent semantic model.

The goal is to make covenant-based agent commerce easier to understand,
compare and discuss across implementations.

---

# Acknowledgements

Special thanks to the Kaspa and Argent developers whose documentation,
examples and discussions helped shape this exploration.

In particular, feedback from the community helped improve the distinction
between the protocol-independent Axiom model and native Kaspa implementation.

---

# License

MIT
