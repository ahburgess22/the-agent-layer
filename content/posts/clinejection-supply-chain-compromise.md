---
title: "Clinejection: How a GitHub Issue Compromised Cline's Entire NPM Supply Chain"
date: 2026-03-09
draft: false
tags:
  - breach-catalog
  - prompt-injection
  - supply-chain
  - ci-cd
  - github-actions
  - agent-security
series: ["Breach Catalog"]
showToc: false
TocOpen: false
---

<div style="border-left: 3px solid #6b7280; background: rgba(255,255,255,0.04); padding: 12px 18px; margin: 1.5rem 0; border-radius: 0 4px 4px 0;">
<strong>TLDR:</strong> Cline's own AI support bot was weaponized against them — an attacker fed it instructions through a public GitHub issue, it complied, and the result was malicious software shipped to over a million weekly users. <strong>They didn't breach the system. They gave the AI instructions through a public channel, and it executed them.</strong> The bot didn't have direct access to publishing credentials. It corrupted the infrastructure that did.
</div>

Cline is one of the most popular AI coding tools in the world. Their GitHub repo runs an AI bot that automatically reads and responds to incoming issues — helpful for triage at scale. Someone figured out that if you craft the right issue title, the bot doesn't just read it. It follows it.

The attacker opened a fake issue with **embedded instructions**. The bot executed them — installed an attacker-controlled package, which ran malicious code before installation even completed. That code didn't steal anything directly. Instead, it **poisoned a shared build cache** that Cline's release pipeline trusted. When the nightly release job ran, it loaded the corrupted cache, harvested the NPM publishing credentials, and the attacker used them to ship a **malicious version of Cline to the public registry**.

The bot never had access to those secrets. It didn't need direct access. It just needed to corrupt something that a privileged workflow depended on.

**The lesson that matters:** When you give an AI agent broad tool permissions and point it at public input — GitHub issues, emails, support tickets — you're not just automating a task. You're creating **an execution environment that anyone on the internet can attempt to program**. The blast radius isn't limited to what the agent can directly access. It extends to everything that trusts the environment the agent operates in.

**Least privilege** isn't a nice-to-have on agentic systems. It's the primary control.
