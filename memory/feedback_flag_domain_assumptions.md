---
name: flag-domain-assumptions
description: "In pattern-drafter domain (sewing/CAD/textile), flag assumptions when proposing a model; don't present confident simplifications that need correction"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 8a95c1c5-178a-4f71-9e77-e15ea3c9cb13
---

When proposing a model or architecture in a domain Justin knows deeply and I don't (pattern drafting, sewing, garment construction, drafting systems, body measurement conventions), flag the assumptions I'm making and invite correction *before* committing to a recommendation. Don't present a confident simplified model that turns out to need correction round-trip-by-round-trip.

**Why:** Across the 2026-05-13 design-review conversation, Justin had to correct me three times where my initial framing was wrong in domain-load-bearing ways:

1. I said "pattern belongs to wearer" as one relationship. Real: patterns *belong to the designer*, and are *for the wearer's benefit* — two relationships with different implications for URL structure, deletion semantics, ownership.
2. I framed wearer-measurements as a drafting prerequisite (option C: "bounce them to Measurements with explicit handoff"). Real: measurements are an optional drafter convenience; the competitor is the slip of paper, not other apps.
3. I said "shared names stay as one entry" when applying multiple templates. Real: Aldrich-Bust and Armstrong-Bust are *the same name but not the same measurement*; merging them silently produces a wrong value. Provenance is per-entry.

Each correction cost a round-trip and required me to revise prior memories. A "here's the model I'd propose, flagging that I'm assuming X about Y — is that right?" framing would have surfaced the corrections one turn earlier.

**How to apply:**
- When proposing a domain model in pattern-drafting / sewing / measurement / drafting-systems: name the assumption explicitly ("I'm treating wearer as the pattern's parent — is that the right relationship?", "I'm assuming Bust means the same thing across drafting systems — does it?").
- Prefer questions over confident framings when the domain content is load-bearing.
- Less applicable when the domain is software/architecture/UI — there I have my own ground to stand on. Specifically applies when the load-bearing concept is *garment construction*, *body measurement*, *drafting method*, *textile workflow*, or *the difference between drafting systems*.
- See [[user-pattern-drafter-frame]] — Justin's framing is craft-first; corrections often carry textbook-level specificity. They're not pushback for its own sake.
