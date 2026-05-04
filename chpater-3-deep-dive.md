# Chapter 3 Deep-Dive: Your Codebase Is the Interface

### Companion document to "Software Development in the Agentic Era"

**By Mike, in collaboration with Claude (Anthropic)**

---

The main guide argues that in the agentic era, your codebase has become the primary interface to the AI — its architecture, tests, and documentation determine whether agents help or create chaos. Easy to state; harder to operationalize. This chapter looks at what the claim means in practice, what the evidence actually supports, and where the received wisdom is already being challenged.

The thesis narrows to this: **AI agents perform dramatically differently on the same task depending on the engineering environment they inherit, and the artifacts teams are adding to adapt — context files, test harnesses, rule systems — are themselves consequential enough to be worth designing carefully rather than generating automatically.**

That second clause matters. A lot of 2026 advice is "add an AGENTS.md and your AI gets better." The evidence is more interesting than that.

It helps to think of the "interface" as three distinct properties the codebase needs to provide to an agent, each addressed by a different part of this chapter:

- **Legibility** — can the agent make sense of the code, conventions, and constraints? (Part 1.1 on architecture, Part 1.2 on context files.)
- **Feedback** — can the agent tell whether its changes worked? (Part 1.3 on test infrastructure.)
- **Trust** — what is the agent allowed to read, execute, and modify, and how do you keep that boundary from being subverted? (Part 2 on security.)

The three are related but not interchangeable. A codebase can be highly legible and still lack the feedback loop an agent needs to self-correct. It can have excellent tests and still expose a trust boundary an adversary can cross. The main guide's Chapter 3 treats these together; this deep-dive separates them because the evidence, the failure modes, and the mitigations are different for each.

---

## Part 1: What the Evidence Actually Shows About Codebase Structure

### 1.1 Architecture Conditions the Effect, Probably Substantially

The Jellyfish dataset covered in Chapter 2 — 20+ million PRs from 200,000 developers across roughly 1,000 companies — is the largest single window we have into how codebase structure interacts with AI performance. The headline finding, reported by Jellyfish's head of research Nicholas Arcolano: centralized codebases saw roughly 4x productivity gains from AI adoption, balanced architectures saw meaningful gains, highly distributed architectures (engineers regularly working across many repos) saw essentially none.

The proposed mechanism is context availability. In a centralized codebase, the agent can see the relevant code, conventions, and patterns in a single context window. In distributed architectures, critical integration knowledge lives in engineers' heads — how service A talks to service B, which repo owns the shared types, what the convention is for cross-service error handling.

The finding is suggestive and directionally consistent with the Chapter 1 deep-dive pattern — Reco's gnata worked partly because the JSONata problem was well-bounded; Carlini's compiler worked partly because compilation stages are natural modular boundaries. But it's one large dataset from one vendor, observational rather than experimental, and the specific 4x figure should be treated as a data point rather than a law. The direction of the effect is robust; the magnitude is not.

*Source: Jellyfish, "AI benchmarks: What Jellyfish learned from analyzing 20 million PRs," March 2026.*

### 1.2 Context Files Help — Conditionally

The most-cited artifact of the agentic era is the repo-root markdown file that the agent reads at session start: `CLAUDE.md` for Claude Code, `AGENTS.md` for Codex and (increasingly) other tools, `GEMINI.md` for Gemini CLI. Practitioner sources report AGENTS.md in tens of thousands of repositories — one commonly cited figure is 60,000+, though the precise count is hard to verify — and the format is emerging as a cross-tool convention.

The near-universal advice is "write one, and your agent gets better." Then a March 2026 ETH Zurich paper (the AGENTbench study) tested the claim.

The researchers built a novel benchmark of 138 real-world Python tasks sourced from niche repositories — deliberately avoiding the memorization bias of benchmarks like SWE-bench. They tested four agents (Claude 3.5 Sonnet, Codex GPT-5.2, GPT-5.1 mini, Qwen Code) across three conditions: no context file, LLM-generated context file, human-written context file.

The results complicate the naive story:

