Design a fast KV store similar to redis
- set
- get
- delete
- transaction (commit, rollback)


## Requirements
- Primary Capabilities
  - set(key, value)
  - get(key)
  - delete(key)
  - Transaction
    - begin
    - commit
    - rollback

## Rules and Completion
- Keys are unique
- Overwrite existing keys
- Transactions are isolated per session 
- Nested Transaction supported
- Reads inside transaction get uncommitted changes
-

## Error Handling
- `get` on missing key -> return null/error
- `delete` on missing key -> return null/error
- `commit` without transaction -> error
- `rollback` without transaction -> error

## Scope
- In-memory 
- Single Node (No distrubuted)
- No eviction policy 
- No TTL

## Entities

### KeyValueStore (Orchistrator)
- Main interface exposed to users
- Coordinates operations + transaction handling

### Storage
- Stores key-values
- Responsible for read/writes

### TransactionManager
- Handles transaction lifecycle
- Maintains stack transactions

### Transaction
- Represents a single transaction 
- tracks changes 

## Relationships

### KeyValueStore
- has-a: Storage
- has-a: TransactionManager

### TransactionManager
- has-many:  Transaction

### Transaction
- contains: local changes

## Class Design

KeyValueStore
- State
  - storage: Storage
  - txnManager: TransactionManager
- behaviour
  - set(key, value)
  - get(key)
  - delete(key)
  - begin()
  - commit()
  - rollback()

Storage
- State
  - store: Map<String, String>
- behaviour
  - set(key, value)
  - get(key)
  - delete(key)

TransactionManager
- State
  - txnStack: Stack<Transaction>
- behaviour
  - begin()
  - commit(storage)
  - rollback()
  - currentTransaction()

Transaction
- state
  - writes: Map<String, Optional<String>>
- behaviour
  - set(key, value)
  - delete(key)
  - get(key, fallbackStorage)
  - merge(childTxn) (for nested txn)

## Implementation
```JavaScript
class KeyValueStore {
  constructor() {
    this.storage = new Storage()
    this.txnManager = new TransactionManager()
  }

  put(key, value) {
    this.storage.put(key, value)
  }

  get(key) {
    return this.storage.get(key)
  }

  delete(key) {
    this.storage.delete(key)
  }
}

class Storage {
  constructor() {
    this.store = new Map(); 
  }

  put(key, value) {
    this.store.set(key, value);
  }

  get(key) {
    return this.store.has(key) ? this.store.get(key) : null;
  }

  delete(key) {
    this.store.delete(key);
  }

  begin() {
    this.txnManager.begin();
  }

  commit() {
    this.txnManager.commit(this.storage);
  }

  rollback() {
    this.txnManager.rollback();
  }
}

class TransactionManager {
  constructor() {
    this.txnStack = [];
  }

  begin() {
    this.txnStack.push(new Transaction());
  }

  current() {
    if (this.txnStack.length === 0) return null;
    return this.txnStack[this.txnStack.length - 1];
  }

  commit(storage) {
    if (this.txnStack.length === 0) {
      throw new Error("No active transaction");
    }

    const txn = this.txnStack.pop();

    if (this.txnStack.length == 0) {
      for (const [key, value] of txn.writes.entries()) {
        if (value === null) {
          storage.delete(key);
        } else {
          storage.put(key, value);
        }
      }

      return
    }

    const parent = this.txnStack[this.txnStack.length - 1];
    parent.merge(txn);
    return
  }

  rollback() {
    if (this.txnStack.length === 0) {
      throw new Error("No active transaction");
    }

    this.txnStack.pop();
  }
}

class Transaction {
  constructor() {
    this.writes = new Map();
  }

  set(key, value) {
    this.writes.set(key, value);
  }

  delete(key) {
    this.writes.set(key, null);
  }

  get(key, storage) {
    if (this.writes.has(key)) {
      return this.writes.get(key);
    }
    return storage.get(key);
  }

  merge(childTxn) {
    for (const [key, value] of childTxn.writes.entries()) {
      this.writes.set(key, value);
    }
  }

}

```

