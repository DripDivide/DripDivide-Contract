# DripsDivide-Contract

A Splitwise-style decentralized application built on the Stellar blockchain using Soroban smart contracts. DripsDivide enables groups to manage shared expenses and settle debts instantly across international borders without high bank fees.

## Overview

DripsDivide-Contract is a Soroban smart contract that provides:

- **Group Expense Management**: Create and manage expense-sharing groups
- **Flexible Expense Splitting**: Support for equal and custom expense splits
- **Intelligent Debt Simplification**: Graph-based algorithm to minimize settlement transactions
- **Multi-Asset Support**: Handle expenses in XLM and custom Stellar assets
- **Cross-Border Settlements**: Instant international payments with minimal fees
- **Comprehensive Tracking**: Complete history of expenses and settlements

## Key Features

### Group Management
- Create expense groups with admin controls
- Add/remove members with proper authorization
- Query all groups for a member

###  Expense Recording
- Record expenses with flexible splitting options
- Equal split: Divide evenly among participants
- Custom split: Specify exact amounts per participant
- Multi-asset support (XLM and custom tokens)

### Debt Calculation
- Automatic debt graph updates
- Intelligent debt simplification algorithm
- Net balance calculations per member
- Separate debt tracking per asset type

### Settlement Processing
- Execute settlements through the contract
- Automatic debt reduction
- Settlement history tracking
- Event emission for all settlements

###  Querying & History
- Balance queries per member and asset
- Expense history with pagination
- Settlement history with filtering
- Date range and asset type filters

## Technology Stack

- **Language**: Rust
- **Platform**: Soroban (Stellar Smart Contracts)
- **SDK**: soroban-sdk v21+
- **Network**: Stellar Blockchain
- **Storage**: Persistent, Instance, and Temporary storage types

## Architecture

### High-Level Components

```
┌─────────────────────────────────────────┐
│       Contract Entry Points             │
│  (initialize, create_group, etc.)       │
└─────────────────┬───────────────────────┘
                  │
    ┌─────────────┼─────────────┐
    │             │             │
    ▼             ▼             ▼
┌────────┐  ┌──────────┐  ┌──────────┐
│ Group  │  │ Expense  │  │   Debt   │
│  Mgmt  │  │Recording │  │Calculation│
└────┬───┘  └────┬─────┘  └────┬─────┘
     │           │             │
     └───────────┼─────────────┘
                 │
                 ▼
        ┌────────────────┐
        │ Storage Layer  │
        │ (Persistent)   │
        └────────────────┘
```

### Module Structure

```
src/
├── lib.rs          # Contract entry points
├── types.rs        # Data structures
├── storage.rs      # Storage patterns
├── group.rs        # Group management
├── expense.rs      # Expense recording
├── debt.rs         # Debt simplification
├── settlement.rs   # Settlement execution
├── query.rs        # Query functions
├── error.rs        # Error types
└── test.rs         # Tests
```

## Data Models

### Group
```rust
pub struct Group {
    pub id: u64,
    pub name: String,
    pub admin: Address,
    pub members: Vec<Address>,
    pub created_at: u64,
}
```

### Expense
```rust
pub struct Expense {
    pub id: u64,
    pub group_id: u64,
    pub payer: Address,
    pub amount: i128,
    pub asset: Address,
    pub description: String,
    pub split_type: SplitType,
    pub shares: Map<Address, i128>,
    pub timestamp: u64,
}
```

### Debt
```rust
pub struct Debt {
    pub payer: Address,
    pub payee: Address,
    pub amount: i128,
    pub asset: Address,
}
```

## Contract Interface

### Initialization
```rust
pub fn initialize(env: Env, admin: Address) -> Result<(), Error>
```

### Group Operations
```rust
pub fn create_group(env: Env, creator: Address, name: String) -> Result<u64, Error>
pub fn add_member(env: Env, admin: Address, group_id: u64, member: Address) -> Result<(), Error>
pub fn remove_member(env: Env, admin: Address, group_id: u64, member: Address) -> Result<(), Error>
pub fn delete_group(env: Env, admin: Address, group_id: u64) -> Result<(), Error>
pub fn get_member_groups(env: Env, member: Address) -> Result<Vec<u64>, Error>
```