- **LLM-generated AGENTS.md files reduced task success rates by approximately 3% on average**, increased inference costs by over 20%, and required 2–4 additional reasoning steps.
- Human-written files performed better than LLM-generated ones, but the benefit over no file at all was modest and came with its own token-cost overhead.
- Architectural overviews and structural references were particularly counterproductive when stale, actively misleading the agent without improving task success.

The mechanism the authors propose: context files compete for the agent's attention budget. Content the agent could have inferred from the codebase itself — standard commands, common framework conventions — is net-negative if it costs tokens to include. Stale structural content is worse than absent structural content. Auto-generation tools produce exactly this kind of content. Matthew Groff's widely-read 2026 practitioner guide flags the same pattern from a different angle: "most repos either have nothing, or they have a bloated auto-generated file that the model quietly ignores."

There's also a ceiling on what any context file can fix. In a widely-circulated January 2026 post, Andrej Karpathy enumerated recurring failure modes in Claude Code — silent assumptions, overcomplication, changing unrelated code as a side effect — and observed that "all of this happens despite a few simple attempts to fix it via instructions in CLAUDE.md." A community project that operationalized his observations as a structured CLAUDE.md (the `andrej-karpathy-skills` repo, tens of thousands of stars within weeks) illustrates the kind of non-inferable, judgment-focused content the ETH Zurich study suggests is useful — whether it measurably improves outcomes remains unstudied.

What the evidence supports:

- A human-curated context file focused on non-inferable specifics (unusual tooling, repo-specific commands, hard constraints, rejected alternatives) probably helps, though with token-cost overhead.
- An auto-generated context file probably hurts.
- No context file fully compensates for judgment gaps in the model itself.
- The common advice "just add an AGENTS.md" is underspecified; the content and curation matter more than the existence.

*Sources: ETH Zurich AGENTbench study, March 2026 (reported via InfoQ, "New Research Reassesses the Value of AGENTS.md Files for AI Coding"); Augment Code, "How to Build Your AGENTS.md (2026)"; Matthew Groff, "Implementing CLAUDE.md and Agent Skills In Your Repository," February 2026; Andrej Karpathy, "notes from claude coding," X, January 26, 2026; Forrest Chang, `andrej-karpathy-skills` GitHub repository.*

### 1.3 Test Infrastructure as Agent Feedback Loop

The test suite's role shifts substantively in the agentic era. In traditional development, tests verify that humans didn't break things. With agents, tests are also the mechanism by which the agent knows whether its own changes work — the feedback loop the agent runs inside.

This changes what "good tests" means in ways that aren't fully captured by traditional testing guidance. Owain Lewis frames the shift: when humans write code, the feedback loop is invisible and internal — stack traces, red squiggles, failing tests, iteration. Agents need the same loop exposed explicitly. "Most engineers treat agents like they should do something we've never done: write working code on the first try without running it."

Concrete implications that experienced practitioners have converged on:

**Speed.** If your test suite takes ten minutes, the agent's iteration cycle is ten minutes. Eric Elliott's framing from a practitioner guide: test execution speed directly determines the feedback loop. Slow tests don't just slow humans — they degrade agent behavior, because the agent either waits (burning wall-clock and tokens) or skips tests (losing the feedback).

**Signal-to-noise ratio in test output.** Carlini's C-compiler project in Chapter 1 went to unusual lengths here: minimized console output, pre-computed summary statistics, infrequent progress updates. The reason was that agents burn context parsing output. A test runner dumping 500 lines of stack traces, deprecation warnings, and noise is worse than one producing clean red/green with clear failure messages — not just aesthetically, but functionally, because the agent has a finite attention budget and noise displaces signal.

**What TDD becomes with agents.** Martin Fowler's January 2026 fragment cites Unmesh Joshi's account of using TDD with Claude Code for substantial feature work: "When you're directing thousands of lines of code generation, you need a forcing function that makes you actually understand what's being built. Tests are that forcing function. You can't write a meaningful test for something you don't understand."

A more ambitious claim has emerged from practitioners: TDD isn't just compatible with agentic AI, it's what makes the feedback loop tight enough for agents to self-correct effectively. The "Coding Is Like Cooking" blog interviewed TDD practitioners using agentic tools and reported that most had shifted their workflow — tests and code are now often written together rather than strictly test-first — but the practice is still recognizably TDD: small steps, fast feedback, refactoring on green.

