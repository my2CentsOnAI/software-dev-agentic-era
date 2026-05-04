# Chapter 2 Deep-Dive: The Measurement Problem

### Companion document to "Software Development in the Agentic Era"

**By Mike, in collaboration with Claude (Anthropic)**

---

The main guide says subjective productivity reports are unreliable. This chapter narrows that claim to a more specific and more useful one:

**AI frequently improves coding-stage activity. Teams often mis-measure whether those gains survive contact with the full delivery system.**

That is the thesis. What follows is the evidence for it, the structural reasons it's true, and what better measurement looks like.

---

## Part 1: Three Levels of Evidence

The mismatch between what AI feels like it's doing and what it's measurably doing shows up at the individual, team, and executive levels. The findings aren't identical across levels — the mechanism differs — but they point in the same direction.

### 1.1 Individual Level: METR (2025 + 2026 Follow-Up)

METR's 2025 randomized controlled trial is the most-cited study in this space. It's also the most misread, in both directions. Here's the arc.

Sixteen experienced open-source developers, 246 real tasks on codebases they'd maintained for years, random assignment of AI-allowed vs. AI-disallowed. Developers primarily used Cursor Pro with Claude 3.5/3.7 Sonnet. Before starting, they estimated AI would save them 24% of task time. Afterward, they estimated it had saved 20%. Measured result: AI use increased task completion time by 19%.

Critics pointed out the participants had minimal Cursor experience. Fair. METR's August 2025 follow-up — 57 developers, 800+ tasks, more diverse projects — produced a smaller estimated slowdown of -4% with a wide confidence interval (-15% to +9%). More importantly, METR discovered that 30–50% of invited developers refused to participate if they couldn't use AI, which biased the original sample toward developers willing to work without it. By February 2026, METR revised their position: "AI likely provides productivity benefits in early 2026."

What remains robust across the iterations is not the slowdown number but the gap between self-report and measured outcome. Subjective impressions were a poor guide to measured impact even as the effect size itself moved. That is a narrower claim than "developers can't tell if AI is helping" but it's the claim the data actually supports.

