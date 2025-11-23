
---

## 2️⃣ `smart-contracts/SMART_CONTRACT_SPEC_V1.md` (o `SPEC.md`)

```md
# DISTRIAI – Smart Contract Specification v1

Network: EVM (L2 / sidechain, to be finalized)  
Standard: ERC20

Total supply: **1,000,000,000 DISTRIAI**

---

## 1. Contract Modules

1. **DISTRIAI ERC20 Token**
2. **Reward Minter / Emissions Module**
3. **Presale Contract**
4. **Vesting / Lockup Contracts**
5. **Treasury Multisig Integration**
6. **Claim Module (off-chain credits → on-chain tokens)**
7. **Optional: Governance (Phase 2)**

---

## 2. Token Supply & Allocation (Protocol-Level)

High-level indicative allocation (subject to finalization):

- **Presale & Airdrop:** 10%  
- **Public Sale:** 10%  
- **Team & Contributors:** 15%  
- **Ecosystem Rewards (incl. compute rewards):** 35%  
- **Treasury (DAO):** 15%  
- **Liquidity:** 10%  
- **Strategic Partnerships:** 5%

**Supply model:**

- **650,000,000** tokens **pre-minted** at deployment.  
- **350,000,000** tokens reserved as **reward pool**, minted only by the Reward Minter contract under strict rules.

---

## 3. Core Contract Responsibilities

### 3.1 ERC20 Token Contract

- Standard ERC20 implementation (OpenZeppelin-based).  
- No custom transfer tax / reflections.  
- Roles:
  - `OWNER` or `ADMIN` (initially deployer / multisig).
  - `MINTER_ROLE` assigned to Reward Minter and possibly Vesting contracts.  

- Functions:
  - `mint(address to, uint256 amount)` – restricted to MINTER_ROLE.  
  - `burn(uint256 amount)` – optional, for future tokenomics adjustments.

No upgradeable proxy in v1 (prefer simplicity and auditability).

---

### 3.2 Reward Minter / Emissions

Purpose:  
Mint from the **350M reward pool** according to emissions logic.

Key requirements:

- Hard cap: **cannot mint more than 350,000,000 tokens** total.  
- Only callable through:
  - claim module (credits conversion)  
  - admin-controlled special cases (if any, with timelock).

State variables (indicative):

- `uint256 public constant REWARD_CAP = 350_000_000e18;`  
- `uint256 public totalRewardsMinted;`

Core functions:

- `mintRewards(address to, uint256 amount)`  
  - `require(totalRewardsMinted + amount <= REWARD_CAP);`  
  - `token.mint(to, amount);`  
  - update `totalRewardsMinted`.

Access controlled by a role (e.g. `REWARD_CONTROLLER_ROLE`), assigned to Claim contract.

---

### 3.3 Presale Contract

Purpose: handle token presale against **USDT**.

Assumptions:

- USDT (ERC20) is used on the launch chain for presale and LP.  
- Token price is set in USDT per DISTRIAI.  
- Vesting/lockup rules apply per phase.

Core features:

- Accept USDT deposits.  
- Track user contributions.  
- Enforce:
  - min / max contribution per wallet  
  - hard cap / soft cap  
- Issue claimable token balances with:
  - lockup (e.g. 1–4 months)  
  - linear vesting where applicable.

Key state:

- `IERC20 public usdt;`  
- `IERC20 public token;`  
- `uint256 public tokenPrice;` (in USDT decimals)  
- `uint256 public totalRaised;`  
- per-user:
  - `contributedUSDT`  
  - `allocation` (token amount)  
  - `claimed` (claimed so far)

LP logic is handled off-chain / by treasury after presale (presale contract does **not** create LP directly in v1, to keep it simple).

---

### 3.4 Vesting & Lockup

Different allocations (team, advisors, presale, etc.) are locked and vested through dedicated contracts.

Patterns:

- **Team & Advisors:**
  - cliff: 6–12 months  
  - vesting: 18–24 months lineare  

- **Presale / Public Sale (if needed):**
  - lockup: 1–4 months  
  - linear unlock or cliff + unlock.

Implementation:  
Use standard **VestingVault** contract, mapping:

- `beneficiary` → `totalAllocation`  
- schedule (start, cliff, duration)  
- `released` amount  

Functions:

- `release(address beneficiary)` – transfers claimable tokens to beneficiary.  
- `releasableAmount(beneficiary)` – view.

Token is held by the vesting contract and released over time.

---

### 3.5 Treasury & Multisig

Treasury is **off-chain managed** by a 2/3 multisig wallet that holds:

- presale USDT leftovers  
- team tokens (if not fully in vesting contracts)  
- treasury allocation  

On-chain, treasury is just a standard EOA / multisig address, not a separate contract in v1.

Later, this can be migrated to a DAO-controlled contract.

---

### 3.6 Claim Module (Credits → Tokens)

This contract bridges **off-chain credits** (tracked by backend) and **on-chain DISTRIAI tokens**.

Flow:

1. Backend computes `claimable_amount` for a user.  
2. Backend builds signed payload:
   - `user`  
   - `amount`  
   - `nonce`  
   - `deadline`  

3. Backend signs with private key.  
4. User / dApp submits to Claim contract:
   - payload + signature.

Contract checks:

- signature validity  
- nonce not used  
- deadline not expired  
- amount within emission / per-user limits  

On success:

- calls Reward Minter: `mintRewards(user, amount)`.

Core variables:

- `address public signer;` (backend signer)  
- `mapping(address => uint256) public nonces;`

Events:

- `Claimed(address indexed user, uint256 amount);`

---

## 4. Security & Invariants

- Total supply must **never exceed 1,000,000,000 DISTRIAI**.  
- Reward pool cannot mint above 350M.  
- No raw `mint` exposed to EOA, only to controlled modules.  
- Presale cannot withdraw USDT to arbitrary addresses except treasury / team multisig.  
- Claim contract cannot bypass emission caps.  
- No external upgradeability in v1 (or if used, strictly controlled and documented).

Recommended:

- Use OpenZeppelin templates wherever possible.  
- Use Foundry for fuzz testing and invariant checks.  

---

## 5. Events & Transparency

Important events:

- `TokenDeployed(address token);`  
- `PresaleContribution(address indexed user, uint256 usdtAmount, uint256 tokenAllocation);`  
- `VestingCreated(address indexed beneficiary, uint256 allocation);`  
- `VestingReleased(address indexed beneficiary, uint256 amount);`  
- `Claimed(address indexed user, uint256 amount);`  
- `RewardsMinted(address indexed to, uint256 amount);`

All on-chain token flows must be traceable and explainable from Etherscan / explorer.

---

## 6. Phase 1 Scope (MVP On-Chain)

For v1 contracts, minimum scope:

- ERC20 token deployed with 650M pre-minted.  
- Reward Minter with 350M cap.  
- Presale contract with USDT support.  
- Simple vesting contracts for:
  - presale (if locked)  
  - team allocation  

- Claim module for backend-signed emissions.

Governance/DAO, advanced LP lock, complex emission curves → Phase 2.