The strongest version of this claim is empirically unsettled. We have practitioner reports of good outcomes. We have a popular tool — Jesse Vincent's Superpowers plugin, 42,000+ GitHub stars, added to Anthropic's official marketplace in January 2026 — that operationalizes it. We have theoretical reasons it should work. We have endorsements from heavy users: Karpathy's January 2026 notes explicitly flag "get it to write tests first and then pass them" as a core pattern for getting leverage from agents, framing the broader point as "don't tell it what to do, give it success criteria and watch it go." What we don't have is an RCT. The softer version of the claim — tests as the agent's primary verification signal, with design consequences — is well-supported across multiple sources and consistent with the Chapter 1 success cases. Reco's gnata rewrite succeeded in part because 1,778 existing JSONata test cases defined "correct" before the AI was involved. Carlini's compiler relied on the GCC torture test suite as an external oracle.

What this supports as guidance rather than doctrine:

- Test speed and output quality are now agent-UX concerns, not just engineering-hygiene concerns.
- Investments in test infrastructure plausibly pay off more with agents than without them, though the claim is consistent with practitioner experience rather than demonstrated.
- TDD or TDD-adjacent workflows (tests-as-specification, small steps, fast green/red cycles) appear to work well with agents, though formal evidence lags practitioner experience.

*Sources: Owain Lewis, "The 10x Skill for AI Engineers in 2026: Agent Feedback Loops"; Eric Elliott, "Better AI Driven Development with Test Driven Development"; Martin Fowler, "Fragments" (January 8, 2026), citing Unmesh Joshi; "Coding Is Like Cooking" practitioner survey, March 2026; The Neuron, "Test-Driven Development for AI Coding," February 2026.*

### 1.4 An Aside on Vendor Evidence

Several sources in this section come from tool vendors or consultants with commercial interests in the conclusions — Jellyfish, Augment, tooling-specific blogs. Following the Chapter 2 convention: vendor telemetry is well-suited to identifying patterns at scale, because nobody else has the data, but less well-suited to validating net-impact claims. The ETH Zurich AGENTbench study is independent academic work and is the strongest single piece of evidence in this chapter; vendor claims are triangulated against it where possible.

---

## Part 2: The Codebase as Attack Surface

One development that has accelerated sharply since the main guide was written: the codebase is now a security boundary in ways it wasn't in 2025. Agents read files, rules, configs, and READMEs automatically, and they can execute code, install dependencies, and make network calls. The threat model splits naturally into two halves — what the agent reads (which can be poisoned) and what the agent is allowed to do (which can be weaponized) — and most of the documented attacks in 2025–2026 are chains that cross from one to the other.

### 2.1 Agent-Readable Inputs: The Poisoning Surface

The main guide referenced Pillar Security's March 2025 "Rules File Backdoor" disclosure: hidden malicious instructions in `.cursorrules` or similar config files, concealed via zero-width joiners and bidirectional Unicode markers, manipulating agents into inserting vulnerable or backdoored code. The technique is invisible to human review and propagates through repository forks.

Twelve months later, the category has expanded substantially. A partial catalog:

- **AIShellJack (September 2025):** An empirical study of prompt injection against GitHub Copilot in VSCode and Cursor, across Claude-4-Sonnet and Gemini-2.5-pro. Attack success rates up to 84% — and these are systems with terminal access, so successful injection means arbitrary command execution.
- **CVE-2025-54135 (CurXecute) and CVE-2025-54136 (MCPoison):** Two different Cursor vulnerabilities disclosed in August 2025. CurXecute: content summarized by Cursor's AI could rewrite MCP configuration files and execute commands, with no user interaction beyond requesting a summary. MCPoison: the trust model for MCP configs bound trust to the server name rather than the command, so an approved config could be silently modified afterward and re-execute on every project open.
- **NomShub (Straiker, January 2026):** A chain combining indirect prompt injection via a README, a sandbox bypass in Cursor's command parser, and weaponization of Cursor's own remote tunnel feature. Opening a malicious repository was enough to trigger persistent authenticated shell access on the victim's machine.

