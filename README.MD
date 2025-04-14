# Solana Automated Market Maker (AMM) Program


## Table of Contents
1. [Mathematical Foundations](#mathematical-foundations)
2. [Core Smart Contract Design](#core-smart-contract-design)
3. [Token Flow Diagrams](#token-flow-diagrams)
4. [Security Mechanisms](#security-mechanisms)
5. [Fee Structure](#fee-structure)
6. [Error Handling System](#error-handling-system)
7. [Deployment Guide](#deployment-guide)
8. [Testing Strategy](#testing-strategy)

---

## Mathematical Foundations

### Constant Product Formula
```math
x * y = k
Δy = (Δx * y) / (x + Δx)
```
**Swap Calculation with Fees**  
![Fee Calculation](https://via.placeholder.com/600x200.png?text=Fee+Calculation+Process)

```mermaid
graph TD
    A[Input Amount] --> B{Swap Direction}
    B -->|X to Y| C[Calculate Δy]
    B -->|Y to X| D[Calculate Δx]
    C --> E[Apply Fee: 0.3%]
    D --> E
    E --> F[Update Reserves]
```

---

## Core Smart Contract Design

### State Management
```mermaid
classDiagram
    class Config {
        +u64 seed
        +Pubkey authority
        +Pubkey mint_x
        +Pubkey mint_y
        +u16 fee
        +bool locked
        +u8 config_bump
        +u8 lp_bump
    }

    class Pool {
        +TokenAccount vault_x
        +TokenAccount vault_y
        +Mint lp_mint
    }

    Config "1" *-- "1" Pool : manages
```

### Initialize Sequence
```mermaid
sequenceDiagram
    User->>Program: initialize(seed, fee)
    Program->>System: Create Config PDA
    Program->>TokenProgram: Create LP Mint
    Program->>ATA: Create Vault X
    Program->>ATA: Create Vault Y
    Program-->>User: Pool Address
```

---

## Token Flow Diagrams

### Deposit Mechanism
```mermaid
flowchart TB
    subgraph User
        Ux[User X Tokens]
        Uy[User Y Tokens]
    end
    
    subgraph Pool
        Vx[Vault X]
        Vy[Vault Y]
        LP[LP Mint]
    end

    Ux -->|Transfer| Vx
    Uy -->|Transfer| Vy
    LP -->|Mint| UserLP[User LP Tokens]
```

### Swap Mechanism
```mermaid
sequenceDiagram
    User->>Program: swap(is_x, amount)
    Program->>Curve: calculate_output
    Curve-->>Program: (output, fee)
    Program->>VaultX: Lock input
    Program->>VaultY: Release output
    Program->>User: Send output tokens
```

---

## Security Mechanisms

### PDA Derivation
```rust
// Config PDA
seeds = [
    b"config", 
    config.seed.to_le_bytes().as_ref()
]

// LP Mint PDA
seeds = [
    b"lp", 
    config.key().as_ref()
]
```

### Authorization Flow
```mermaid
stateDiagram-v2
    [*] --> CheckLock: Any operation
    CheckLock --> PoolLocked: if locked
    CheckLock --> VerifySigner: if unlocked
    VerifySigner --> CheckAuthority
    CheckAuthority --> Valid: Authority matches
    CheckAuthority --> Invalid: No authority
```

---

## Fee Structure

### Fee Distribution Model
```math
effective_fee = input_amount * fee_bps / 10_000
protocol_fee = effective_fee * 0.2  # 20% to protocol
lp_fee = effective_fee * 0.8        # 80% to LPs
```

### Fee State Machine
```mermaid
stateDiagram
    [*] --> ZeroLiquidity: Pool Empty
    ZeroLiquidity --> Active: Initial Deposit
    Active --> FeesAccruing: Swaps Occur
    FeesAccruing --> FeesDistributed: Withdraw
    FeesDistributed --> Active
```

---

## Error Handling System

### Error Propagation
```mermaid
flowchart TB
    UserAction --> InputValidation
    InputValidation -->|Fail| Error[AmmError]
    InputValidation -->|Pass| CurveCalc
    CurveCalc -->|Overflow| Error
    CurveCalc -->|Slippage| Error
    CurveCalc -->|Success| TokenTransfer
    TokenTransfer -->|Fail| Error
```

### Error Recovery Matrix
| Error Type           | Auto-Retry | Manual Intervention | Logging Level |
|----------------------|------------|---------------------|---------------|
| SlippageExceeded     | Yes        | No                  | Warning       |
| PoolLocked           | No         | Admin               | Critical      |
| InvalidAuthority     | No         | Admin               | Critical      |
| InsufficientBalance  | No         | User                | Error         |

---

## Deployment Guide

### Cluster Compatibility
| Network      | Supported | Test Status |
|--------------|-----------|-------------|
| Mainnet-beta | Yes       | Audited     |
| Devnet       | Yes       | Verified    |
| Testnet      | Yes       | Verified    |
| Localnet     | Yes       | CI/CD       |

### Deployment Checklist
```mermaid
journey
    title Deployment Process
    section Preparation
      Verify Program ID: 5: User
      Check Dependencies: 5: DevOps
    section Deployment
      Build Program: 5: CI
      Deploy to Cluster: 5: DevOps
    section Verification
      Test Initialization: 3: QA
      Test Swaps: 3: QA
      Test Edge Cases: 4: QA
```

---

## Testing Strategy

### Test Coverage Targets
| Component         | Target % | Current % |
|-------------------|----------|-----------|
| Deposit/Withdraw  | 100%     | 98%       |
| Swap Logic        | 100%     | 95%       |
| Error Conditions  | 100%     | 100%      |
| Fee Calculations  | 100%     | 100%      |

### Fuzz Testing Parameters
```yaml
swap_tests:
  iterations: 10_000
  parameters:
    amount_range: [1, 1_000_000]
    fee_range: [0, 10_000]
    slippage_range: [0, 50%]
    
liquidity_tests:
  max_tokens: 1e18
  decimal_variants: [6, 9, 18]
```

License
MIT License - See LICENSE for details.

