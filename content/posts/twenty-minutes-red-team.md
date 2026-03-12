---
title: "20 Minutes With an AI Agent Changed How I Think About Recon"
date: 2026-03-11
draft: false
tags:
  - breach-catalog
  - red-team
  - reconnaissance
  - ai-as-attacker
  - agent-security
series: ["Breach Catalog"]
showToc: false
TocOpen: false
---

<div style="border-left: 3px solid #6b7280; background: rgba(255,255,255,0.04); padding: 12px 18px; margin: 1.5rem 0; border-radius: 0 4px 4px 0;">
<strong>TLDR:</strong> I spent 20 minutes doing AI-assisted red team simulation with Claude Cowork against a live production SaaS. No purpose-built tooling. No prior knowledge of the target. I walked away with confirmed PII exposure — real names, email addresses, account identifiers — on a live system. <strong>The same capabilities that make AI agents useful for legitimate work make them highly capable recon tools. The gap between "helpful assistant" and "passive attacker" is smaller than most people think.</strong>
</div>

I'm a QA analyst and secrets management practitioner — not a red teamer by primary specialization. I don't have a toolkit of custom scripts for offensive security work. What I do have is a browser, Claude Cowork, and enough security fundamentals to know what I'm looking at.

The target was a recently launched SaaS platform — small team, real users, actively onboarding. The kind of product built fast with modern tooling. I'm not naming it. The finding was responsibly disclosed. No data was retained beyond the moment of confirmation.

The methodology was simple: look at what the frontend exposes, use whatever credentials are visible to probe the backend API, see what responds. The agent read the production JavaScript bundle, located embedded API credentials — standard for the framework, by design — and used them to query the backend. The API returned a **complete schema for the entire application**: financial tables, PII fields, internal operations data, everything. No authentication required.

Then it found a callable endpoint that returned **live user records** — real names, email addresses, account identifiers — with no session, no elevated role, nothing beyond the public-facing credentials that were already in the bundle. Confirmed exposure. Stopped reading.

**Total time from start to confirmed PII: under 10 minutes.**

The agent didn't do anything a skilled analyst couldn't do manually. It fetched a URL, scanned text, made API calls, and interpreted responses. The difference is it did it **fast**, **automatically**, and with pattern recognition trained on more framework documentation than I've personally read. That's the asymmetry. The attacker's time-to-finding compresses dramatically. The defender's misconfiguration window doesn't shrink to match.

**The lesson:** If you're running a web application backed by a modern BaaS — Supabase, Firebase, Amplify — and you haven't explicitly audited what your public API surface returns to unauthenticated callers, you should assume an AI agent could enumerate that surface in minutes. The tolerance for "low-probability-of-discovery" misconfigs needs to shrink. **Discovery is getting automated.**
