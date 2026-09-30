# Silo templates

One folder per customer, under `<workspace>/customers/<slug>/`. The slug is the
account email with `@` and `.` replaced by `-` (`ada@acme.io` → `ada-acme-io`).
Create the folder the first time that customer is heard from, and never delete
it — a returning customer must find their own history.

```
customers/ada-acme-io/
  dossier.md      who they are and where things stand   (rewritten in place)
  ledger.md       every action taken                    (append only)
  threads/        one file per email thread, raw        (append only)
  drafts/         replies written but not yet sent
```

## dossier.md

Kept current: rewrite the sections, don't append. Keep it under a page — it is
read at the start of every interaction.

```markdown
# Ada Lovelace — ada@acme.io

- Account: <cmem user id>   Plan: <plan / trial state>   Since: <date>
- Stripe customer: <cus_...>
- Slug: ada-acme-io
- Flags: <vip | churn-risk | refund-given | legal | none>

## Where things stand
One paragraph, current. What they last asked, what we owe them, what they owe us.

## Open
- [ ] <thing> — owed by <date> — <thread file>

## History
- 2026-09-14 Refunded $20 auto-upgrade (first complaint, policy step 1).
- 2026-09-02 Filed thedotmack/claude-mem#4102 for their sync hang; fixed in 13.24.0, told them.

## Known about them
Their setup, their platform, the things already ruled out. This is what stops
the next conversation starting from zero.
```

## ledger.md

Append only. One line per action that touched money, data, a public repo or the
customer's account. This is the audit trail; nothing that changes state is
allowed to skip it.

```
2026-09-14T18:20Z | refund | $20.00 | re_1Abc... | first auto-upgrade complaint, policy step 1 | agent: refund-runner
2026-09-14T18:22Z | reply-sent | thread 019a | refund confirmed, asked about their sync setup | agent: customer-ada-acme-io
2026-09-15T09:03Z | issue-filed | thedotmack/claude-mem#4102 | sync hang on Windows, repro redacted | agent: issue-filer
```

Fields: UTC timestamp, action, target/amount, id or link, one-line reason,
which agent did it.

## threads/

One file per email thread, named `<date>-<subject-slug>.md`. Paste the thread as
it arrived, newest at the bottom, secrets redacted as `[REDACTED: api key]`.
Drafts live in `drafts/` until sent, then the sent copy is appended to the
thread file and the draft deleted.

## Memory

After each interaction, save one short note to claude-mem so future sessions can
search it, using the customer's own project name:

```bash
curl -s localhost:37700/api/memory/save \
  -H 'content-type: application/json' \
  -d '{"project":"barb-ada-acme-io","title":"Refund + sync hang","text":"<two or three sentences>"}'
```

The port is whatever the local worker runs on; read it from the user's
claude-mem settings rather than assuming. The dossier is the working copy;
claude-mem is the searchable history.