### Expense Operations
```rust
pub fn record_expense(env: Env, recorder: Address, group_id: u64, expense: ExpenseInput) -> Result<u64, Error>
pub fn modify_expense(env: Env, creator: Address, expense_id: u64, updates: ExpenseUpdate) -> Result<(), Error>
pub fn delete_expense(env: Env, creator: Address, expense_id: u64) -> Result<(), Error>
pub fn get_expenses(env: Env, group_id: u64, offset: u32, limit: u32) -> Result<Vec<Expense>, Error>
```

### Debt & Settlement Operations
```rust
pub fn get_debts(env: Env, group_id: u64, asset: Address) -> Result<Vec<Debt>, Error>
pub fn settle_debt(env: Env, payer: Address, group_id: u64, payee: Address, amount: i128, asset: Address) -> Result<(), Error>
pub fn get_balance(env: Env, group_id: u64, member: Address, asset: Address) -> Result<i128, Error>
pub fn get_settlements(env: Env, group_id: u64, offset: u32, limit: u32) -> Result<Vec<Settlement>, Error>
```

## Debt Simplification Algorithm

DripsDivide uses an intelligent debt simplification algorithm to minimize the number of settlement transactions:

### How It Works

1. **Calculate Net Balances**: For each member, compute `net_balance = total_paid - total_owed`
2. **Separate Creditors and Debtors**: Split members into those owed money (positive) and those owing money (negative)
3. **Greedy Matching**: Match the largest creditor with the largest debtor repeatedly
4. **Generate Simplified Debts**: Create minimal debt records

### Example

```
Initial Debts:
  Alice owes Bob: $30
  Alice owes Carol: $20
  Bob owes Carol: $10

Net Balances:
  Alice: -$50 (owes)
  Bob: +$20 (owed)
  Carol: +$30 (owed)

Simplified Debts:
  Alice pays Carol: $30
  Alice pays Bob: $20

Result: 2 transactions instead of 3
```

### Complexity
- **Time**: O(n log n) for sorting + O(n) for matching
- **Space**: O(n) for storing balances
- **Scalability**: Handles groups up to 50 members efficiently

## Installation

### Prerequisites

- Rust 1.70+
- Soroban CLI
- Stellar account with testnet/mainnet access

### Setup

1. Clone the repository:
```bash
git clone https://github.com/yourusername/drips-divide-contract.git
cd drips-divide-contract
```

2. Install Soroban CLI:
```bash
cargo install --locked soroban-cli
```

3. Build the contract:
```bash
soroban contract build
```

4. Run tests:
```bash
cargo test
```

## Deployment

### Deploy to Testnet

1. Configure Soroban for testnet:
```bash
soroban config network add testnet \
  --rpc-url https://soroban-testnet.stellar.org:443 \
  --network-passphrase "Test SDF Network ; September 2015"
```

2. Deploy the contract:
```bash
soroban contract deploy \
  --wasm target/wasm32-unknown-unknown/release/drips_divide_contract.wasm \
  --source ADMIN_SECRET_KEY \
  --network testnet
```

3. Initialize the contract:
```bash
soroban contract invoke \
  --id CONTRACT_ID \
  --source ADMIN_SECRET_KEY \
  --network testnet \
  -- initialize \
  --admin ADMIN_ADDRESS
```

## Usage Examples

### Create a Group

```bash
soroban contract invoke \
  --id CONTRACT_ID \
  --source CREATOR_SECRET_KEY \
  --network testnet \
  -- create_group \
  --creator CREATOR_ADDRESS \
  --name "Trip to Paris"
```

### Add Members

```bash
soroban contract invoke \
  --id CONTRACT_ID \
  --source ADMIN_SECRET_KEY \
  --network testnet \
  -- add_member \
  --admin ADMIN_ADDRESS \
  --group_id 1 \
  --member MEMBER_ADDRESS
```

### Record an Expense

```bash
soroban contract invoke \
  --id CONTRACT_ID \
  --source MEMBER_SECRET_KEY \
  --network testnet \
  -- record_expense \
  --recorder MEMBER_ADDRESS \
  --group_id 1 \
  --expense '{"payer":"PAYER_ADDRESS","amount":300,"asset":"ASSET_ADDRESS","description":"Dinner","split_type":"Equal","participants":["ADDR1","ADDR2","ADDR3"],"custom_shares":null}'
```

