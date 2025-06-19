+++
title = "Decentralized Lottery using Solidity"
date = 2024-06-17
description = "A decentralized lottery system using Chainlink VRF and Foundry framework."

[taxonomies]
tags=["solidity"]

[extra]
#link_to = "https://github.com/4l0n3r/lottery"
+++

This project implements a decentralized lottery system using Solidity and the Foundry framework. The randomness for winner selection is powered by [Chainlink VRF](https://docs.chain.link/vrf), ensuring fairness and transparency.

### 🛠 Features

- Participants enter the lottery by paying a fixed entrance fee (in ETH).
- A Chainlink VRF (Verifiable Random Function) is used to pick a truly random winner.
- The winner receives the total accumulated ETH in the contract.
- The contract automatically resets for a new round after a winner is selected.
- Integrated with mock contracts for local testing and forked mainnet simulations.

### 📦 Tech Stack

- **Solidity** for smart contract logic
- **Foundry** for development and testing
- **Chainlink VRF** for randomness
- **Hardhat (optional)** for simulation/testing
- **Remix** for contract interaction in browser

### 🧪 Test Coverage

- Full test suite using Forge (Foundry).
- Mocks are used to simulate Chainlink services locally.
- Tests include edge cases like:
    - Multiple entries
    - Time interval restrictions
    - VRF fulfillment logic
    - ETH transfer assertions

### 🚀 Deployment

Deployed and tested on the Sepolia testnet. Chainlink VRF subscription was used with mocks and real coordinator.

---

Check out the source code and try your luck 🎲: [GitHub Repo](https://github.com/4l0n3r/lottery)
