---
layout: post
title:  "Servirtium 2.0.0-alpha.20"
date:   2026-11-24
keywords: "servirtium-vcr, release, ntlm, dotnet framework, commit, rewrite upstream base"
categories: [release]
icon: fa-rss
---

[Servirtium 2.0.0-alpha.20](https://github.com/servirtium/servirtium-vcr/releases/tag/v2.0.0-alpha.20) is out.
Most of it came from a .NET Framework 4.8 user's field report: a test suite recording and replaying a SOAP client
on Windows, which found real gaps in connection-bound authentication and in URLs a service writes into its own responses.

## Breaking: a recording is written only by Commit

Servirtium 2.x keeps no deliberate backward compatibility, and this release removes what was left, in every binding at once.
A record VCR used to write its tape when it was closed, disposed or dropped. Now only an explicit
`Commit()` writes the tape, and `Discard()` drops the recording. A record VCR closed with neither writes nothing
and raises, with "record VCR closed without commit or discard, so no tape was written". A test that fails part-way, or
forgets to commit, can no longer mint a tape that later replays as the truth. The scoped helpers in each language
(`with` in Python, `usingRecord` in F#, the block forms in Ruby and Crystal, and so on) discard the recording when their
block raises. Playback is unchanged. Rust's `finish()` is now `commit()`.

`MatchJsonBody` and `MatchXmlBody` are gone as well; `PrettyJson(n)` and `PrettyXml(n)` on a playback builder turn on the same
canonical request-body matching. The [feature reference]({{ site.baseurl }}/features/#commit-discard) has the current API.

## NTLM and Negotiate record correctly

Record mode now pairs each client connection with an upstream connection of its own. Connection-bound authentication
(NTLM, and Negotiate carrying NTLM, as with IIS Windows Authentication, TFS, SharePoint and intranet WCF) used to go wrong
when two clients' handshakes shared one pooled upstream connection. The `pre-recorded/ntlm` directory in the repository has
tapes of each leg of the handshake and a README that sets out what record and playback do with them. See
[NTLM and Negotiate]({{ site.baseurl }}/features/#ntlm-and-negotiate).

## rewrite_upstream_base and the VCR's own address

A service that writes its own absolute URLs into responses (TFS's location service, a HATEOAS API, a paging `nextLink`)
no longer sends the client around the recorder. With the opt-in `rewrite_upstream_base`, the upstream's base URL becomes the
recorder's in what the client receives, and the tape holds `{% raw %}{{baseUrl}}{% endraw %}`. Playback now fills
`{% raw %}{{baseUrl}}{% endraw %}`, `{% raw %}{{request.host}}{% endraw %}` and `{% raw %}{{request.scheme}}{% endraw %}` on the request side too,
so a tape whose requests carry the VCR's own URL replays on any port.
See [the VCR's own address]({{ site.baseurl }}/features/#vcr-address-placeholders).

## .NET Framework 4.8

The one NuGet package serves .NET 8 and .NET Framework 4.8 and puts the native engine in the output directory for
`packages.config` projects, for `PackageReference` without a `RuntimeIdentifier`, and for projects that reach the package through a
`ProjectReference`. AnyCPU gets both Windows engines; a 32-bit build fails with `SVCR001` rather than a
`DllNotFoundException` at run time, because there is no 32-bit engine. The package now carries API documentation and a README.

## The site

This release also brings the site up to date: a [page for each of the thirty languages]({{ site.baseurl }}/languages/) with exact install steps and
copyable snippets, a [feature reference]({{ site.baseurl }}/features/), a [prompt you can hand to a coding agent]({{ site.baseurl }}/agent-prompt/),
a note on [Servirtium and OpenAPI]({{ site.baseurl }}/openapi/), and an [llms.txt]({{ site.baseurl }}/llms.txt).
