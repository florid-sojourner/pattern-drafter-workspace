---
name: Ask for artifacts (PDFs, images, JSON) before diagnosing
description: When investigating a visual or output-quality bug, ask Justin for the actual artifacts (printed PDF, canvas screenshot, document JSON) before proposing a root cause from code reading alone
type: feedback
originSessionId: 6246e3bc-9aa5-4af8-aa15-669267d16eaa
---
When Justin reports a defect in a rendered output (PDF, image, exported file), ask up front for the actual artifacts: the buggy output, the reference (e.g., canvas screenshot), and the source data (document JSON). Don't jump straight to code-reading and present a hypothesis as if it were diagnosed.

**Why:** Justin had the PDFs, the canvas image, and the JSON ready. Skipping them meant my "root cause" was an unverified inference — strong on mathematical plausibility, but not actually confirmed against the real defect. He called this out directly: "Did you find the problem without those?"

**How to apply:** At the start of any visual / output-quality investigation, before reading code, ask: "Do you have the buggy artifact, a reference (screenshot of what you expected), and the source document JSON? Drop them in and I'll diagnose against real evidence." Use the artifacts to confirm or rule out hypotheses, not as decoration after the fact. Code reading is for explaining *why* the artifact looks wrong once you've seen the wrong-ness.