What these share is a common structure: agents read context — repos, READMEs, rules files, external data sources — automatically, and the boundary between "data the agent processes" and "instructions the agent follows" is fuzzy by design. Attackers exploit the fuzziness. The implication for codebase design is direct: configuration files, rules files, and MCP configs are now part of the threat model, reviewed in PRs with the same rigor as production code.

### 2.2 Agent-Executable Actions: The Weaponization Surface

Reading malicious content is only dangerous if the agent can act on it. The second half of the threat model is about what the agent is allowed to do once influenced, and this is where many of the distinctive agentic failure modes emerge — because agents can run commands and install dependencies that chat-based tools only suggest.

The clearest case is hallucinated dependencies. Models invent package names that don't exist; attackers register those names with malicious payloads. Seth Larson of the Python Software Foundation formalized the pattern as "slopsquatting."

A USENIX Security 2025 study tested 16 code-generation models across 576,000 code samples and found ~20% of samples recommended non-existent packages. The more important finding for attack economics: hallucinations were persistent rather than random. The same prompts reliably produced the same fake names on re-query, turning the problem from "rare noise" into "predictable target list." The attack has moved past theoretical. A malicious `huggingface-cli` package reportedly accumulated 30,000+ downloads in three months, and an April 2025 dark-web playbook documented tooling for generating plausible package-name variants at scale using LLMs.

This is a chat-era problem that becomes an agent-era problem at the point of execution. A chat-based tool suggests `pip install totally-fake-package` and a human has a chance to notice. An agent with shell access runs it autonomously, and the post-install script executes before anyone reviews anything. Endor Labs cites roughly 40% of AI-generated code introducing vulnerable dependencies — part hallucination, part the model preferring popular packages regardless of maintenance status or CVE history. The mitigation doesn't live in the AI tool: it's the boring dependency-policy layer (lockfiles, allowlists, fresh-package delays, SCA in CI, ideally pre-install hooks that block unrecognized packages) combined with scoping what the agent can execute without approval.

The broader pattern is the one the CVE list in 2.1 already hinted at: every attack there eventually cashes out through an action the agent was allowed to take without approval — writing to files outside its sandbox, editing MCP configs, opening a tunnel, running shell commands, installing a dependency. The injection gets the agent to do something; the permission model determines whether that "something" is contained or catastrophic. The Chapter 1 cases (Replit/SaaStr deleting the production database, Amazon/Kiro inheriting elevated permissions) are the same failure pattern without an external attacker — the agent's action capability outran the constraints around it. Back to this chapter's thesis: legibility and feedback determine whether an agent does useful work; trust — specifically, what the agent can execute without asking — determines whether the worst case is recoverable.

### 2.3 What This Implies for Codebase Design

Three concrete implications, scoped to what the evidence supports:

**Treat agent-readable config files as production code.** `CLAUDE.md`, `AGENTS.md`, `.cursorrules`, MCP configs — these are now attack surface. Review changes in PRs. Watch for Unicode anomalies. Don't auto-approve configs from external repositories. The specific mechanics vary by tool; the principle is that a file the agent reads with elevated trust needs to be reviewed with elevated scrutiny.

**Scope what agents can do, not just what they can see.** The Chapter 1 Replit/SaaStr case and the Amazon/Kiro cases both stemmed from agents inheriting human permissions. Environment separation, explicit permission boundaries, no-production-access by default, and approval gates for irreversible operations are not optional additions — they're the compensating controls that make autonomous execution survivable.

**Auto-run is one of the biggest risk multipliers.** Practitioner security guidance consistently emphasizes this: a large fraction of the documented attack scenarios require auto-run mode — or some equivalent permissive-execution setting — to complete. Disabling it, or requiring explicit per-command approval, breaks much of the attack chain even when the initial injection succeeds. The cost is slower agent workflows. The benefit is that the worst failure modes become containable.

*Sources: Pillar Security, "Rules File Backdoor"; AIShellJack paper (arXiv:2509.22040); Check Point Research, "MCPoison" (CVE-2025-54136); Tenable, CurXecute (CVE-2025-54135); Straiker, "NomShub" (January 2026); Endor Labs, "Cursor Security: How to Secure AI-Generated Code in 2026."*

