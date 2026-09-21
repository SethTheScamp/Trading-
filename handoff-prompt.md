# Handoff Prompt — pulling account state from a chat where Robinhood works

The scheduled routine currently has no Robinhood connector (see the **Setup
Requirement** section of `log.md`). Until that is fixed in the routine's
settings, account data has to come in by hand.

Paste the block below into a chat **that has the Robinhood connector enabled**.
It is read-only: it pulls state and prints it, and places no orders.

Paste the output back into the trading session (or commit it yourself) and it
goes straight into `log.md`.

---

## Paste this

````text
You have the Robinhood connector enabled. I need a read-only snapshot of my
Robinhood "Agentic" account — that account ONLY, no other account.

Do not place, modify, or cancel any order. Do not preview or stage an order.
This is a data pull only.

Please do three things:

1. List the exact names of the Robinhood tools available to you, one per line.
   I need the literal tool names, not descriptions.

2. Pull the current state of the Agentic account:
   - total account value
   - cash balance
   - buying power
   - every open position: symbol, stock vs option, (for options) strike,
     expiration and call/put, quantity, average entry price, current price,
     unrealized P&L
   - every open/pending order

3. Output the result as a single fenced markdown block in exactly this format,
   filling in real numbers and leaving any field you genuinely cannot retrieve
   as the literal word UNAVAILABLE (do not estimate, do not substitute a
   plausible value):

```
ACCOUNT SNAPSHOT
timestamp_utc: <YYYY-MM-DD HH:MM UTC>
account: Agentic
account_value: <number>
cash: <number>
buying_power: <number>

POSITIONS
<symbol> | <stock|option> | <strike+expiry+call/put, or "-" for stock> | qty <n> | entry <price> | current <price> | pnl <amount> (<percent>)
<one line per position; write "none" if there are no positions>

OPEN ORDERS
<symbol> | <buy|sell> | <type> | qty <n> | limit <price> | status <status>
<one line per order; write "none" if there are no open orders>

TOOLS AVAILABLE
<one literal tool name per line>
```

Accuracy matters more than completeness here: this snapshot becomes the
baseline account value and all-time high for an automated trading log, so a
wrong number is worse than UNAVAILABLE.
````

---

## If that other session is a Claude Code CLI session

A claude.ai chat cannot edit a routine. A **local Claude Code CLI** session can:

```
/schedule update
```

Then ask it to add the Robinhood connector to the trading routine. This fixes
the root cause permanently, unlike the snapshot above, which is a one-time
manual import. `/schedule` is unavailable inside cloud sessions, so this only
works from a terminal.

## What to do with the output

Hand the `ACCOUNT SNAPSHOT` block back to the trading session. It becomes:

- **All-time high account value** and **Day's starting value** in Standing
  State (first real snapshot sets both)
- The **Positions** line of the next log entry
- The recorded tool names, so a later run knows what to call the moment the
  connector reaches the routine

## Limits this does NOT lift

A manual snapshot is a point-in-time reading, not live access. Without the
connector in the routine, a scheduled run still cannot re-check prices, verify
fills, or run the required order preview before a trade. So this unblocks
**record-keeping and paper analysis only** — it does not make live order
placement safe or permitted. Fixing the routine's connector list remains the
real fix.
