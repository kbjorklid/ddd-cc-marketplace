# Invariant Examples

This document provides examples of how to document business invariants in domain designs.

## Aggregate-Level Invariants

### Order Aggregate Invariants

| ID | Invariant | Notes |
|----|-----------|-------|
| ORDER-1 | Order must have at least one item to be confirmed | Prevents processing of empty orders |
| ORDER-2 | Order total must equal sum of item quantities × unit prices | Ensures pricing consistency |
| ORDER-3 | Order cannot transition to Paid without a Confirmed status | Enforces state machine constraints |
| ORDER-4 | Confirmed orders cannot be modified | Locks order after confirmation |
| ORDER-5 | Draft orders expire after 7 days of inactivity | Business rule for order validity |

### Account Aggregate Invariants

| ID | Invariant | Notes |
|----|-----------|-------|
| ACCT-1 | Account balance cannot go below minimum balance | Overdraft protection rule |
| ACCT-2 | Daily withdrawal limit cannot exceed $10,000 | Fraud prevention measure |
| ACCT-3 | Account must have at least one owner | Ensures accountability |
| ACCT-4 | Closed accounts cannot perform transactions | Final state enforcement |

### Invoice Aggregate Invariants

| ID | Invariant | Notes |
|----|-----------|-------|
| INV-1 | Invoice total must equal sum of line items | Mathematical consistency |
| INV-2 | Invoice number must be unique within organization | Regulatory requirement |
| INV-3 | Invoice due date must be after invoice date | Temporal logic constraint |
| INV-4 | Paid amount cannot exceed invoice total | Prevents overpayment |

## Entity-Level Invariants

### OrderItem Entity Invariants

| ID | Invariant | Notes |
|----|-----------|-------|
| ITEM-1 | Quantity must be greater than zero | Enforced by Quantity value object |
| ITEM-2 | Unit price must be positive | Enforced by Money value object |
| ITEM-3 | ProductId is required | Cannot have line item without product |

### Shipment Entity Invariants

| ID | Invariant | Notes |
|----|-----------|-------|
| SHIP-1 | Shipment must have at least one shipping item | Empty shipments not allowed |
| SHIP-2 | Shipping address must be valid | Validated by Address value object |
| SHIP-3 | Tracking number format must match carrier pattern | Format validation |

## Cross-Aggregate Invariants (Eventual Consistency)

### Customer-Order Consistency

| ID | Invariant | Notes |
|----|-----------|-------|
| CUST-ORD-1 | Customer cannot be deleted if they have active orders | Enforced via eventual consistency |
| CUST-ORD-2 | Customer email changes must propagate to order notifications | Handled by domain events |

### Inventory-Order Consistency

| ID | Invariant | Notes |
|----|-----------|-------|
| INV-ORD-1 | Order items cannot exceed available inventory | Checked via domain service, eventual consistency |
| INV-ORD-2 | Reserved stock must be released when order cancelled | Handled by compensating actions |

## State Machine Invariants

### Order Status Transitions

Valid transitions only:
- Draft → Confirmed
- Confirmed → Paid
- Paid → Shipped
- Shipped → Delivered
- (Any state) → Cancelled (with conditions)

| ID | Invariant | Notes |
|----|-----------|-------|
| ORDER-STATE-1 | Cannot transition from Cancelled to any other state | Terminal state |
| ORDER-STATE-2 | Cannot ship unpaid orders | Process requirement |
| ORDER-STATE-3 | Cannot cancel shipped orders | Operational constraint |
| ORDER-STATE-4 | Cannot modify items after confirmation | Immutable after state change |

## Temporal Invariants

### Subscription Aggregation

| ID | Invariant | Notes |
|----|-----------|-------|
| SUB-1 | Subscription end date must be after start date | Temporal consistency |
| SUB-2 | Subscription cannot end before trial period | Trial period enforcement |
| SUB-3 | Renewal must occur before expiration | Grace period handling |

## Business Rule Invariants

### Discount Application

| ID | Invariant | Notes |
|----|-----------|-------|
| DISC-1 | Percentage discount cannot exceed 100% | Mathematical constraint |
| DISC-2 | Cannot stack more than 3 promotional discounts | Business policy |
| DISC-3 | Discount codes cannot be applied to sale items | Exclusion rule |

### Payment Processing

| ID | Invariant | Notes |
|----|-----------|-------|
| PAY-1 | Payment amount must be positive | Fundamental rule |
| PAY-2 | Payment method must be active | System integration rule |
| PAY-3 | Refund cannot exceed original payment amount | Financial integrity |

## Documenting Invariants

### Best Practices

1. **Be Specific**: Clearly state what must be true
2. **Explain Why**: Use notes column for business rationale
3. **Group by Aggregate**: Organize invariants by what enforces them
4. **Use Structured IDs**: Prefix with aggregate/entity abbreviation
5. **Distinguish Levels**: Separate aggregate vs entity vs cross-aggregate rules

### ID Format

Use `{ABBREVIATION}-{number}` format:
- `ORDER-1`, `ORDER-2` for Order aggregate
- `ITEM-1`, `ITEM-2` for OrderItem entity
- `ACCT-1`, `ACCT-2` for Account aggregate
- `CUST-ORD-1` for cross-aggregate rules

### Notes Column Usage

**Good notes:**
- "Prevents processing of empty orders"
- "Enforced by Money value object validation"
- "Business policy from finance department"

**Unnecessary notes:**
- "Self-evident" (if truly obvious, omit notes)
- "Required" (why it's required is more valuable)
