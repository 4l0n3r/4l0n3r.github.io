+++
title = "Silo - Formal Verification"
date = "2025-02-10"
+++

# 🧪 Formal Verification Participation - Silo V2 (Cantina Contest)

This repository documents my participation in the **Silo V2 Formal Verification Contest** hosted by [Cantina](https://cantina.xyz/), which ran from **Feb 10, 2025**. The goal of this contest was to analyze, reason about, and formally verify the correctness and security properties of the smart contracts provided by Silo using **Certora Prover**.

---

## 🧾 Contest Overview

* **Github Project**: [Silo Rules](https://github.com/4l0n3r/silo-v2-cantina-fv/)
* **Contest Host**: Cantina
* **Start Date**: Feb 10, 2025
* **Focus**: Silo V2 Contracts
* **Category**: Formal Verification (SMT-based logic, rule-based specifications)

---

## 🔍 What I Worked On

This repo showcases my efforts in:

1. **Understanding the behavior of the Silo V2 codebase**
2. **Writing formal rules and invariants using Certora’s specification language**
3. **Verifying those rules against the actual contract implementation**
4. **Iteratively refining the specifications to match protocol behavior**

---

## 🛠 Tools & Stack

* 📜 **Certora Prover**: For writing and verifying smart contract specs
* 🧠 **Certora Specification Language**: To encode protocol invariants and API rules
* 💻 **Solidity**: Smart contract language under verification

---

## 🔭 Scope of Verification
The contest involved analyzing the following key contracts from the Silo V2 protocol:

| Contract                 | SLOC (Source Lines of Code) |
| ------------------------ | --------------------------- |
| `Silo.sol`               | 452                         |
| `PartialLiquidation.sol` | 155                         |
| `Actions.sol`            | 373                         |


## 📜 Lessons Learned

* Hands-on experience with the **Certora Prover** workflow
* Better understanding of how to model complex DeFi lending systems formally
* Learned to handle false positives and debug verification failures effectively

---

## 🧑‍💻 Author

**Johny Vasamsetti**
Web3 & Infrastructure Engineer | Passionate about Smart Contract Security and Formal Verification

Connect with me:

* [GitHub](https://github.com/4l0n3r)
