# Servirtium market-fit notes — competing *alongside* the Postman ecosystem

Internal strategy note. **Not published** (Jekyll excludes `docs/`). Captures
thinking on derived products that attack the Postman ecosystem from an angle
Servirtium's shape makes defensible — not the core "record/replay for tests"
pitch, which the site already covers.

## Framing: don't fight Postman head-on

Postman isn't a tool, it's a **hosted + collaborative + discoverable** platform
(collections, environments, mock servers, monitors, the public API Network, the
"Run in Postman" button, enterprise governance). Its moat is the network effect
and the cloud control plane.

Servirtium is the opposite shape: a **plain-Markdown file in Git, no server, no
account, no network**. Out-Postmanning Postman is a loser. The wedge has to come
from where Servirtium's shape is a strength Postman *structurally can't copy*:

- the tape is **real captured traffic**, not hand-authored examples;
- it's **plain Markdown in Git** — diffable, reviewable, renders on the portal;
- it's the artifact teams **already review** in PRs.

## The two bets we're pursuing

Chosen directions: **mock-from-reality** (the differentiated product) and the
**Postman importer/bridge** (the adoption on-ramp that feeds it). They pair:
the bridge siphons traffic out of Postman; mock-from-reality is what it lands in.

### Bet 1 — Postman importer / bridge (the on-ramp)

A `postman-collection ⇄ Servirtium tape` converter.

- **Why it works:** it doesn't compete, it *siphons*. Teams keep exploring in
  the Postman GUI, then "freeze this exchange as a replayable tape" for CI.
  Servirtium becomes the **test-time layer beneath** the Postman exploration
  layer rather than a rival to it.
- **Shape:** CLI / library. `servirtium import collection.json` → tapes;
  `servirtium export tape.md` → a collection (best-effort, for round-tripping).
- **Adoption flywheel:** low-glory, high-adoption. Every Postman team is a
  potential entry point; the bridge is the cheapest thing to build and the
  thing most likely to get Servirtium into a repo it isn't in yet.
- **Honest limits:** collections carry scripts/auth/env logic a tape has no
  concept of. The import is "capture the request/response shape", not "port the
  Postman runtime." Be explicit that exploration stays in Postman.

### Bet 2 — Mock-from-reality (the differentiated product)

Point at a recorded conversation, get an instant mock that is **byte-identical
to production**, versioned in Git, diffable.

- **The pitch:** *"Your mocks are fiction; ours are recordings."* Postman mock
  servers are hand-authored example responses that drift from reality and live
  in Postman's cloud. A mock built from a real capture cannot drift, because it
  *is* the capture.
- **Why Postman can't copy it:** the core asset is a corpus of real, diffable
  recordings. Postman would have to become Servirtium to match it.
- **Shape:** the engine already serves playback locally (one server per port).
  The product layer is packaging + ergonomics: `servirtium mock ./tapes` →
  a local URL; optionally an ephemeral hosted mock for sharing a branch.
- **On-ramp from Bet 1:** import a Postman collection → tapes → instant mock.
  That's a single demoable flow that starts in a world the user already knows.

## Reinforcing the existing site story

Both bets lean on the **vendor DX / market-share** argument already on the
About page: a challenger vendor ships real, replayable tapes to win developer
goodwill; the importer + mock-from-reality flow is how a client consumes them.
The cross-language angle is the differentiator vs the single-runtime
"Run in Postman" button.

## Considered and parked (not chosen, kept for context)

- **Drift-detection SaaS** — re-record against the live API on a schedule and
  surface semantic drift as a `git diff` PR (the TCK idea productized as
  monitoring). Highest recurring-revenue potential and the play Postman Monitors
  structurally can't match (they ping up/down; this diffs *meaning*). Parked for
  now but the strongest standalone-business candidate if the two chosen bets
  validate demand.
- **Example marketplace / "Replay these tapes" button** — a registry of recorded
  vendor scenarios that render on GitHub and replay in any language; the
  cross-language, Git-native analog of "Run in Postman." Cheapest to seed and
  reinforces the vendor-DX story.

## Where Servirtium honestly cannot compete

- Interactive exploration / the request-builder GUI — Postman owns this.
- Real-time collaboration & enterprise governance console — wrong shape for a
  file-in-Git tool.
- Design-first mocks invented *before* any real traffic exists — that's
  OpenAPI / TypeSpec territory (already acknowledged on the site).
