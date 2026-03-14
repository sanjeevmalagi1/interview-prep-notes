

## Idempotency
An operation is idempotent if performing it multiple times has the same effect as performing it once.

`f(f(x)) = f(x)`

### Idempotency Keys
A unique client-generated key attached to a request.

```
POST /pay
Headers:
  Idempotency-Key: 8f3d92ab-1234-xyz

Body:
  { amount: 100, currency: "INR" }
```

### Deduplication Windows
You don’t store idempotency keys forever. You store them for a limited time window.

Keep keys for 24 hours

After TTL expires → key is removed

### Exactly-Once vs At-Least-Once Delivery