---

## Part 3: What to Actually Do

The main guide's Chapter 3 prescriptions (separation of concerns, test design for agents, context files, ADRs) remain directionally correct. What this deep-dive adds is calibration on how much each one matters and where the traps are.

### 3.1 The Priority Order

Not everything is equal. Based on the evidence surveyed, and mapped to the triad that opens this chapter:

1. **Test infrastructure that's fast, deterministic, and produces clean output** *(feedback)*. The highest-leverage investment. It's the agent's feedback loop, and without it the agent cannot self-correct. Also the most durable — every future AI tool will need it.

2. **Module boundaries the agent can reason about independently** *(legibility)*. Small units with clear interfaces, so the agent's changes have a bounded blast radius. The old Separation-of-Concerns prescription, operationally upgraded: the AI's ability to reason about your system is bounded by your system's decomposability.

3. **Security boundaries on agent actions** *(trust)*. Environment separation, permission scoping, auto-run disabled where possible, approval gates for irreversible ops. The SaaStr/Amazon lesson from Chapter 1 translated into policy.

4. **A curated, minimal context file** *(legibility, with real tradeoffs)*. `AGENTS.md` / `CLAUDE.md` written by a human, focused on what the agent cannot infer. Based on the ETH Zurich evidence, this is lower-leverage than commonly claimed and actively harmful when auto-generated. Useful when done well; counterproductive when done lazily.

5. **ADRs and intent documentation** *(legibility, longer horizon)*. Pays off on architectural decisions and prevents the agent from confidently suggesting things the team already tried and rejected. High value, low urgency — but the value compounds over time.

### 3.2 Anti-Patterns the Evidence Supports

**Auto-generated AGENTS.md.** The ETH Zurich data is fairly direct on this. If the tool writing your AGENTS.md is the same kind of system that will read it, you're paying token cost for content the reader could have inferred.

**Verbose test output.** A test suite that "works" but dumps noise is worse for agents than one that's equally correct but produces clean signal. This is worth designing for explicitly.

**Trusting config files from external sources.** Repository-level rules files, MCP configs, and similar artifacts should never be auto-approved. The CVE list is long enough now that this qualifies as table-stakes hygiene, not defense-in-depth.

**Confusing "the agent has access" with "the agent should act."** Reading permissions and write permissions are separate; write permissions and execute permissions are separate; execute permissions and production-access permissions are separate. Each layer deserves its own decision.

### 3.3 Questions That Distinguish Useful from Theatrical

In the spirit of the Chapter 1 and 2 self-assessments:

- Does your test suite run fast enough that an agent can iterate on failures, and clean enough that an agent can diagnose them? If the failure output is a wall of noise, the agent is burning context parsing it instead of fixing the bug.
- If you have an AGENTS.md, was it written by a human focused on non-inferable specifics, or auto-generated from the codebase? If auto-generated, the ETH Zurich evidence suggests it's probably net-negative.
- What can your coding agent do without asking? Read files? Run tests? Modify code? Install dependencies? Execute arbitrary shell commands? Access production? Each "yes" is a policy decision that should have been made deliberately.
- If someone committed a malicious `.cursorrules` or MCP config to your repo, would anyone notice? If not, you're one indirect prompt injection away from the NomShub scenario.
- When was the last time you read your own context file end-to-end and asked whether each line earns its tokens?

---

## Conclusion

The main guide framed this chapter's territory as "your codebase is the interface." A year into the agentic era, the framing is holding up — but in practice "interface" decomposes into three distinct properties: legibility (the agent can make sense of the code and constraints), feedback (the agent can tell whether its changes worked), and trust (the agent's reach is bounded by something other than the agent itself). Different evidence, different failure modes, different investments. The specifics are moving faster than any single document can keep up with.

The parts that look durable: test infrastructure is the agent's feedback loop and deserves to be treated as such; module boundaries bound the blast radius of agent actions; documentation of intent matters more than documentation of mechanics because the agent can read what the code does but not why. These are the same fundamentals that made teams effective before AI, now upgraded with operational consequences.

