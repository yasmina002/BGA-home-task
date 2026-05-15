# Blockchain HomeTask — Applicant Instructions

## Overview

You have been given a working blockchain application with a professional layered backend
(Express) and a React frontend. Your job is to extend it with the two features below.

Read the codebase before you start — understanding the existing structure is part of
what we are evaluating.

**Time budget: 4–6 hours.**

---

## Phase 1 — Setup & Code Comprehension

```bash
npm install
cp .env.example .env

# Terminal 1 — React dev server (port 3000)
npm start

# Terminal 2 — API server (port 3002, auto-reload)
npm run dev
```

Open `http://localhost:3000` and verify the UI loads correctly.

### Understand the architecture before coding

| Layer | Location | Purpose |
|---|---|---|
| Config | `config/index.js` | All env-driven settings |
| Domain models | `models/blockchain.js` | Block, Transaction, Blockchain classes |
| Singleton | `models/index.js` | Single shared blockchain instance |
| Utilities | `utils/` | Logger, response builder, validators |
| Middleware | `middleware/` | CORS, request logging, error handler, rate limiter, body validation |
| Routes | `routes/` | One file per resource |
| Controllers | `controllers/` | Business logic called by routes |
| API client | `src/api/` | All frontend HTTP calls go through here |
| Hooks | `src/hooks/` | `useBlockchain` polls the API and owns state |

Be ready to walk through the code in the interview and explain any part of it.

---

## Task 1 — Transaction Fees & Miner Fee Collection

**Goal:** Every user-submitted transfer carries an explicit **fee** paid to the miner. Fees
must be deducted from the sender’s balance and included in the miner’s payout when a block
is mined.

### Domain rules

- Add a numeric **`fee`** field to `Transaction` for normal transfers (`fromAddress` is not
  `null`). Fees must be **≥ 0** (zero fee allowed unless you document a minimum — if you
  add a minimum, make it configurable via `config/index.js`).
- **Coinbase / reward transactions** (`fromAddress === null`) do **not** carry a user fee;
  do not require a `fee` field for them.
- A transaction is only accepted if the sender’s **confirmed** balance can cover
  **`amount + fee`** (use the same notion of “balance” as the rest of the app — pending
  spends should not let an address go negative; match whatever rules the starter already
  uses and extend them for the fee).
- When a block is mined, the miner’s reward for that block must be:
  **`miningReward` (from config) + sum of all `fee` values** from every user transaction
  included in that block (exclude reward transactions from the fee sum).

### Backend

- [ ] Extend `Transaction` (constructor, hash input used for signing/validation if any, and
  serialisation) so fees are represented consistently end-to-end.
- [ ] Update balance logic so outgoing transfers subtract **`amount + fee`** from the
  sender; incoming transfers still credit **`amount`** to the recipient (fees are burned
  from the sender’s perspective and recreated as part of the miner payout — not sent to
  `toAddress`).
- [ ] Update `minePendingTransactions` (or equivalent) so the reward paid to
  `miningRewardAddress` reflects **base reward + collected fees** for that block.
- [ ] Extend `POST /api/transactions` validation (middleware + controller) to accept
  `fee`, validate it, and return clear errors when balance is insufficient.

### Frontend

- [ ] Add a **fee** input to `TransactionForm` (sensible default, e.g. `0`).
- [ ] Show the user an indication of **total debit** (`amount + fee`) where it helps avoid
  confusion.

### Rules

- Follow existing `routes/` → `controllers/` patterns.
- Use `utils/response.js` (`sendSuccess` / `sendError`) for API responses.
- Use `validateBody` where you accept new JSON fields.

---

## Task 2 — Paginated Chain & Confirmed Transaction History

**Goal:** The API and UI must not assume the full chain or full transaction list is small.
Expose **paginated read APIs** and update the React app to browse history in pages.

### Backend

- [ ] Add **`GET /api/chain/blocks`** with query parameters **`offset`** and **`limit`**
  (both non-negative integers; enforce a **maximum `limit`** (e.g. 50) to avoid abuse).
  - Response should include **metadata**: at minimum **`total`** block count (excluding
    genesis vs including it — pick one behaviour, document it in JSDoc, and stay
    consistent), plus the **slice** of blocks for the page.
  - Document the **sort order** (e.g. genesis-first ascending height vs newest-first) in
    JSDoc on the controller or service helper.
- [ ] Add **`GET /api/transactions/confirmed`** (or extend the existing confirmed route if
  you prefer, but keep a clear contract) with the same **`offset` / `limit`** pattern and
  **`total`** count for **confirmed** transactions only (mined in blocks, excluding pending
  pool).
- [ ] Reuse validators / shared parsing for offset/limit so both endpoints behave
  consistently.

### Frontend

- [ ] Update or add API helpers in `src/api/` for the new query-string endpoints.
- [ ] Update the UI so the user can **page through blocks** and/or **confirmed
  transactions** without loading the entire datasets at once (exact UX is up to you —
  simple “Prev / Next” buttons are fine).
- [ ] Handle **loading** and **empty** states without breaking the existing layout.

### Rules

- Do **not** load unbounded full chain or full confirmed transaction arrays in the
  background on every poll if that defeats pagination — adjust polling strategy or fetch
  only what the current page needs.
- New route handlers belong in the appropriate `routes/*.routes.js` and
  `controllers/*.controller.js` files.

---

## Phase 3 — Documentation

- [ ] Add a `## Changes` section to `README.md` describing what you built and any new env
  vars or config knobs.
- [ ] Add JSDoc comments to every new function and class you write.
- [ ] Note any known limitations or trade-offs at the bottom of your `README.md` changes.

---

## Evaluation Criteria

| Area | Weight | What we look at |
|---|---|---|
| **Correctness** | 35% | Does it work end-to-end? Are edge cases handled? |
| **Architecture fit** | 30% | Does the new code follow the existing layered patterns? |
| **Code quality** | 25% | Naming, readability, no dead code, proper error handling |
| **Documentation** | 10% | JSDoc, README update, clear commit messages |

---

## Resources

- [Express — req.query](https://expressjs.com/en/4x/api.html#req.query)
- [MDN — URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams)
- [Node.js crypto](https://nodejs.org/api/crypto.html) (if you touch hashing / signing)

---

**Good luck. We value clean, well-considered code over rushed, complete code.**