### Get Simplified Debts

```bash
soroban contract invoke \
  --id CONTRACT_ID \
  --network testnet \
  -- get_debts \
  --group_id 1 \
  --asset ASSET_ADDRESS
```

### Settle a Debt

```bash
soroban contract invoke \
  --id CONTRACT_ID \
  --source PAYER_SECRET_KEY \
  --network testnet \
  -- settle_debt \
  --payer PAYER_ADDRESS \
  --group_id 1 \
  --payee PAYEE_ADDRESS \
  --amount 100 \
  --asset ASSET_ADDRESS
```

## Testing

### Unit Tests

Run unit tests for individual functions:
```bash
cargo test --lib
```

### Property-Based Tests

Run property-based tests (100+ iterations per property):
```bash
cargo test --features proptest
```

### Integration Tests

Run integration tests:
```bash
cargo test --test integration
```

### Test Coverage

Generate test coverage report:
```bash
cargo tarpaulin --out Html
```

## Security Considerations

### Access Control
- **Group Admin**: Only group creator can add/remove members
- **Group Members**: Only members can record expenses
- **Debt Payers**: Only payer can initiate settlements
- **Contract Admin**: Only admin can perform contract-level operations

### Input Validation
- All amounts must be positive
- All addresses must be valid Stellar accounts
- Custom splits must sum to total expense amount
- Expense modifications only allowed within 24 hours

### Storage Security
- TTL management for all persistent data
- Data isolation by asset type
- Cascade deletion for group removal

## Error Codes

| Code | Error | Description |
|------|-------|-------------|
| 1 | NotInitialized | Contract not initialized |
| 2 | AlreadyInitialized | Contract already initialized |
| 10 | Unauthorized | Caller not authorized |
| 11 | NotGroupAdmin | Caller is not group admin |
| 12 | NotGroupMember | Caller is not group member |
| 20 | GroupNotFound | Group does not exist |
| 30 | ExpenseNotFound | Expense does not exist |
| 31 | InvalidAmount | Amount is invalid |
| 32 | InvalidSplit | Split configuration is invalid |
| 33 | SplitSumMismatch | Split shares don't sum to total |
| 34 | ExpenseModificationExpired | 24-hour window expired |
| 40 | DebtNotFound | Debt does not exist |
| 41 | InsufficientDebt | Settlement exceeds debt |
| 50 | SettlementFailed | Settlement execution failed |

## Performance

### Storage Optimization
- Instance storage for config (64KB limit)
- Persistent storage for user data
- Temporary storage for computation cache

### Gas Optimization
- Minimal storage reads
- Efficient data structures (Vec, Map)
- Batch TTL extensions

### Scalability
- Supports groups up to 50 members
- Pagination for large result sets
- O(n log n) debt simplification

## Roadmap

### Phase 1 (Current)
- Core group management
- Expense recording and splitting
- Debt simplification algorithm
- Settlement execution
- Multi-asset support

### Phase 2 (Planned)
- [ ] Recurring expenses
- [ ] Expense categories
- [ ] Multi-currency conversion
- [ ] Expense approval workflow
- [ ] Group templates

### Phase 3 (Future)
- [ ] Off-chain indexing
- [ ] Mobile SDK
- [ ] Web interface
- [ ] Notification system
- [ ] Analytics dashboard

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Development Guidelines

- Follow Rust best practices and idioms
- Write tests for all new features
- Update documentation for API changes
- Run `cargo fmt` and `cargo clippy` before committing
- Ensure all tests pass before submitting PR

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- [Stellar Development Foundation](https://stellar.org/) for the Soroban platform
- [Splitwise](https://www.splitwise.com/) for the inspiration
- The Rust and Stellar communities for their excellent tools and documentation

## Support

- **Documentation**: [Stellar Developers](https://developers.stellar.org/)
- **Issues**: [GitHub Issues](https://github.com/yourusername/drips-divide-contract/issues)
- **Discussions**: [GitHub Discussions](https://github.com/yourusername/drips-divide-contract/discussions)
- **Discord**: [Stellar Discord](https://discord.gg/stellar)

## Contact

- **Project Maintainer**: Your Name
- **Email**: your.email@example.com
- **Twitter**: [@yourhandle](https://twitter.com/yourhandle)

---

Built with  using Stellar and Soroban