The parts that are genuinely new and still settling: context files are more conditional than the adoption numbers suggest, auto-generation is probably a mistake, TDD with agents looks promising but lacks formal evidence, and the security attack surface is expanding faster than most teams' threat models. Each deserves a team-specific answer rather than a one-size prescription.

The risk in this area is the one the main guide flagged about AI generally: confident advice running ahead of evidence. "Write an AGENTS.md" is now the kind of received wisdom that a new research finding can partially invalidate. The teams that do well across multiple AI-tooling generations will probably be the ones that treat all of this — including the prescriptions in this chapter — as provisional, measure what they're doing, and update when the evidence changes.

One concrete thing to watch, regardless of which prescriptions turn out to be durable: what Karpathy has called *comprehension debt* — what accumulates when agents one-shot code that nobody on the team ever reads. It's adjacent to the skill-atrophy concern in the main guide's Chapter 5 and a natural companion to the quality-debt cycle in Chapter 2, distinct enough to deserve its own attention.

---

## Key References

| Source | Year | Relevance |
|---|---|---|
| Jellyfish, "AI benchmarks: 20M PRs" | 2026 | Architecture conditions AI benefit; ~4x for centralized, ~0 for distributed |
| ETH Zurich AGENTbench study (via InfoQ) | 2026 | LLM-generated AGENTS.md reduces success ~3%, +20% inference cost |
| Augment Code, "How to Build Your AGENTS.md (2026)" | 2026 | Practitioner synthesis; acknowledges ETH Zurich findings |
| Mohsenimofidi et al., "Context Engineering for AI Agents in OSS" | 2026 | 155 AGENTS.md files analyzed; conventions still in flux |
| Owain Lewis, "The 10x Skill: Agent Feedback Loops" | 2025 | Framing of agents needing exposed feedback loops |
| Martin Fowler Fragments, citing Unmesh Joshi | 2026 | TDD as forcing function for staying in the loop |
| "Coding Is Like Cooking," TDD with Agentic AI | 2026 | Practitioner survey; TDD workflow adaptations |
| Eric Elliott, TDD + AIDD guide | 2025 | Test execution speed as feedback-loop determinant |
| Jesse Vincent, Superpowers plugin | 2026 | 42K-star TDD-for-Claude-Code implementation; Anthropic marketplace |
| Pillar Security, "Rules File Backdoor" | 2025 | Original AI config file poisoning disclosure |
| AIShellJack (arXiv:2509.22040) | 2025 | Prompt injection attacks, up to 84% success rate on Copilot/Cursor |
| Check Point, "MCPoison" (CVE-2025-54136) | 2025 | MCP config trust model vulnerability |
| Straiker, "NomShub" | 2026 | Indirect injection + sandbox bypass + tunnel weaponization |
| Endor Labs, Cursor Security Guide 2026 | 2026 | Practitioner security guidance; auto-run risk analysis |
| Spracklen et al., "We Have a Package for You!" (USENIX Security 2025) | 2025 | 576K code samples, 16 models; ~20% recommend non-existent packages; persistence of hallucinations |
| Trend Micro, "Slopsquatting: When AI Agents Hallucinate Malicious Packages" | 2025 | Attack mechanics; agent-specific execution risk |
| The Register, "AI code suggestions sabotage software supply chain" | 2025 | "_Iain" dark-web playbook; documented weaponization at scale |
| Aikido Research, slopsquatting case studies | 2026 | `unused-imports`, `react-codeshift`; concrete spread patterns |
| Matthew Groff, CLAUDE.md implementation guide | 2026 | Practitioner view on context file anti-patterns |
| VS Code team, "How VS Code Builds with AI" | 2026 | First-party practitioner account of agent-assisted development |
| Andrej Karpathy, "notes from claude coding" (X post) | 2026 | First-person observations on ~80% agent coding; CLAUDE.md limits; tests-first pattern; "comprehension debt" |
| Forrest Chang, `andrej-karpathy-skills` GitHub repo | 2026 | Karpathy's observations operationalized as a CLAUDE.md file; ~48K stars |
