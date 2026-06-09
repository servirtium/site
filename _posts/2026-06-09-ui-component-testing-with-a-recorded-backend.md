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

## A worked example

There's a small demo in the monorepo —
[`integration/vue-storybook`](https://github.com/servirtium/servirtium-vcr/tree/main/integration/vue-storybook)
— that puts the two ideas together:

- A **Vue** control: a form that does an HTTP **POST** and renders the created
  record. One source file, `PostForm.vue`.
- That *same* control is **presented in Storybook** (the tool that made
  "detaching a UI from the stack" routine — [browse it live here]({{ site.baseurl }}/storybook/))
  **and** driven, outside Storybook, in a **Selenium** harness in real headless
  Chrome. Both import the one component — no parallel widget, no doubled test.
- The **mock backend is Servirtium**. The VCR serves the built page from its own
  static-content mount, so the form's `POST` is *same-origin* (no CORS, no
  preflight) and lands straight on the VCR — which either **records** it
  (forwarding once to a throwaway upstream) or **replays** it from a committed
  Markdown tape. Record vs playback is just a choice of which script you run.

So the control runs disconnected from any backend, the test is offline and
deterministic, and it's fast — exactly the 2017 brief, with the down-stack cut
at the network rather than the model.

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
