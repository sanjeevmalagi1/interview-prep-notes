Categories
- Greedy thinking
- State updates
- Simulation problems
- Two-pointer technique
- Basic flow intuition


# Greedy thinking
At every step, you make the locally optimal choice, hoping it leads to a globally optimal solution.
You don’t reconsider previous decisions. No backtracking. No DP table. Just "take the best choice now" and move forward.

## Example Problem #1

### Question
Let's say there are at most 500 bank accounts, some of their balances are above 100 and some are below. How do you move money between them so that they all have at least 100? Just to be clear we are not looking for the optimal solution, but a working one.

Example input:

AU: 80
US: 140
MX: 110
SG: 120
FR: 70

Output:

from: US, to: AU, amount: 20
from: US, to: FR, amount: 20
from: MX, to: FR, amount: 10

### Solution

We have 
- accounts with surplus
- accounts with deficit

So, filter the main object array into 2 separate arrays.
Then Redistribute.
How to Redistribute?
- Take the money from surplus account 
- Transfer it until
  - Surplus becomes 0
  - Deficit becomes 0
- If Surplus becomes 0
  - Move to next surplus account
- If Deficit becomes 0
  - Move to next deficit account

Code:
```Javascript

const LIMIT = 100

const accounts = [
  {
    "country": "AU",
    "balance": 80
  },
  {
    "country": "US",
    "balance": 140
  },
  {
    "country": "MX",
    "balance": 110
  },
  {
    "country": "SG",
    "balance": 120
  },
  {
    "country": "FR",
    "balance": 70
  },
]

function redistribute(accounts) {
  const surplusAccounts = accounts.filter((account) => (account.balance - LIMIT) > 0)
  const deficitAccounts = accounts.filter((account) => (LIMIT - account.balance) > 0)

  var surplusPointer = 0;
  var deficitPointer = 0;

  var transfers = [];

  while (surplusPointer < surplusAccounts.length && deficitPointer < deficitAccounts.length) {
    const { country:surplusCountry, balance: surplusBalance } = surplusAccounts[surplusPointer]
    const { country:deficitCountry, balance: deficitBalance } = deficitAccounts[deficitPointer]

    const amount = Math.min(surplusBalance - LIMIT, LIMIT - deficitBalance)

    transfers.push({
      from: surplusCountry,
      to: deficitCountry,
      amount,
    })

    surplusAccounts[surplusPointer].balance -= amount;
    deficitAccounts[deficitPointer].balance += amount;

    if (surplusAccounts[surplusPointer].balance == LIMIT) {
      surplusPointer++
    } 

    if (deficitAccounts[deficitPointer].balance == LIMIT) {
      deficitPointer++
    }

  }

  return transfers
}
```



## Problem #2

### Question
Ledger Balance

You are building a simplified ledger system for a financial platform.

You are given a list of transactions. Each transaction is represented as a tuple:
(type, user_id, amount)

Where:
type is a string and can be:
- "deposit"
- "withdraw"
user_id is a string representing the user
amount is a positive integer

Requirements
Implement a function that processes the transactions in order and returns:

- Final balances for each user
- Failed transactions

Rules

Every user starts with a balance of 0.
- "deposit":
 Add the amount to the user's balance.
- "withdraw":
 Subtract the amount from the user's balance.
- If the user does not have enough balance, the transaction fails.
- Failed transactions must NOT modify the balance.
- Transactions must be processed sequentially in the order given.
- Assume all amounts are positive integers.
- Ignore invalid transaction types (optional extension: treat them as failed).

### Solution

```JavaScript
const transactions = [
  ["deposit", "user1", 100],
  ["withdraw","user1", 30],
  ["withdraw","user1", 80],
]

function balanceLedger(transactions) {
  const res = {}
  const transactionRes = []
  
  transactions.forEach(transaction => {
    if (transaction[0] == "deposit") {
      res[transaction[1]] = res[transaction[1]] ? res[transaction[1]] + transaction[2] : transaction[2]
      transactionRes.push([...transaction, true]);
    }

    if (transaction[0] == "withdraw") {
      const accountBalance = res[transaction[1]]

      if (!accountBalance) {
        transactionRes.push([...transaction, false]);
        return
      }
      
      const updatedBalance = accountBalance - transaction[2]
      if (updatedBalance < 0) {
        transactionRes.push([...transaction, false]);
        return
      }

      res[transaction[1]] = updatedBalance
      transactionRes.push([...transaction, true]);
    }
  });

  return { res, transactionRes }

}

(() => {
  const expectedResults = [
    ["user1", 70]
  ];

  const transactionsRes = [
    ["deposit", "user1", 100, true],
    ["withdraw", "user1", 30, true],
    ["withdraw", "user1", 80, false],
  ]
  const res = balanceLedger(transactions)

  console.log(res)
})()
```

## Problem #3

### Question
Duplicate Charge Detection

You are building a fraud detection system for a payment processing platform. Occasionally, duplicate charges occur due to network retries, payment gateway timeouts, or client-side resubmissions.

Your task is to detect duplicate charges from a list of transactions.

You are given a list of transactions. Each transaction is represented as a tuple:

(user_id, amount, date)

Where:

user_id is a string

amount is a positive integer

date is a string in "YYYY-MM-DD" format

Example:

[
  ("user1", 100, "2026-02-01"),
  ("user1", 100, "2026-02-01"),
]

Output
Return a list of transactions that are duplicates.

A transaction is considered a duplicate if:

Another transaction exists with the same:

user_id

amount

date

If a transaction appears more than once, all occurrences after the first should be returned as duplicates.

Output
[
  ("user1", 100, "2026-02-01"),
]

### Solution

```Javascript

const transactions = [
  {
    "user": "user1",
    "amount": 100,
    "date": "2026-02-01"
  },
  {
    "user": "user2",
    "amount": 200,
    "date": "2026-02-01"
  },
  {
    "user": "user1",
    "amount": 100,
    "date": "2026-02-01"
  },
  {
    "user": "user1",
    "amount": 100,
    "date": "2026-02-02"
  },
  {
    "user": "user2",
    "amount": 200,
    "date": "2026-02-01"
  },
]

function resolveDuplicates(transactions) {
  const res = {}
 
  const response = transactions.filter(transaction => {
    const compositeKey = getCompositeKey(transaction)

    if (res[compositeKey]) {
      return false
    }

    res[compositeKey] = true
    return true
  });

  return response
}

function getCompositeKey(transaction) {
  return `${transaction.user}-${transaction.date}-${transaction.amount}`
}

(() => {

  const res = resolveDuplicates(transactions)

  console.log(JSON.stringify(res))
})()

```

## Problem #4 Design a parking system in code


## Problem #5 Command-Based Wallet System

### Question
Part 1 – Basic Commands

process(commands: string[]): string[]

CREATE user1
DEPOSIT user1 100
TRANSFER user1 user2 30
BALANCE user1

Rules:

- Users start with 0 balance.
- TRANSFER fails if insufficient funds.
- BALANCE returns "user1 70"


Return output only for:
- BALANCE
- Failed operations (e.g. "ERROR insufficient_funds")


