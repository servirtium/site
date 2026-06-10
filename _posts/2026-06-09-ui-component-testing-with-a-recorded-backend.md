---
layout: post
title:  "UI Component Testing, with a Recorded Backend"
date:   2026-06-09
keywords: "component testing, storybook, selenium, vue, servirtium"
categories: [component-testing]
icon: fa-rss
---

Back in 2017, Paul Hammant argued — in [UI Component Testing](https://paulhammant.com/2017/02/01/ui-component-testing/)
— for something that's since become mainstream: stop driving whole pages through
Selenium for everything. Test the **smallest rectangle** you reasonably can — the
control — in a **throwaway harness that never ships to production**, cut off from
the slow down-stack, fast enough to run *whole tests* by the second. Storybook,
Cypress Component Testing and Playwright Component Testing all landed in that
frame in the years after. (He [revisited the idea in 2025](https://paulhammant.com/2025/06/17/ui-component-testing-revisited/).)

Every component test has to decide one thing: **where to cut the down-stack.**
There are two seams.

- **The model seam.** Shove data straight into the UI's model and skip the
  network — Protractor's back door, or Paul's
  [ngWebDriver](https://github.com/paul-hammant/ngWebDriver) `mutate(...)`. Fast
  and low, but it bypasses the control's real wire behaviour.
- **The HTTP seam.** Let the control make its *actual* request, and answer it
  from a recording. That's where **Servirtium** fits — a recorded HTTP
  conversation in Markdown, captured once against a real (or stub) backend and
  replayed forever, offline and deterministic.

## Two Vue controls, one recorded backend

The demo lives in the monorepo at
[`integration/vue-storybook`](https://github.com/servirtium/servirtium-vcr/tree/main/integration/vue-storybook).
There are two small Vue controls, and for each one the *same* source component is:

- **presented in Storybook** — the tool that made "detach the control from the
  stack" routine ([browse the stories]({{ site.baseurl }}/storybook/)); and
- **driven outside Storybook, in a Selenium harness**, in real headless Chrome.

One component, two consumers — no parallel widget, no doubled test. And in both
cases the mock backend is a **Servirtium VCR**: it serves the built page from
its own static-content mount, so the control's request is *same-origin* (no
CORS, no preflight) and lands straight on the VCR — which either **records** it
(forwarding once to a throwaway upstream) or **replays** it from a committed
Markdown tape. Record vs playback is just which script you run.

### A plain POST: the message form

`PostForm.vue` is the minimal case — a "Leave a message" field and a **Post**
button. Submitting fires a `POST /api/messages` and the control renders the
server's reply: *Created #1: "hello from servirtium" (created)*. One request,
one response, round-tripped through a tape.

- **Play with it live:**
  [Servirtium / PostForm](https://servirtium.dev/storybook/?path=/story/servirtium-postform--default)
  — a Storybook decorator stubs `fetch` with the same response shape the tape
  carries, so the control is live in Storybook too.
- **The recording:**
  [`tapes/post.md`](https://github.com/servirtium/servirtium-vcr/blob/main/integration/vue-storybook/tapes/post.md)
  — a single interaction; the POST body and the JSON reply sit right there in
  the Markdown, diffable in a pull request.

This is the "is a POST even mockable at the HTTP seam?" proof, and the answer is
yes: the control runs disconnected from any backend, the test is offline and
deterministic, and it's fast — exactly the 2017 brief, with the down-stack cut
at the network rather than the model.

### "Good, Cheap, Fast — pick two"

The second control makes a sharper point: the **backend is the source of
truth**, and the UI renders only what the server says — never an optimistic
local flip. It's the old engineering joke as three checkboxes over a Venn
diagram. Every toggle is a POST; pick two and the server names the pair (Slow /
Expensive / Low Quality); reach for a **third** and the server **evicts the
oldest** — you watch a ticked box pop back off. The control can't hold all three
because the backend won't let it.

- **Play with it live:**
  [Servirtium / GoodCheapFast](https://servirtium.dev/storybook/?path=/story/servirtium-goodcheapfast--default)
  — a stubbed backend enforces the rule, so you can click the joke for yourself.
- **The recording:**
  [`tapes/triple-pick-two.md`](https://github.com/servirtium/servirtium-vcr/blob/main/integration/vue-storybook/tapes/triple-pick-two.md)
  — three interactions, the last one dropping `good` so `cheap`+`fast` win.

Because the eviction is the *server's* decision, recorded and replayed, the
control's **unchecking** behaviour is tested for free: the same Markdown tape
that drives the green ticks also drives a box back to empty. A model-seam test
that shoved state straight into the component would never have exercised that
round-trip.

## Why the extra inch is worth it

Cutting at the HTTP seam costs you a real request that the model seam skips. In
return you exercise the control's **actual wire behaviour** — the POST body it
builds, the headers it sends, the response it parses — against a conversation
that was *once a real backend*. And because the recording is Markdown:

- it's **human-inspectable** and renders nicely on GitHub;
- it's **git-diffable** — when a vendor sneaks in a new header, the next
  recording shows it as a red/green line in a pull request;
- it's **shareable across teams and languages** — the same tape replays under
  any Servirtium binding, so a control recorded by a JavaScript test can be the
  fixture a Java or Python service test plays back.

None of that is new in spirit; it's the 2017 component-testing argument with the
boundary moved to where a lot of real flakiness actually lives — the network —
and a recording format built for being read, reviewed and shared. Which is the
job Servirtium was made for.
