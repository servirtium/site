---
layout: post
title:  Kotlin Implmentation And Demo
date:   2020-02-01
keywords: "kotlin"
categories: [meta]
icon: fa-rss
---

David Denton made a Kotlin implementation as part of the larger `http4k` "Server as a Function"
technology that he created. As well as that, two ClimateAPI demos - one that uses Kotlin with the 
new library, and one that uses plain Java with the same Kotlin library.

David's intention is to use and support this library going forward.

Library as part of the http4k monorepo: https://github.com/http4k/http4k/tree/master/http4k-testing-servirtium
Climate API Demo: https://github.com/http4k/servirtium-demo-kotlin-climate-tck

Kotlin Examples (Climate API)

* Playback of a Servirtium recording: https://github.com/http4k/servirtium-demo-kotlin-climate-tck/blob/master/src/test/kotlin/servirtium/http4k/kotlin/DiskPlaybackClimateApiTests.kt
* Making a Servirtium recording: https://github.com/http4k/servirtium-demo-kotlin-climate-tck/blob/master/src/test/kotlin/servirtium/http4k/kotlin/DiskRecordingClimateApiTest.kt
* In the same packagage, see also a direct example for contrast, and one that can playback from recordings themselves on a website rather than from local disk.

Java Examples (Climate API):

* Playback of a Servirtium recording: https://github.com/http4k/servirtium-demo-kotlin-climate-tck/blob/master/src/test/kotlin/servirtium/http4k/java/DiskReplayClimateApiTests.java
* Making a Servirtium recording: https://github.com/http4k/servirtium-demo-kotlin-climate-tck/blob/master/src/test/kotlin/servirtium/http4k/java/DiskRecordingClimateApiTests.java
* In the same packagage, see also a direct example for contrast, and one that can playback from recordings themselves on a website rather than from local disk.
