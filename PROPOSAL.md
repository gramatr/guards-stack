# PROPOSAL: community telemetry + governance telemetry (two separate tracks)

**Status: proposal only. Nothing in this document is implemented.** No
code has been written against it, no license text has changed, and no
data is being collected anywhere as a result of this doc. It exists to
capture a design discussion in writing before anyone (legal, product,
or an engineer) acts on it.

## Why this exists

`guards-stack` is open-sourced as a trust signal — grāmatr's concrete
commitment to protecting developers, not just a slogan (see README).
Two further ideas came out of that: could the community that runs this
stack benefit from pooling what it collectively catches, and could
grāmatr's own proprietary product (`cpp-dwa`) use the same underlying
signal internally to understand where risk concentrates. Those are
**two different systems with two different consent models**, and the
biggest risk in building either one is building them as if they were
the same thing. This document exists specifically to keep them apart.

## Track A — `guards-stack` community telemetry (opt-in, value exchange)

**Who this is for**: anyone self-hosting `guards-stack` independently,
with no relationship to `cpp-dwa` or any grāmatr product. The default
is **no collection at all** — this is an explicit opt-in program, not a
default behavior.

**What would be collected, if a deployer opts in**: aggregate counts
only —

```
{finding_type: secret_leak | pii | injection, pattern_family, time_bucket}
```

`pattern_family` is a category, not a value — e.g. "obfuscated/base64
injection attempt," "AWS-shaped credential," "SSN-shaped number" — the
same "report the shape of the finding, never the finding itself"
discipline every service in this stack already follows. **Never**: raw
text, the matched secret/PII value, actor identity, session identity,
or anything that could identify a specific deployer or their users.

**The value exchange**: an opted-in deployer gets access to a curated
community security report back from grāmatr — trending injection
pattern families, common secret types being caught across the
community, that kind of aggregate threat-intel digest. Give a little
anonymous signal, get useful signal back. This is a well-precedented
pattern in security tooling (crowdsourced threat-intel sharing), not a
novel one.

**What this explicitly is NOT**: a public leaderboard naming specific
vendors, models, or harnesses. That idea was raised and deliberately
rejected during this design discussion — real defamation-adjacent and
reputational risk, plus a representativeness problem (an opt-in sample
is not a fair basis for ranking anyone). If "top violating X" framing
ever resurfaces, route it back through this same rejection, not around
it.

**Open questions, not yet answered** (need product + legal input before
any of this is built):
- Where the opt-in consent language lives — a separate, explicit
  data-sharing notice a deployer actively accepts, **not** inside
  `LICENSE`. A software license governs code reuse; it is not the
  right vehicle for a data-collection agreement, and conflating them
  would weaken both.
- Who operates the collector infrastructure, and under what retention
  policy.
- What, specifically, goes into a "community security report," how
  often, and in what format.
- A minimum-sample-size floor before any aggregate is shown back to
  the community, so a thin pattern_family/time_bucket cell can't be
  reverse-engineered to a specific deployer's traffic.

## Track B — `cpp-dwa` internal governance telemetry (proprietary, not opt-in in the community sense)

**Who this is for**: cpp-dwa's own paying tenants, under cpp-dwa's
existing commercial ToS/DPA relationship. This is not a volunteer
program — consent mechanics here follow normal enterprise
data-processing terms, not a community opt-in flow.

**What it would capture**: rollup counters, not per-event actor logs by
default —

```
counts by {finding_type, harness, model, time_bucket}
```

The governance layer *could* know which specific actor/session
triggered a given finding (it has that context, guards-stack never
does), but the current proposal is deliberately scoped to counts, not
individual attributed events — "how many, by type, by harness" is the
stated goal, not "audit a specific actor."

**Purpose**: internal product decisions only — e.g. prioritizing which
harness needs a guard adapter built next. **Not published, not fed into
Track A's community reports** unless it goes through the exact same
anonymization and minimum-sample treatment Track A requires. That
firewall is the whole point of keeping these as two tracks instead of
one: cpp-dwa customer data should never leak into the open community
program's dataset through a side door.

## What this document does not decide

- Whether either track ships at all.
- Any license text change (the MIT boilerplate in `guards-stack` stays
  exactly as published).
- Any specific collector architecture, API, or storage choice.
- Any public-facing ranking or named comparison of models/harnesses —
  explicitly out of scope, see Track A above.

## Next step

Both tracks need a product + legal pass before either moves from this
document into actual design work — Track A because it's a new
public-facing data-collection surface with its own consent UX, and
Track B because it's a new category of data cpp-dwa's existing ToS/DPA
may or may not already cover and would need to be checked against.
