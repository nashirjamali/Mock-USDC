# Transaction History Test Guide

## Overview
This document describes how to test the newly implemented transaction history functionality for the Mock-USDC token.

## New Methods Added

### 1. `get_transactions(req: GetTransactionsRequest): GetTransactionsResponse`
- Retrieves a range of transactions by block index
- Parameters:
  - `start`: Starting transaction index (block ID)
  - `length`: Number of transactions to retrieve
- Returns: Array of transactions with their block IDs and total log length

### 2. `get_transaction(txId: TxIndex): ?Transaction`
- Retrieves a single transaction by its block ID
- Returns: The transaction if found, null otherwise

### 3. `get_account_transactions(req: GetAccountTransactionsRequest): GetAccountTransactionsResponse`
- Retrieves transactions for a specific account
- Parameters:
  - `account`: The account to query
  - `start`: Optional starting transaction index
  - `max_results`: Maximum number of transactions to return
- Returns: Array of transactions involving the account and oldest transaction ID

## Testing Steps

### 1. Create a Token
```bash
# Deploy the canister
dfx deploy

# Create a token with initial supply
dfx canister call backend create_token '(
  record {
    token_name = "Mock USDC";
    token_symbol = "mUSDC";
    initial_supply = 1000000;
    token_logo = "https://example.com/logo.png";
  }
)'
```

### 2. Test get_transactions
```bash
# Get first 10 transactions
dfx canister call backend get_transactions '(
  record {
    start = 0;
    length = 10;
  }
)'

# Get transactions starting from index 5
dfx canister call backend get_transactions '(
  record {
    start = 5;
    length = 5;
  }
)'
```

### 3. Test get_transaction
```bash
# Get transaction by block ID (index 0)
dfx canister call backend get_transaction '(0)'

# Get transaction by block ID (index 1)
dfx canister call backend get_transaction '(1)'
```

### 4. Test get_account_transactions
```bash
# Get transactions for your account
dfx canister call backend get_account_transactions '(
  record {
    account = record {
      owner = principal "your-principal-id";
      subaccount = null;
    };
    start = null;
    max_results = 10;
  }
)'
```

### 5. Perform Some Transfers
```bash
# Transfer tokens to test transaction history
dfx canister call backend icrc1_transfer '(
  record {
    from_subaccount = null;
    to = record {
      owner = principal "another-principal-id";
      subaccount = null;
    };
    amount = 1000;
    fee = null;
    memo = null;
    created_at_time = null;
  }
)'
```

### 6. Verify Transaction History
After performing transfers, check the transaction history again to see the new transactions.

## Expected Results

1. **get_transactions**: Should return transactions with their block IDs (0, 1, 2, etc.)
2. **get_transaction**: Should return the specific transaction for the given block ID
3. **get_account_transactions**: Should return only transactions involving the specified account

## Block ID System

- Each transaction gets a unique block ID (TxIndex) starting from 0
- Block ID 0 is typically the initial mint transaction
- Subsequent transactions get incremental block IDs (1, 2, 3, etc.)
- This mimics how real ICRC-1 tokens work with their transaction indexing

## Notes

- The transaction history is persistent across canister upgrades
- All transaction types (Mint, Burn, Transfer, Approve) are included in the history
- The block ID system follows ICRC-1 standards for compatibility with real tokens
