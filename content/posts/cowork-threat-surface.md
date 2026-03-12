---
title: "Cowork Threat Surface: When the Agent Is the Attack Vector"
date: 2026-03-10
draft: false
tags:
  - breach-catalog
  - prompt-injection
  - agent-security
  - cowork
  - threat-surface
  - consumer-ai
series: ["Breach Catalog"]
showToc: false
TocOpen: false
---

<div style="border-left: 3px solid #6b7280; background: rgba(255,255,255,0.04); padding: 12px 18px; margin: 1.5rem 0; border-radius: 0 4px 4px 0;">
<strong>TLDR:</strong> Anthropic's new consumer AI agent can read your files, browse the web, and run commands on your machine — all on your behalf, all autonomously. <strong>When an AI acts on your behalf, anything it reads can act on your behalf too.</strong> No confirmed breach yet, but the same attack class already produced a supply chain compromise at Cline. The difference is this product is marketed to everyone at $20 a month.
</div>

Claude Cowork is Anthropic's general-purpose desktop agent — released to all Pro subscribers in early 2026. It reads your files, fetches web pages, runs terminal commands, and executes multi-step tasks on your behalf. **That's the product. That's also the attack surface.**

Any content the agent ingests during a task is a potential instruction source. A document in the folder you gave it access to. A webpage it fetches while doing research. A search result. If any of that content contains instructions, there's a non-trivial chance the agent reads them as legitimate directives and acts on them — alongside, or instead of, your actual task.

This isn't theoretical. It's a known class of attack called **indirect prompt injection**, and it works exactly because the agent has no reliable way to distinguish "instructions from the user" from "instructions embedded in content the user asked me to process." The attack is especially potent here because the target demographic — the whole point of Cowork — is **non-technical users**. People who don't know what prompt injection is, can't recognize anomalous agent behavior, and were told to "watch for suspicious actions," which is not actionable guidance.

The conditions that produced the Cline supply chain compromise are present here **at consumer scale**. The difference is surface area: Cline was one repo. Cowork is anyone with a $20/month subscription and a folder of work files.

**The lesson:** General-purpose agents processing untrusted external content — files, web pages, search results — require explicit trust boundaries between what the user instructed and what the agent encountered in the wild. Without that boundary, **capability and exploitability are the same thing**.
