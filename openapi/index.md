---
layout: doc
title: Servirtium and OpenAPI
permalink: /openapi/
desc: How Servirtium tapes and OpenAPI specs relate, and how to use exported HAR files with existing tooling.
eyebrow: Docs
heading: Servirtium and OpenAPI
lede: An OpenAPI document describes the contract. A tape is a recorded instance of real traffic. They are complementary, and existing HAR tooling connects them.
back_url: /features/
back_title: Features
---

## Two different artifacts

OpenAPI says what *can* happen: endpoints, parameters, schemas. A Servirtium tape records what *did* happen: one real conversation, replayable offline and diffable in Git. Servirtium does not read an OpenAPI document and does not check a tape against one. If you want that, the route is the HAR recipe below, using tools that already exist.

## When you own the service

Generate the spec from the code. Reflection-based tools do this (ASP.NET Core's built-in OpenAPI support, Swashbuckle, springdoc, and similar). That spec is authoritative because it comes from the implementation, and Servirtium does not try to compete with it. Tapes are still useful there as worked examples and regression guards.

## When you do not own the service

This is Servirtium's usual case: a vendor's or another team's API. You may have no spec, or a published one you cannot fully trust. The recipe is to export the tapes to HAR and hand the HAR files to tools that read HAR.

### 1. Export each tape to HAR

`servirtium export` converts a tape to a HAR 1.2 file. (The CLI is built from a checkout of the repository; it is not a release download yet. See the [CLI](/#cli).)

```sh
servirtium export reads_and_writes_secrets_with_the_chosen_token.md vault.har
# servirtium: exported reads_and_writes_secrets_with_the_chosen_token.md -> vault.har
```

The tape used here is one of the Vault recordings in
[`pre-recorded/from-testcontainers/vault/`](https://github.com/servirtium/servirtium-vcr/tree/main/pre-recorded/from-testcontainers/vault).
The first entry of the resulting HAR holds the request and the recorded response:

```json
{ "request": { "method": "GET", "url": "http://localhost/v1/sys/health", ... },
  "response": { "status": 200,
                "content": { "mimeType": "application/json",
                             "text": "{\"initialized\":true,\"sealed\":false, ... \"cluster_id\":\"{{uuid-1}}\"}" } } }
```

Do this for each tape you want to include; each export is one HAR file.

### 2. Infer a draft spec from the HAR

[mitmproxy2swagger](https://github.com/alufers/mitmproxy2swagger) reads a HAR file (`-f har`) and writes an OpenAPI 3 document. It works in two passes. The first pass lists the paths it saw, each prefixed `ignore:`; remove the prefix from the ones you want, and run it again:

```sh
mitmproxy2swagger -i vault.har -o spec.yml -p http://localhost -f har
# edit spec.yml: change "- ignore:/v1/sys/health" to "- /v1/sys/health"
mitmproxy2swagger -i vault.har -o spec.yml -p http://localhost -f har
```

For the one path we kept, the second pass produced this (abridged):

```yaml
paths:
  /v1/sys/health:
    get:
      summary: GET health
      responses:
        '200':
          content:
            application/json:
              schema:
                type: object
                properties:
                  initialized: {type: boolean}
                  sealed: {type: boolean}
                  version: {type: string}
                  cluster_name: {type: string}
```

This was run with mitmproxy2swagger 0.10.1. Its `-e` (examples) and `-hd` (headers) options copy more of the traffic into the spec; leave them off unless you have checked what they would copy.

An inferred spec only reflects what was observed. A field that never appeared, an error status the tests never provoked, or a value that is always the same in your recordings will be missing or too narrow. Treat it as a draft to review, not as the provider's contract.

### 3. Compare with the published spec, or see what the tests cover

With a spec in hand, whether inferred or the provider's own, you can use the same HAR files in two ways with HAR-aware tools:

- **Check recorded traffic against the provider's published spec**, using a validator that accepts HAR input or traffic replayed from it.
- **See which operations your tests exercise**, by comparing the paths in the HAR with the operations in the spec. The path list that mitmproxy2swagger prints on its first pass is already that inventory.

This page names only the tool it ran. Other HAR-aware tools exist; check that one accepts HAR before relying on it.

## What is in the HAR

A tape's credential redactions carry through: the exported HAR holds the tape's tokens, not the original secrets. The Vault tape above has `X-Vault-Token: X_VAULT_TOKEN_1` in its requests, and so does the HAR made from it. So a HAR exported from a tape is as safe to share as the tape itself, and no safer: anything the tape still holds (a body with personal data, say) is in the HAR too. Review the tape before you share either.

Two other things come along from the tape: values that are `{{tokens}}` stay as tokens (`{{uuid-1}}` above), and the timing fields in the HAR are placeholders, since a tape does not record timing.

## The other direction

`servirtium import <in.har> <out.md>` goes the other way. A capture from Chrome DevTools, Fiddler, Charles, mitmproxy or a Postman session export becomes a tape, which then replays under any binding or with `servirtium serve`.

```sh
servirtium import capture.har tapes/from_browser.md
```

See the [HAR import and export](/features/#har-import-export) feature for how requests and headers are mapped.
