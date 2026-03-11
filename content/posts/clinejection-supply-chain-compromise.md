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
description: "A five-stage attack chain turned a GitHub issue title into a full supply chain compromise of the Cline NPM package. This is what agentic failure looks like in production."
showToc: true
TocOpen: false
---

*Breach Catalog — Entry #001. Source: Simon Willison's Blog via Adnan Khan. Incident date: March 2026.*

---

A developer opened a GitHub issue against Cline — a popular AI coding assistant — and by the time it was over, an attacker had published a malicious version of the package to NPM with over a million weekly downloads. The root cause wasn't a zero-day. It wasn't a credential leak. It was an AI agent reading a GitHub issue title and doing exactly what it was told.

This is the first entry in the Breach Catalog: a running record of real-world agentic breaches, cataloged for pattern recognition. This one earns the first slot because it's pedagogically perfect — a clean five-stage chain that demonstrates, in a single incident, nearly every foundational failure mode in agentic security.

---

## What the System Was

Cline ran an AI-powered issue triage workflow on their public GitHub repository. When any user opened an issue, the workflow automatically triggered `anthropics/claude-code-action@v1` — Claude Code running inside GitHub Actions.

The agent was configured with broad tool permissions: `Bash`, `Read`, and `Write`. It had the ability to execute arbitrary shell commands inside the CI/CD environment. This was considered acceptable because the agent was doing legitimate work — reading issues, triaging them, responding. The workflow was convenient. It worked well most of the time.

The problem is what "most of the time" obscures.

---

## The Attack Chain

### Stage 1: Prompt Injection via Issue Title

The attacker opened a GitHub issue with a crafted title. The workflow was configured to interpolate the issue title directly into the agent's prompt without any sanitization — meaning the agent received the attacker's text as part of its instructions rather than as data it was processing.

This is **indirect prompt injection**: instead of attacking the model directly, the attacker poisoned the external content the agent was told to process. The issue title was a surface the agent trusted. It shouldn't have.

The injected instructions directed the agent to take specific actions. Because the agent had `Bash` execution permissions, it had the means to comply.

### Stage 2: Malicious Package Installation

The injected instructions directed the agent to run `npm install` on an attacker-controlled package hosted on GitHub. The package — named `cacheract` — contained a `preinstall` script in its `package.json`: code that executes automatically before installation completes.

This gave the attacker arbitrary code execution inside the workflow environment without needing any additional exploitation step. The agent installed the package; the package ran the attacker's code. From the attacker's perspective, this was the moment they owned the environment.

### Stage 3: Cache Eviction via Storage Overflow

Here's where the attack becomes genuinely sophisticated.

The attacker's code stuffed existing GitHub Actions cache paths with approximately 11GB of junk data. GitHub automatically evicts caches that exceed 10GB. This wasn't a bug — it's documented cache management behavior. The attacker weaponized it.

The eviction cleared the legitimate cached `node_modules` folder that workflows in the repository depended on. This created a vacancy — an empty cache slot waiting to be filled.

### Stage 4: Cache Poisoning

After eviction, the attacker's code created new cache files under the same cache key that had just been cleared:

```
${{ runner.os }}-npm-${{ hashFiles('package-lock.json') }}
```

This cache key was shared between the issue triage workflow and the nightly release workflow. The poisoned cache now contained a secret-stealing mechanism, sitting under a key that the release workflow would load as trusted infrastructure.

The critical point: the issue triage workflow never had access to NPM publishing secrets. It didn't need to. The attacker didn't try to steal them directly. They compromised the environment that a privileged workflow depended on instead.

### Stage 5: Secret Exfiltration via the Release Workflow

When the nightly release workflow ran, it loaded what it believed was its legitimate cached `node_modules`. It was actually loading the attacker's poisoned cache, which harvested the NPM publishing secrets present in the release workflow's environment.

Those secrets were then exfiltrated, and the attacker used them to publish `cline@2.3.0` to NPM — a malicious version of the package, now available to every developer who ran `npm install cline`.

The attacker showed restraint in this case, only adding an OpenClaw installation hook. The capability for far worse — credential harvesting, backdoor injection, downstream compromise of every developer using the package — was fully established.

---

## What Security Assumptions Failed

Each stage of this chain represents a distinct assumption failure. They're worth naming individually because each one recurs across other incidents.

**"Users opening issues are not adversarial actors."** Against a public GitHub repository, every issue comes from an anonymous internet user. The entire internet is a potential adversary. Interpolating untrusted external input directly into an agent's prompt is not an edge case risk — it's the baseline threat model for any public-facing agentic workflow.