One new measurement problem surfaced with agentic tools. Several developers reported they couldn't accurately track time-spent because they worked on other things while agents ran. Time-based measurement becomes harder to interpret as a proxy for effort once the work parallelizes. METR's own transcript analysis of internal staff using Claude Code estimated 1.5x to 13x time savings — but flagged that much of this came from concurrency (kicking off agents and doing other work) and from task substitution (doing things with AI that wouldn't have been done otherwise). Neither translates directly into business productivity.

*Sources: METR, "Measuring the Impact of Early-2025 AI on Experienced Open-Source Developer Productivity," arXiv:2507.09089, July 2025; METR follow-up, February 2026; METR transcript analysis, February 2026; Domenic Denicola, "My Participation in the METR AI Productivity Study," July 2025.*


### 1.2 Team Level: Uplevel (~800 Developers, 2024)

Uplevel analyzed engineering telemetry — not self-reports — from nearly 800 developers across their customer base. Half had Copilot access; half didn't. The study has methodological limits (3-month baseline, 351 developers in treatment, single tool), but the finding relevant here is narrow: telemetry and self-report diverged sharply.

Measured: no improvement in PR cycle time, no improvement in throughput, a 41% increase in bugs for Copilot users. Concurrent industry surveys: developers overwhelmingly reporting productivity gains from AI tools. The two findings aren't necessarily contradictory — they measure different constructs, and developers can find AI useful at the task level while telemetry shows no delivery-level improvement. But they cannot be treated as equivalent evidence about AI's impact on software delivery.

The useful takeaway isn't "Copilot causes bugs." It's that when telemetry and sentiment disagree this strongly, sentiment is not a reliable proxy for what's happening in the delivery pipeline.

*Source: Uplevel Data Labs, "Can Generative AI Improve Developer Productivity?" 2024.*


### 1.3 Executive Level: NBER (February–March 2026)

Two National Bureau of Economic Research papers brought the question to the macroeconomic level. The first surveyed nearly 6,000 executives across the US, UK, Germany, and Australia. The second studied ~750 CFOs.

Findings: 69% of firms actively use AI. 89% report no productivity impact over the past three years. Yet the same executives forecast 1.4% productivity gains over the next three.

The CFO study documented what the researchers call a "productivity paradox" — perceived gains consistently exceeded measured gains, likely reflecting delayed revenue realization. Apollo chief economist Torsten Slok summarized the situation: "AI is everywhere except in the incoming macroeconomic data."

The Solow Paradox parallel is obvious and the NBER authors draw it directly: IT investments were widely deployed for a decade before productivity gains appeared in the statistics. The AI version might resolve the same way. It might not. The data doesn't yet distinguish between these possibilities.

The gains that *are* visible in the data cluster in predictable places: high-skill services and finance benefit more than manufacturing; larger and already-productive firms benefit more than smaller ones. This matches the DORA finding from the main guide — already-high-performing teams get the boost.

*Sources: Yotzov, Barrero, Bloom et al., "Firm Data on AI," NBER Working Paper 34836, February 2026; Baslandze et al., "Artificial Intelligence, Productivity, and the Workforce," NBER Working Paper 34984, March 2026.*


### 1.4 A Worker-Experience Complement

One more study, offered as complement rather than evidence for delivery impact. HBR's February 2026 research found that workers given AI tools voluntarily expanded their workloads — working faster, taking on broader scope, extending into more hours — describing "a sense of always juggling, even as the work felt productive."

This doesn't prove anything about AI's effect on software delivery. But it offers one plausible mechanism for why the individual-level perception gap exists: doing more generates the feeling of productivity, so throughput and felt productivity can both rise without delivery outcomes moving.

*Source: Harvard Business Review, "AI Doesn't Reduce Work — It Intensifies It," February 2026.*


---

## Part 2: Why the Gap Exists

The three levels of evidence don't show the same result — they show related ones. Self-reports often exceed hard outcome gains. Coding-stage gains often don't translate cleanly to system-level gains. Organizational productivity effects remain uneven and often undetected. What they share is a measurement implication: what you measure and how you measure it will determine what story the data tells.

Three structural factors explain most of the pattern. Each points to a specific measurement failure.

A note on sources before going further. Several of the largest datasets in this section come from vendors (Jellyfish, Atlassian's engineering blog, GitHub's own studies). Vendor telemetry is well-suited to identifying patterns at scale — they have access to data nobody else does — but less well-suited to validating claims about AI's net impact, because the vendor has commercial interests in the conclusions. Independent academic work (METR, He et al., Liu et al., NBER) is cited throughout as a counterweight. Where the chapter relies on vendor data, the claim is scoped to what the data can support.

### 2.1 Amdahl's Law Applied to Software Development

Gene Amdahl's 1967 insight: the maximum speedup from improving one part of a system is bounded by the fraction of time that part accounts for. Speed up 30% of the work by infinity and the system improves by at most 1.43x. The other 70% hasn't changed.

Estimates of how much of the software development lifecycle is "writing code" vary considerably by team, methodology, and definition — 20–35% is a commonly cited range, though the true figure depends heavily on what you count (does code review count as coding? debugging?). The specific number matters less than the direction: coding is one stage among many, and requirements, design, review, testing, deployment, and coordination consume meaningful fractions of the rest. Even a dramatic speedup of the coding stage leaves most of the pipeline untouched.

Philipp Dubach's March 2026 synthesis notes that at 92.6% monthly AI adoption, multiple independent research efforts cluster around roughly 10% organizational productivity gains. That figure is consistent with what Amdahl's Law predicts for a partial speedup of a minority stage. It doesn't prove the law is the mechanism — organizational productivity has many inputs — but it's the expected order of magnitude.

Atlassian's engineering blog illustrates the downstream effect with a scenario: a developer adopts AI tools and completes code for three features in a day. The product manager has a full review queue from stakeholder meetings; the senior engineer responsible for approvals is overwhelmed. Nothing ships. Individual stats look great. System stats are flat. By end of week, the developer has several open PRs, reviewers start to skim, and quality erodes. The individual got faster. The system didn't.

*Sources: Philipp D. Dubach, "AI Coding Productivity Paradox: 93% Adoption, 10% Gains," March 2026; Atlassian, "How Amdahl's Law still applies to modern-day AI inefficiencies," April 2026.*


### 2.2 The Bottleneck Moves

The Jellyfish dataset is the largest current window into what happens at team scale: 20+ million pull requests from 200,000 developers across roughly 1,000 companies, June 2024 through early 2026. Full AI adoption correlated with approximately 2x PR throughput and 24% faster cycle times. Clear gains at the activity level.

Faros AI's telemetry (10,000+ developers, 1,255 teams) measured what happened downstream: PR review times increased 91%, PR sizes inflated 154%, and at the company level the correlation between AI adoption and DORA delivery performance metrics disappeared.

The interpretation that fits both datasets: more code arriving at a review pipeline whose effective capacity didn't expand proportionally. This is consistent with the Amdahl bottleneck-shift prediction, though the data is observational — neither study directly measured review capacity, and the causal chain (AI generates more code → reviewers can't keep up → review time increases → delivery stays flat) is inferred rather than observed.

