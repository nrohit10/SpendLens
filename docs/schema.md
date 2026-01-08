# Canonical Data Schema

## Overview

This document defines the canonical data schema used by **SpendLens**.
The schema is intentionally designed as a **single physical table** to
support simplicity in early development while remaining flexible enough
to enable advanced analytics, recurrence detection, and AI-driven insights.

Each row in the table represents **one financial transaction** derived
from bank statements, credit card statements, or UPI records.

---

## Table: expenses

### Purpose

Stores normalized, transaction-level financial data that serves as the
foundation for all analytics, insights, and AI features in the system.

---

### Field Definitions

#### Identifiers

| Field Name     | Type   | Description                                                  |
| -------------- | ------ | ------------------------------------------------------------ |
| transaction_id | STRING | Unique identifier for the transaction (generated or derived) |
| serial_no      | STRING | Statement serial or reference number, if available           |

---

#### Dates

| Field Name       | Type | Description                                                        |
| ---------------- | ---- | ------------------------------------------------------------------ |
| transaction_date | DATE | Date on which the transaction occurred                             |
| posted_date      | DATE | Date on which the transaction was posted (useful for credit cards) |

---

#### Description

| Field Name      | Type   | Description                                           |
| --------------- | ------ | ----------------------------------------------------- |
| raw_description | STRING | Original transaction description from the statement   |
| merchant_name   | STRING | Normalized merchant name derived from raw_description |

---

#### Amount & Direction

| Field Name | Type          | Description                                    |
| ---------- | ------------- | ---------------------------------------------- |
| amount     | DECIMAL(12,2) | Transaction amount (always stored as positive) |
| currency   | STRING        | Currency code (default: INR)                   |
| direction  | STRING        | Indicates debit or credit (debit / credit)     |

---

#### Payment Instrument

| Field Name       | Type   | Description                                                                        |
| ---------------- | ------ | ---------------------------------------------------------------------------------- |
| transaction_type | STRING | Mode of transaction (credit_card / upi / bank_transfer)                            |
| card_provider    | STRING | Card issuer or bank name (e.g., HDFC, ICICI, HSBC); NULL for non-card transactions |

---

#### Categorization (Derived)

| Field Name    | Type    | Description                                                  |
| ------------- | ------- | ------------------------------------------------------------ |
| category      | STRING  | High-level expense category (e.g., Food, Travel, Investment) |
| subcategory   | STRING  | Optional finer-grained classification                        |
| is_investment | BOOLEAN | Indicates whether the transaction represents an investment   |

---

#### Recurrence & Analytics Flags (Derived)

| Field Name            | Type    | Description                                                 |
| --------------------- | ------- | ----------------------------------------------------------- |
| is_recurring          | BOOLEAN | Indicates if the transaction is part of a recurring pattern |
| recurrence_confidence | FLOAT   | Confidence score for recurrence detection (0.0 – 1.0)       |

---

#### Rewards (Optional)

| Field Name    | Type    | Description                                             |
| ------------- | ------- | ------------------------------------------------------- |
| reward_points | INTEGER | Reward points earned for the transaction, if applicable |

---

#### Metadata

| Field Name | Type   | Description |
| ---------- | ------ | ----------- |
| source     | STRING | Origin      |