**"The triage workflow doesn't touch important secrets, so the blast radius is limited."** An agent doesn't need direct access to secrets if it can manipulate the environment a privileged workflow depends on. This is lateral movement — a classical offensive technique — applied to shared CI/CD infrastructure. The isolation between workflows existed at the secret level but not at the infrastructure level. That gap was the attack surface.

**"Workflow caches are isolated between workflows."** They aren't, by default. GitHub Actions caches are shared by cache key across the repository. Any workflow can write to and read from a cache key, regardless of which workflow created it. When a lower-privilege workflow shares a cache key with a higher-privilege workflow, the lower-privilege workflow can poison what the higher-privilege workflow loads.

**"Broad tool permissions are fine because the agent is doing legitimate work."** `--allowedTools "Bash,Read,Write"` given to an agent processing untrusted external input is an enormous attack surface. An issue triaging agent has no legitimate reason to execute arbitrary shell commands. The tool permissions existed because they were convenient, not because they were necessary.

---

## What Would Have Prevented It

**Input sanitization at the prompt boundary.** Issue titles — and any other content from external untrusted surfaces — should never be interpolated raw into an agent's prompt. Strip anything that looks like an instruction. Better yet, pass external content through a sanitization layer that summarizes or reformats it before it reaches the agent's context. The agent should receive a processed representation, not raw user input.

**Principle of least privilege on tool access.** An issue triage agent needs to read the issue and possibly respond to it. It does not need Bash execution, file write access, or the ability to install packages. Tool permissions should be scoped to the minimum required for the stated task. When you grant `Bash` to an agent processing public input, you've given the internet a shell.

**Workflow secret isolation.** Secrets used by privileged workflows — particularly publishing and deployment credentials — should never be accessible to or inferable by lower-trust workflows. These belong in separate environments with strict boundaries. The fact that the issue triage workflow could affect what the release workflow loaded was a design failure.

**Cache namespace isolation between workflows.** Cache keys should be namespaced per workflow, not shared across workflows with different privilege levels. A release workflow that loads cache produced by a public-facing triage workflow has implicitly trusted unverified content. That trust relationship should never exist by default.

**Human approval gates on privileged actions.** Any agent action that touches publishing, deployment, or external package installation should require explicit human approval before execution. This is not a performance tradeoff — it's a basic security control. Autonomous execution of privileged actions in a public-facing pipeline is the root condition that made everything else in this chain possible.

---

## Technique Reference

This incident demonstrates four distinct techniques that will recur across the Breach Catalog:

**Indirect Prompt Injection** — The attacker's instructions weren't sent directly to the model. They were embedded in external content (the issue title) that the agent was directed to process. Any agent that ingests content from untrusted external sources is potentially vulnerable to this technique.

**Supply Chain Attack via Agent** — The agent was used as the delivery mechanism for installing a malicious package. This is a new variant of a classical attack: instead of compromising a developer's machine or a build system directly, the attacker used an AI agent's autonomous execution capabilities to achieve the same outcome.

**Cache Poisoning** — Corrupting shared build infrastructure to laterally move to a higher-privilege workflow. Not novel in CI/CD contexts, but the agent was the mechanism that enabled it here.

**Privilege Escalation via Shared Infrastructure** — Bypassing secret isolation through environmental manipulation rather than direct access. The attacker never needed access to the release workflow's secrets directly. They only needed to compromise something the release workflow trusted.

---

## Open Questions

A few things this incident surfaces that don't have clean answers yet:

How should agent frameworks differentiate between content from trusted internal sources and untrusted external surfaces? The current model — pass everything to context, let the model sort it out — is clearly insufficient. What does a principled trust boundary look like in practice?

Where is the right line between useful automation and dangerous autonomy in CI/CD workflows? Requiring human approval for every agent action defeats the purpose of automation. But fully autonomous execution in public-facing pipelines is clearly wrong. The design space between those poles isn't well mapped.

How do you architect cache infrastructure when multiple workflows with different privilege levels need to interact with the same repository? Shared cache keys are a footgun. What does safe-by-default look like here?

How many other AI-powered GitHub Actions in the wild right now have the same basic vulnerability profile as this workflow? The answer is probably: a lot. The issue triage pattern is common. The prompt injection surface is the same everywhere.

---

*This is Breach Catalog Entry #001. Each technique identified here — indirect prompt injection, supply chain attack via agent, cache poisoning, privilege escalation via shared infrastructure — will get its own entry in the Technique Library as this catalog grows.*