Jellyfish also surfaced a finding that connects this chapter to the main guide's Chapter 3 (codebase as interface). Across their dataset, codebase architecture was strongly associated with the magnitude of AI's benefit: centralized codebases saw roughly 4x productivity gains, balanced architectures saw meaningful gains, and highly distributed architectures (engineers regularly working across many repos) saw essentially no gain. Nicholas Arcolano, Jellyfish's head of research, frames this as a context problem — AI can't access the tribal knowledge that lives in engineers' heads about how services interact across repositories.

The Harvard/Jellyfish collaboration (January 2026) studied 100,000 engineers across 500 companies and confirmed the larger pattern: AI is making coding measurably faster, code quality isn't visibly suffering at the PR level, and the gains aren't translating into business outcomes. Their conclusion: "Coding isn't the bottleneck. For many teams, the limiting factor is everything around the code."

If your measurement tracks the part that sped up, AI looks transformative. If your measurement tracks the whole system, the gains are modest or absent. Both measurements are correct. They're measuring different things.

*Sources: Jellyfish, "AI benchmarks: What Jellyfish learned from analyzing 20 million PRs," March 2026; Jellyfish/Harvard collaboration, January 2026; Faros AI telemetry, 2025.*


### 2.3 Activity Metrics Inflate; Outcome Metrics Don't

Activity metrics — lines of code, commits, PRs opened, PRs merged — rise when the coding stage speeds up. Whether the output holds up is a separate question, answered by quality metrics measured on the same code.

Four large-scale studies span the picture.

**GitClear (2025, 211M lines of code):** Code churn — code reverted or significantly updated within two weeks of being written — rose from 3.1% in 2021 to 5.7% in 2024. Copy/paste code surpassed refactoring for the first time in the dataset's history. Duplicated code blocks increased roughly 8-fold. The trends correlate with AI adoption but causation is observational. GitClear's 2026 follow-up, using direct API integration with Cursor, Copilot, and Claude Code, identified "Power Users" authoring 4–10x more code than non-users, with persistent side effects in churn and duplication.

