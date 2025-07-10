+++
title = "Blend - Formal Verification"
date = "2025-02-24"
+++

# 🧪 Formal Verification Participation - Blend Protocol (Cantina Contest)

This repository documents my participation in the **Blend Protocol Formal Verification Contest** hosted by [Cantina](https://cantina.xyz/) in **February 2025**. The project focuses on analyzing and verifying key Rust smart contracts of the Blend Protocol using **SorabanProver**, a static verification tool for Soroban (Stellar’s smart contract platform).

---

## 📘 About Blend Protocol

**Blend** is a universal liquidity primitive that enables the permissionless creation of lending pools. It’s written in **Rust** and deployed on the **Soroban** smart contract platform.

* 🛠️ Language: Rust
* 🧠 Target: Formal Verification with [SorabanProver](https://github.com/stellar/soroban-prover)

---

## 💾 Contest Overview

* **Github Project**: [Blend Protocol](https://github.com/4l0n3r/2025-02-blend-fv)
* **Contest Host**: Cantina
* **Date**: Feb 2025
* **Focus**: Rust-based smart contracts
* **Category**: Formal Verification using SMT-solvers and rule-based specifications

---

## 🔍 Scope of Verification

The verification focused on the following Rust source files:

| Contract File        | Description               |
| -------------------- | ------------------------- |
| `withdrawal.rs`      | Handles user withdrawals  |
| `user.rs`            | User identity and actions |
| `deposit.rs`         | Token deposits logic      |
| `fund_management.rs` | Fund pool and accounting  |
| `pool.rs`            | Lending pool core logic   |

---

## 📁 Project Layout for SorabanProver

```
backstop/
├── confs/                           # Configuration files per contract
│   └── *.conf
├── src/
│   └── certora_specs/              # Verification specifications
│       ├── *.spec
│       ├── summaries/              # Function summaries
│       └── mocks/                  # Mock implementations
├── mutations/                      # Mutation tests for evaluation
└── certora_build.py                # Build helper script
```

---

## 🧪 Running Verification

### 1. Compile the Code

All edits and spec additions should occur under the `backstop/` directory.

```bash

cd backstop
just build
```

### 2. Run the Prover

Run the SorabanProver for a specific contract configuration:

```bash
  cd confs
  certoraSorobanProver withdrawal.conf   # example
```

> 🔒 You may need to first run:
>
> ```bash
> chmod +x certora_build.py
> ```

---

## ✅ Key Highlights

* Defined and tested:

    * **Function-level specifications**
    * **State invariants**
* Verified business logic and safety properties around:

    * Pool reserve tracking
    * User deposit and withdrawal correctness
    * Consistency in fund management

---

## 🧠 Learnings

* Gained hands-on experience with **SorabanProver**
* Practiced writing custom **mock functions and summaries**
* Understood the **verification lifecycle** for Rust-based smart contracts
* Tackled challenges with symbolic state, mutation testing, and specification refinement

---

## 🧑‍💻 Author

**Johny Vasamsetti**
Smart Contract Security Enthusiast | Formal Verification | Rust | Web3 Infra

* [GitHub](https://github.com/4l0n3r)

---

## 📌 Note

This repository is a submission for educational and learning purposes. It does not contain any security disclosures or sensitive vulnerabilities.
