# DISTRIAI – Smart Contracts

This repository contains all on-chain components of the DISTRIAI protocol, including:

## 📌 Modules
- **ERC20 Token Contract** (DISTRIAI)
- **Presale Module**
- **Vesting / Lockup**
- **Reward Minter**
- **Treasury (2/3 Multisig)**
- **Governance (Phase 2)**

## 🎯 Goals
- Secure token issuance
- Transparent reward distribution
- Sustainable economics model
- Full auditability

## 🛠️ Stack
- Solidity (>=0.8.x)
- Hardhat (development)
- Foundry (testing + security)
- OpenZeppelin libraries

## 📄 Contract Architecture (v1)
- Pre-minted supply (650M)
- Reward pool (350M) – minted by Minter contract
- Controlled emissions
- Signature-based or Merkle-based reward claiming
- Separation of concerns (token ⇌ minter ⇌ treasury)

## 🔒 Security
- No compute logic on-chain
- Strict access control
- Upgradability TBD (most likely non-upgradeable for core contracts)

## 🧪 Testing
- Unit tests (Foundry)
- Stress + fuzz testing
- Gas profiling and optimizations

## 📦 Directory Structure (placeholder)