**He et al. (MSR '26, peer-reviewed):** A difference-in-differences study of 807 Cursor-adopting GitHub repositories against matched controls. Cursor adoption produced 3–5x increases in lines added in the first month post-adoption; this velocity boost dissipated within two months. Static analysis warnings increased 30% and code complexity increased 41%, and these changes persisted across the study window. Panel GMM models found that accumulated technical debt was associated with reduced subsequent velocity. The authors frame this as a self-reinforcing cycle, though the observation window (roughly two years) means "persistent" is a stronger claim than "permanent."

**Liu et al. (arXiv, March 2026):** Static analysis of 304,362 verified AI-authored commits from 6,275 GitHub repositories across five AI assistants (Copilot, Claude, Cursor, Gemini, Devin). More than 15% of commits from every AI assistant introduced at least one quality issue. Of 484,606 distinct AI-introduced issues tracked, 24.2% still survived at the latest repository revision — not fixed, not removed, accumulating.

**Rahman & Shihab (arXiv, January 2026):** A complication for the "AI code is disposable" narrative. Tracking 200,000+ code units across 201 open-source projects, they found AI-authored code actually survives *longer* than human-written code (15.8% lower modification hazard at line level). Combined with the Liu et al. finding, the implication is that AI code tends to persist, and when it contains quality issues, those issues persist with it.

What the activity metrics showed: more code, more PRs, more commits. All true. What the quality metrics showed: more churn, more duplication, less refactoring, more warnings, more complexity, more unfixed issues. Also true. These aren't contradictory; they're measuring different things. The perception gap lives in the space between them — PRs merge, throughput rises, and maintenance costs accumulate outside the metrics teams are watching.

One artifact worth flagging. GitHub's 2022 "55% faster" Copilot study still appears in enterprise sales decks in 2026. The study: one JavaScript task, 35 completers, no quality assessment of the output, confidence interval of 21% to 89%. It measures speed on an isolated, AI-friendly task without checking correctness. That is a real finding about a specific scenario. Used as evidence for site-wide productivity claims, it functions more as marketing support than as demonstration of broad productivity impact.

*Sources: GitClear, "AI Copilot Code Quality: 2025 Data"; GitClear 2026 cohort follow-up; He et al., "Speed at the Cost of Quality," arXiv:2511.04427; Liu et al., "Debt Behind the AI Boom," arXiv:2603.28592; Rahman & Shihab, "Will It Survive?" arXiv:2601.16809; GitHub, "Research: Quantifying GitHub Copilot's impact on developer productivity and happiness," 2022.*

---

## Part 3: Where AI Actually Helps

The evidence isn't all negative. Gains show up in specific contexts, and understanding which contexts matters for measurement — because if you apply AI where it doesn't help and measure it where it does, you'll draw the wrong conclusions in both directions.

### 3.1 Context, Not Seniority, Is the Variable

The pattern the evidence supports: AI helps most when implementation mechanics dominate the work. It helps less when judgment, local context, or architectural tradeoffs dominate.

That frame explains findings that otherwise look contradictory. GitHub's studies show juniors benefiting most from Copilot — true, because juniors face more syntax and API discovery work, which is exactly the "implementation mechanics" case. ANZ Bank's internal trial found Copilot most beneficial for *expert* Python developers — also true, because ANZ's experts already knew what to build and used Copilot to write it faster, while juniors at ANZ lacked the domain context to evaluate what Copilot produced. Different contexts, same underlying variable.

Jellyfish's Q2 2025 data adds supporting evidence: juniors caught up to seniors on AI-assisted PR speed (both around 1.2x faster), consistent with AI compressing the gap where implementation dominates.

The senior-developer story adds a late-stage wrinkle. Fastly's 2025 survey found seniors shipping 2.5x more AI-generated code than juniors because they catch mistakes. But nearly 30% reported that fixing AI output consumed most of the time they'd saved. When the bottleneck is judgment, speed gains on implementation don't compound.

None of this means seniority is irrelevant. It means seniority is a proxy for which variable actually matters: whether the work you're doing is bottlenecked on implementation mechanics or on judgment and local context. Measure that, not the developer's title.

*Sources: Jellyfish, "2025 AI Metrics in Review"; ANZ Bank internal study; Fastly 2025 Developer Survey; GitHub 2022 Copilot productivity research.*

One related pattern worth noting here rather than in its own section: much of the apparent speedup from AI comes from parallelization rather than per-task acceleration. Faros found 47% more PRs handled with no change in individual task cycle time. METR's transcript analysis attributed much of the apparent time savings to concurrency — kicking off agents and working on other things while waiting. Measured as "tasks completed per week," this looks like a win. Measured as "delivered value per week," it depends entirely on whether the additional tasks were worth doing. Current measurement rarely distinguishes between the two.


### 3.2 Architecture as a Major Conditioning Variable

The Jellyfish architecture finding — 4x gains for centralized codebases, minimal gains for highly distributed ones — is the strongest single evidence in the dataset that context determines outcome. Same tools, same models, different results.

The proposed mechanism, as Arcolano frames it, is context availability. Centralized codebases let the AI see the relevant code, conventions, and patterns. Distributed architectures force the AI to operate on partial information while critical integration knowledge lives in engineers' heads.

This is one large dataset from one vendor. That's informative but not the kind of replicated cross-study result one would want to turn into doctrine. The finding is suggestive and directionally consistent with the Chapter 1 deep-dive pattern — Reco's gnata succeeded partly because the problem was well-bounded; Carlini's compiler worked partly because each agent could reason about self-contained compilation stages. The underlying mechanism, that AI delivers where the engineering context is coherent and struggles where it isn't, is also what the main guide's Chapter 3 argues from different evidence. But the specific 4x figure should be treated as one data point, not a law.

*Source: Jellyfish, "AI benchmarks: What Jellyfish learned from analyzing 20 million PRs," March 2026.*

---

## Part 4: What to Measure and How

If the failure mode is measuring the stage AI optimizes instead of the system it feeds into, the measurement fix follows from it. The goal isn't perfect measurement — nothing achieves that — but measurement honest enough to tell you whether AI is actually helping the system, not just the stage.

### The Metrics Hierarchy

**Don't use as evidence of impact:** Lines of code, commits, PR count, AI suggestion acceptance rate. These rise with AI adoption whether or not AI is helping. They measure output volume, not delivery outcomes.

**Use with caution:** PR cycle time and throughput. Useful but gameable, and they can mask bottleneck shifts. A spike in throughput with flat cycle time often means AI is being applied to simple tasks that were already fast.

**Use as primary evidence (DORA metrics):** Deployment frequency, lead time for changes, change failure rate, time to restore service. These capture the full delivery cycle. They aren't ground truth — no metric is — and they can be manipulated by gaming the deployment unit or underreporting failures. But they resist the inflation pattern that dominates activity metrics, and they've been validated across thousands of organizations.

**Also track:** Where work actually queues. If coding isn't the bottleneck — and for most teams it isn't — speeding coding won't move delivery metrics. Map the pipeline. Find the wait states. The Atlassian "Maya" scenario happens every day in teams that optimized the wrong stage.


### Use a Framework That Resists Single-Number Collapse

Productivity is multi-dimensional. Collapsing it into throughput or velocity produces an answer that's easy to report and frequently misleading.

The SPACE framework (Satisfaction, Performance, Activity, Communication, Efficiency) from Microsoft Research is one option. Its value here is specifically diagnostic: if activity is up but satisfaction is down, the team is on an unsustainable trajectory — which is what the HBR intensification research would predict. If efficiency rises while communication declines, individual speed is being bought at the cost of coordination. The framework forces you to look at the whole, not just the number that happens to be improving.

Which framework you pick matters less than not picking only one metric. A dashboard showing AI lifted PR throughput while leaving DORA metrics flat is telling you something real; a dashboard showing only the PR throughput is telling you something misleading.


### Track Code Quality Over Time

The He et al. self-reinforcing cycle — velocity gains dissipate, complexity increases persist — is a longitudinal finding. A single-point-in-time quality measurement won't catch it.

CodeScene's CodeHealth metric, validated against expert assessments, is one tool for this. The specific tool matters less than the practice: periodically measure maintainability, complexity, and duplication alongside velocity. If quality metrics drift while velocity rises, the He et al. pattern may be in motion, and today's speed is borrowing against tomorrow's maintainability.

The "Echoes of AI" study found no maintainability degradation at the file level — individual files AI produces are fine. The debt shows up in aggregate, in volume, and in what one March 2026 paper terms "cognitive debt" (team-level erosion of shared understanding) and "intent debt" (lost rationale for why decisions were made). File-level metrics catch one kind. ADRs and documentation practices — covered in Chapter 3 of the main guide — address the other.


### Measurement Anti-Patterns to Avoid

**"Developers say they feel faster" as evidence of ROI.** The individual-level perception gap is well-documented enough that self-report without corroborating telemetry is a starting point, not a conclusion.

**Tracking adoption percentage as success.** Amazon's 80% mandate (Chapter 1 deep-dive) demonstrated that adoption pressure can outpace review capacity. Adoption isn't impact.

**Comparing pre/post AI without controlling for confounds.** The METR selection bias discovery — 30–50% of developers wouldn't participate without AI — shows how contaminated these comparisons get. Task type shifts, team composition changes, and survivorship bias (frustrated users drop out, making the remaining sample look better) all corrupt naive comparisons.

**Evaluating a vendor's tool with the vendor's metrics.** GitHub's "55% faster," Jellyfish's cycle time improvements, Copilot's acceptance-rate dashboards — these come from organizations with direct financial interest in the conclusions. The findings aren't necessarily wrong, but weighing them alongside independent research (METR, Uplevel, NBER, academic work) is how you avoid circular evidence.


### Questions to Pressure-Test Your Own Measurement

These aren't gates to pass before using AI. They're the difference between knowing whether AI is helping and assuming it is.

- What metrics are you using to evaluate AI's impact? If they're all activity-based, you're tracking output volume, not delivery impact.
- Can you identify your actual delivery bottleneck? If it's not coding, faster coding won't move delivery metrics.
- Do your metrics capture rework and quality, or only velocity?
- Have you baselined DORA metrics pre-adoption, so you have a before to compare the after against?
- Is AI adoption tracked as a KPI in ways that would pressure reports of its value?
- Are the people measuring AI's impact the same people who advocated for its adoption?
- Does your organization have a way to say "AI didn't help here" without it being career-limiting?

The summary question: if your AI vendor's dashboard shows a 30% productivity increase and your DORA metrics are flat, which do you believe? The dashboard measures the stage the AI optimized. The DORA metrics measure whether value reached the customer. When they disagree, trust the metrics that reflect whether value actually reached the customer.

---

## Conclusion

The evidence supports the chapter's thesis most clearly at the task and team levels, with suggestive parallels at the macroeconomic level. AI frequently improves coding-stage activity. Whether those gains survive contact with the full delivery system is a separate question, and one most teams don't currently measure well enough to answer.

The fix isn't to stop using AI. It's to measure honestly enough to find out where it creates value and where it creates the appearance of value. The teams that build that measurement infrastructure will know first. The teams that don't will keep feeling productive while the data — when someone finally collects it — says something different.

The Solow Paradox took a decade to resolve. We might be early. But without instrumentation, optimism just delays the moment you find out whether AI actually helped.

---

## Key References

| Source | Year | Key Finding |
|---|---|---|
| METR RCT (original) | 2025 | 16 devs, 246 tasks; self-reported 20% speedup, measured 19% slowdown; durable finding is the gap between perception and measurement |
| METR follow-up | 2026 | 57 devs, 800+ tasks; -4% effect with wide CI; selection bias discovered; "AI likely provides productivity benefits" |
| METR transcript analysis | 2026 | 1.5x–13x time savings for internal staff on Claude Code; concurrency and task substitution caveats |
| Uplevel Data Labs | 2024 | ~800 devs; telemetry showed no PR cycle time improvement; 41% more bugs for Copilot users; telemetry vs. sentiment divergence |
| NBER "Firm Data on AI" (Yotzov, Barrero, Bloom et al.) | 2026 | ~6,000 executives; 89% report no productivity impact; Solow Paradox parallel |
| NBER "AI, Productivity, and the Workforce" (Baslandze et al.) | 2026 | ~750 CFOs; perceived gains exceed measured gains; productivity paradox formally documented |
| Harvard Business Review, "AI Doesn't Reduce Work" | 2026 | Suggestive finding: workers voluntarily expand workloads with AI tools |
| Jellyfish 20M PR analysis | 2025–2026 | 200K devs, 1K companies; 2x throughput at full adoption; architecture strongly conditions the effect |
| Jellyfish/Harvard collaboration | 2026 | 100K engineers, 500 companies; faster coding, flat business outcomes |
| Faros AI telemetry | 2025 | 10K+ devs; 47% more PRs, no individual task speedup; review times +91% |
| GitClear (211M lines) | 2025 | Code churn doubled since 2021; copy/paste surpassed refactoring; 8x duplication increase |
| GitClear cohort follow-up | 2026 | 4–10x authoring volume for power users; persistent side effects |
| He et al., "Speed at the Cost of Quality" (MSR '26) | 2026 | 807 repos; transient velocity boost, persistent 41% complexity increase; observational evidence of self-reinforcing debt cycle |
| Liu et al., "Debt Behind the AI Boom" | 2026 | 304K commits, 6,275 repos; 24.2% of AI-introduced issues unfixed at latest revision |
| Rahman & Shihab, "Will It Survive?" | 2026 | AI code survives longer than human code; contradicts "disposable code" narrative |
| Tsui et al., "From Technical Debt to Cognitive and Intent Debt" | 2026 | Triple-debt model for the AI era |
| Dubach, "93% Adoption, 10% Gains" | 2026 | Amdahl's Law framing; independent research efforts converge on ~10% organizational gains |
| Atlassian, "Amdahl's Law and AI Inefficiencies" | 2026 | "Maya" scenario illustrating system-level effects of individual-stage speedup |
| DORA State of AI-Assisted Software Development | 2025 | Only already-high-performing teams benefit from AI |
| Forsgren et al., SPACE framework | 2021 | Multi-dimensional productivity measurement |
| GitHub Copilot "55% faster" study | 2022 | One task, 35 completers, no quality check; still cited in sales decks |
| CodeRabbit PR Analysis | 2025 | 1.7x more issues/PR in AI-generated code |
| Borg, Farley et al., "Echoes of AI" | 2025 | No file-level maintainability degradation; volume concerns flagged |
