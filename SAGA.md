# SAGA Pattern in Distributed Systems (Database as Source of Truth)

## Overview
The **Saga Pattern** is a distributed transaction management approach used in microservices architectures where a single business transaction spans multiple services and databases.

Instead of using a traditional ACID transaction across services, a saga breaks the transaction into a sequence of **local transactions**. Each service updates its own database and publishes an event that triggers the next step.

If a step fails, the saga executes **compensating transactions** to undo the effects of previous successful steps.

---

## Why SAGA?

Distributed systems typically face the following challenges:

- Each service owns its own database.
- Two-Phase Commit (2PC) introduces tight coupling and scalability issues.
- Long-running business transactions cannot hold distributed locks.
- Services may fail independently.

The Saga Pattern provides:

- Eventual consistency
- Better scalability
- Service autonomy
- Failure recovery through compensation

---

## Example: Order Processing System

Consider an e-commerce platform with the following services:

1. Order Service
2. Inventory Service
3. Payment Service
4. Shipping Service

Each service owns its database.

### Business Flow

```text
Place Order
    |
    v
Create Order
    |
    v
Reserve Inventory
    |
    v
Process Payment
    |
    v
Create Shipment
    |
    v
Order Completed
```

---

## Database as Source of Truth

In a saga-based system:

- Each service persists state in its local database.
- The database remains the source of truth.
- Events are derived from committed database changes.
- Other services react to events and update their own databases.

### Recommended Approach: Transactional Outbox Pattern

To avoid inconsistencies between database updates and event publication:

1. Update business tables.
2. Insert event into an Outbox table.
3. Commit both operations in one database transaction.
4. Background publisher reads Outbox records.
5. Publish events to the message broker.
6. Mark events as processed.

```text
+----------------+
| Service DB     |
+----------------+
| Business Table |
| Outbox Table   |
+----------------+
        |
        v
 Event Publisher
        |
        v
 Message Broker
```

---

## Choreography-Based Saga

Services communicate through events.

```text
Order Service
     |
     | OrderCreated
     v
Inventory Service
     |
     | InventoryReserved
     v
Payment Service
     |
     | PaymentCompleted
     v
Shipping Service
```

### Advantages

- No central coordinator
- Loose coupling
- Easy horizontal scaling

### Disadvantages

- Complex event flows
- Harder debugging and observability
- Circular dependencies may emerge

---

## Orchestration-Based Saga

A central orchestrator controls the workflow.

```text
            +----------------+
            | Saga           |
            | Orchestrator   |
            +----------------+
                    |
       +------------+------------+
       |            |            |
       v            v            v
   Order      Inventory     Payment
  Service      Service      Service
```

### Advantages

- Centralized control
- Easier monitoring
- Better visibility into workflow status

### Disadvantages

- Additional component to maintain
- Orchestrator can become complex

---

## Compensation Transactions

Suppose payment fails after inventory reservation.

```text
Order Created          ✓
Inventory Reserved     ✓
Payment Processed      ✗
```

Compensation steps:

```text
Release Inventory
Cancel Order
```

Example:

| Forward Transaction | Compensation |
|--------------------|--------------|
| Create Order | Cancel Order |
| Reserve Inventory | Release Inventory |
| Process Payment | Refund Payment |
| Create Shipment | Cancel Shipment |

---

## Failure Scenarios

### Service Crash

A service crashes after database commit but before publishing an event.

Solution:

- Transactional Outbox Pattern

### Duplicate Messages

Solution:

- Idempotent consumers

### Message Broker Failure

Solution:

- Durable queues
- Retry mechanisms
- Dead-letter queues (DLQ)

### Partial Completion

Solution:

- Compensation transactions/ State Management

---

## Best Practices

1. Keep local transactions short.
2. Design compensating actions during initial design.
3. Use Transactional Outbox.
4. Make consumers idempotent.
5. Add distributed tracing.
6. Store saga state for observability.
7. Avoid synchronous service chains when possible.
8. Define clear failure and retry policies.

---

## When to Use SAGA

Use Saga when:

- Multiple services own different databases.
- Eventual consistency is acceptable.
- Long-running business workflows exist.
- Distributed ACID transactions are impractical.

Avoid Saga when:

- Strong immediate consistency is mandatory.
- The workflow is confined to a single database.
- Compensation logic is impossible or extremely expensive.

---

## Key Takeaway

The Saga Pattern enables reliable distributed transactions by coordinating a sequence of local database transactions rather than a single global transaction. When the database is the source of truth, the combination of **Saga + Transactional Outbox + Idempotent Consumers** is a widely adopted approach for achieving scalability, resilience, and eventual consistency in microservices architectures.
