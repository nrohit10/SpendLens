# Column Mapping to Canonical Schema

## Purpose

This document defines how raw columns from different financial statement
formats are mapped into the canonical `expenses` table used by SpendLens.

The objective of this mapping is to normalize heterogeneous statement formats
while preserving all relevant information required for analytics, recurrence
detection, and AI-driven insights.

---

## Canonical Table Reference: `expenses`

Each record in the `expenses` table represents a single financial transaction.
The table is defined in detail in `docs/schema.md`.

Key canonical fields referenced in this document:

- transaction_id (generated)
- serial_no
- transaction_date
- posted_date
- raw_description
- amount
- direction
- transaction_type
- card_provider
- reward_points
- source

---

## 1. ICICI Credit Card Statement (PDF)

### Raw Columns

### Column Mapping

| Canonical Field  | Source Column       | Notes                             |
| ---------------- | ------------------- | --------------------------------- |
| transaction_date | Date                | Used as the transaction date      |
| posted_date      | Date                | Same as transaction_date          |
| serial_no        | SerNo.              | Statement reference number        |
| raw_description  | Transaction Details | Stored without modification       |
| reward_points    | Reward Points       | Nullable                          |
| amount           | Amount (in ₹)       | Always stored as a positive value |
| direction        | Derived             | Credit card purchase → debit      |
| transaction_type | Constant            | credit_card                       |
| card_provider    | Constant            | ICICI                             |
| source           | Constant            | ICICI_CC                          |
| transaction_id   | Generated           | Deterministic hash                |

### Notes

- `Intl.#` is ignored in the initial implementation
- Refunds or reversals should be inferred from the transaction description if present

---

## 2. HDFC Credit Card Statement (PDF)

### Raw Columns

### Column Mapping

| Canonical Field  | Source Column           | Notes                             |
| ---------------- | ----------------------- | --------------------------------- |
| transaction_date | DATE & TIME             | Date extracted from timestamp     |
| posted_date      | DATE & TIME             | Same value used initially         |
| raw_description  | TRANSACTION DESCRIPTION | Stored without modification       |
| amount           | AMOUNT                  | Always stored as a positive value |
| direction        | Derived                 | Purchases → debit                 |
| transaction_type | Constant                | credit_card                       |
| card_provider    | Constant                | HDFC                              |
| reward_points    | NULL                    | Not available                     |
| serial_no        | NULL                    | Not available                     |
| source           | Constant                | HDFC_CC                           |
| transaction_id   | Generated               | Deterministic hash                |

### Notes

- `PI` (Purchase Indicator color) is not used in the current version
- This field may be incorporated later if additional semantics are required

---

## 3. Bank Statement (Savings / Current Account)

### Raw Columns

### Column Mapping

| Canonical Field  | Source Column        | Notes                                |
| ---------------- | -------------------- | ------------------------------------ |
| transaction_date | Date                 | Primary transaction date             |
| posted_date      | Value Dt             | Date the transaction was posted      |
| raw_description  | Narration            | Stored without modification          |
| serial_no        | Chq./Ref.No.         | Nullable                             |
| amount           | Withdrawal / Deposit | One of the two will be populated     |
| direction        | Derived              | Withdrawal → debit, Deposit → credit |
| transaction_type | Derived              | upi or bank_transfer                 |
| card_provider    | NULL                 | Not applicable                       |
| reward_points    | NULL                 | Not applicable                       |
| source           | Bank identifier      | e.g. HDFC_BANK                       |
| transaction_id   | Generated            | Deterministic hash                   |

### Amount and Direction Logic

---

## 4. Transaction ID Generation

### Purpose

`transaction_id` is a system-generated identifier used to ensure idempotent
ingestion, deduplication, and stable referencing across pipelines.

### Hash Inputs

The hash must be generated from a combination of stable transaction attributes:

If duplicate collisions are detected, `serial_no` or `posted_date` may be added
to the hash inputs.

### Properties

- Deterministic: same input always produces the same identifier
- Not provided by banks
- Not reversible
- Used internally by SpendLens only

---

## Design Principles

- Missing fields are allowed and represented as NULL
- Constants are explicitly assigned where applicable
- No transformation or business logic is embedded in this document
- All transformation logic lives in ingestion pipelines

---

## Future Extensions

This mapping can be extended to support:

- Additional banks or credit card providers
- Foreign currency transactions
- EMI or installment detection
- Enhanced refund handling

All extensions must remain backward-compatible with the canonical schema.

---
