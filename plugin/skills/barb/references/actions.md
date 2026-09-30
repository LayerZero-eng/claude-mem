# Action agents

One agent per job. Each is a file in `<workspace>/.claude/agents/`. They are
dispatched by a customer agent, they do one thing, and they return a receipt:
what they did, the id, the evidence link, finished or not.

None of them may spawn another agent (`tools:` must not include `Agent`).

## The receipt

Every action agent ends with exactly this shape, and the customer agent copies
it into `ledger.md`:

```
action: refund
target: $20.00 / cus_ABC / re_XYZ
result: done | blocked | refused
reason: first auto-upgrade complaint, policy step 1
evidence: <dashboard or repo link>
```

## account-researcher (read-only)

Answers "what is true about this account?" — plan, trial state, usage, sync
health, version in use, and whether the bug they describe is already known.
Read-only: it never changes anything, so it needs no approval gate.
Tools: `Read, Grep, Glob, Bash, WebFetch, WebSearch`.

It searches claude-mem for the customer's project (`barb-<slug>`) and for the
symptom before concluding anything is new.

## refund-runner

Applies the refund ladder in `policy.md` and stops at the owner's ceiling.
Refunds are issued through the Stripe Dashboard, not by writing to the database.
It never decides a refund outside the ladder — those go back as `refused` with
the reason, and the customer agent escalates.

## billing-runner

Cancellations, downgrades, plan swaps, dunning on a failed payment, invoices,
receipts, and tax or VAT details. Same rule: the payment provider's own surface,
never direct database edits, and the change is confirmed to the customer in the
thread afterwards.

## erasure-runner

Runs the product's own account-erasure path — never hand-deleted rows. It
refuses unless the thread contains a verified identity and a written yes, and it
records the erasure id. Data export is the same agent and the same verification,
with nothing deleted.

## dispute-runner

Stripe chargebacks: find open disputes, gather evidence, respond before the
deadline, report status. Deadlines outrank everything else in the queue.

## issue-filer

Files and updates issues on the product repos with `gh`. Rules:

- Search first: `gh issue list --repo <repo> --search "<symptom>" --state all`.
  An existing issue gets a comment, not a duplicate.
- The issue gets the symptom, version, platform and a redacted repro. It never
  gets the customer's name, email, account id or any key they pasted.
- The issue number goes in the customer's ledger and dossier, so release
  follow-up can find them when it ships.

## Filling in the exact commands

The concrete entry points on this machine — the erasure command, the admin
lookup routes, the worker port, the exact product repo slugs — are environment
facts, not skill content. Before the first real run, confirm each one and write
it into the workspace as `ACTIONS-LOCAL.md`, next to `BOARD.md`. Do not guess a
command into a reply, an issue, or a customer's account: an action agent that
cannot confirm its own entry point returns `blocked` and says what it needs.
