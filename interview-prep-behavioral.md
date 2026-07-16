# Behavioral Interview Prep — Manisha B

> **How to use this doc:** These are your 4 honest anchor stories + self-intro, covering ~9 of the top 10 behavioral questions. Read them out loud, time yourself, and fill in every `⚠️ CONFIRM` marker with a real detail from your own memory. The markers are the spots most likely to get probed in follow-ups.
>
> **The three rules baked into every story here:**
>
> 1. **Hedge numbers, never fake precision.** "A couple of minutes" survives follow-up; "80% reduction" dies to "how'd you measure that?"
> 2. **Claim "I owned X," not "it became the org standard."** Your honest scope is strong enough on its own.
> 3. **End on genuine learning.** At the level you're targeting, the Reflection is what scores — not heroics.

---



## Coverage Map — 4 stories cover the top 10 questions


| #   | Question                                                                  | Covered by                                  |
| --- | ------------------------------------------------------------------------- | ------------------------------------------- |
| 1   | Tell me about yourself                                                    | Self-Intro                                  |
| 2   | Most challenging technical problem                                        | **OTEL**                                    |
| 3   | A mistake / something you'd do differently                                | **OTEL** (manual-first wrong path)          |
| 4   | Disagreement / tough feedback                                             | **Config Push-Back** / OTEL feedback        |
| 5   | Learn something quickly                                                   | **Migration**                               |
| 6   | Ambiguous / poorly-scoped problem                                         | **Migration**                               |
| 7   | Improved something / took initiative                                      | **401 Incident** (systemic fix)             |
| 8   | Handled pressure / an incident                                            | **401 Incident**                            |
| 9   | Why this company / why this role                                          | ✅ **Why This Company** (Procore + template) |
| 10  | Cross-team collaboration                                                  | **401 Incident** / Config                   |
| 11  | Tell me about an AI/automation project · improving developer productivity | **README Automation**                       |
| 12  | Building/owning something novel end-to-end                                | **README Automation**                       |


> ⚠️ **Sequencing note:** OTEL and Config both live at Ford's Cloud Run migration. If asked for two examples in one session, don't tell them back-to-back — lead OTEL with the *technical* angle and Config with the *people/judgment* angle. Pull the 401 story from the incident angle to add range.

---



## SELF-INTRODUCTION 🎤

> "Hi, I’m Manisha. I’m a backend-focused full-stack software engineer with over seven years of experience building scalable and reliable systems across the healthcare and automotive industries.
>
> My primary expertise is in Java, Spring Boot, REST APIs, microservices, distributed systems, databases, and CI/CD. I also have frontend experience with React, Angular, which helps me collaborate effectively across the full application stack.
>
> Most recently, I'm with Oracle Health where I worked on internal engineering tools, application modernization, and AI-enabled workflow automation. One of my key projects involved building an AI-assisted documentation workflow using Cline, MCP servers, scripts, structured prompts, and internal APIs. It automated several repetitive validation and documentation steps, improving developer productivity and consistency. I also modernized a legacy SVN application by migrating it to GitHub and restructuring it as a standardized multi-module Maven project.
>
> Before Oracle, I spent about four years at Ford working on Ford Pro Intelligence, a connected-vehicle and fleet-management platform. I developed and supported Java and Spring Boot microservices, APIs, and event-driven data pipelines using technologies such as Kafka, Azure Event Hubs, and GCP Pub/Sub to process large volumes of vehicle telemetry data. I led an observability initiative where I implemented end-to-end distributed tracing across our microservices, improving system visibility and significantly reducing the time required to troubleshoot production issues.
>
> What I enjoy most is owning problems end-to-end, working closely with cross-functional teams, and building reliable, maintainable systems that are easy to support in production."

---



## STORY 1 — OTEL Observability 🎯 *(your anchor)*

**Primary:** Hardest technical problem
**Also reuses for:** Mistake/would-redo · Tough feedback · Learning quickly

**S —** At Ford, my team ran a set of backend services on GCP Cloud Run handling real-time vehicle events through Pub/Sub. Our observability was thin — application logs in Splunk and not much else. No distributed tracing. So debugging meant grepping logs across services and hoping timestamps lined up, especially painful across async Pub/Sub hops where there's no obvious thread connecting a producer and consumer.

**T —** I was asked to own observability for our services — latency visibility and end-to-end tracing across them.

**A —** My first instinct was to instrument each service by hand — I wired Micrometer timers into a couple of services and piped metrics to Prometheus. The dashboards demoed well. But my manager asked a question that reframed the whole thing: *"what happens when we onboard the next several services — are you hand-instrumenting each one?"* He was right; I'd been thinking too narrowly.

I moved to an OpenTelemetry approach — an OTEL Collector running as a sidecar alongside each Cloud Run service, with apps auto-instrumented via Spring Boot 3.2, so onboarding a new service didn't mean rewriting its code. The Collector let us export telemetry to both our legacy Prometheus/Grafana stack and the newer Grafana Cloud LGTM stack at once, so neither team was blocked during the migration.

The hardest part was async trace continuity: traces died at the Pub/Sub boundary. I had to get trace context propagated through message attributes so a single trace ID followed a request across the async hop. Along the way I navigated real blockers — upgrading services to Spring Boot 3.2 for OTEL compatibility, a dependency on GCP Secrets Manager v2, and migrating the Pub/Sub libraries to Spring Cloud GCP to enable tracing. I documented the prerequisites and updated the onboarding guide.

**R —** I got tracing working across our 7 services. On-call debugging went from hunting through logs for ~20 minutes to pulling up a trace in a couple of minutes. I demoed it to other teams, a few onboarded their own services, and I became one of the go-to people — along with my manager and one other engineer — when they hit questions.

**🪞 Reflection —** The lesson I keep: before I start building, ask what this looks like at the 10th case, not the 1st. My manager's question saved me from shipping something that didn't scale, and I now ask it myself by default.

> ⚠️ **CONFIRM / HOMEWORK:**
>
> - **Review W3C trace-context propagation through Pub/Sub** before using this. The follow-up "how exactly did the trace ID survive Pub/Sub?" is near-certain. If you can't go deep, soften to "I got context propagated through message attributes" and don't over-claim depth.
> - Keep "Spring Boot 3.2 / Secrets Manager v2 / LGTM" only if you actually recall these — they're great *if* real, but they're precise and probeable.

⏱️ Target: ~4 minutes

---



## STORY 2 — SVN→Maven Migration 🔧

**Primary:** Ambiguous / poorly-scoped problem
**Also reuses for:** Learning quickly · Ownership · Complex technical challenge

**S —** Early at Oracle I was asked to migrate an internal application from SVN to GitHub. On paper, a lift-and-shift. When I opened the repo, it was clearly more — a flat Maven structure with tightly coupled modules, duplicated version-specific submodules, conflicting dependency declarations in individual POMs, and no parent POM enforcing anything. Nobody had run it cleanly from a fresh clone in a while.

**T —** The stated task was "move it to GitHub." The real task was "move it so it's actually maintainable" — and I had to figure that out while still onboarding, with limited context.

**A —** My first plan was one big pass — restructure everything, one PR. The deeper I went, the more hidden dependencies surfaced, so I switched to going module by module and validating each increment. I introduced a parent POM using Oracle's standard base POM so plugin versions and build standards were enforced in one place instead of drifting per module. The trickiest part was internal artifact references with hardcoded URLs that broke in the new environment — I traced each one, mapped it to the right repository in settings.xml, and validated resolution module by module. I also fixed the local dev setup end-to-end and documented the clean-clone-to-running steps.

**R —** All modules built cleanly and the app ran from a fresh clone, which hadn't happened in a while. The parent POM meant modules couldn't silently drift from standards going forward, and onboarding got easier because the setup was documented.

**🪞 Reflection —** The lesson was scope honesty — I underestimated because I judged the surface, not the system's health. My fix was to stop trying to understand everything upfront and instead work incrementally and let the real scope reveal itself.

> ⚠️ **CONFIRM:** Keep timeline honest — this was early at Oracle. Don't imply it was fully wrapped and presented at a review within your first few weeks; that math invites scrutiny.

⏱️ Target: ~3.5–4 minutes

---



## STORY 3 — Config Cleanup Push-Back ⚖️

**Primary:** Conflict / disagreement with a coworker
**Also reuses for:** Influence without authority · Tech-debt vs speed judgment · De-risking a decision · Quality vs delivery pressure

**S —** During a Cloud Run migration at Ford, we were restructuring how config was managed. Historically config was scattered — two separate repos for pre-prod and prod, plus local config files inside each app repo. The migration was a chance to consolidate all of it into the base application repo.

**T —** I proposed we clean up stale and redundant config values *during* the migration, so we'd validate everything once. A peer engineer disagreed — they wanted to migrate first and defer cleanup, worried that doing both at once would slow delivery. We were at the same level, so this wasn't something I could just decide over them — I had to actually align with them.

**A —** Rather than push my view harder, I set up a working session to get us on the same page. I walked through how much redundancy was actually sitting across the two config repos, and made the case that deferring wasn't free — later cleanup would mean re-reading every file, re-identifying stale values, re-running validation, and re-coordinating a review. We'd pay the context-switching cost twice.

To de-risk it — because their delivery concern was legitimate — I proposed we not commit to everything upfront. We'd run it on *one* service: migrate and clean in a single pass, validate end-to-end, and extend only if it held up. The pilot did surface a couple of issues — a few config values I'd assumed were unused turned out to still be referenced — but we caught them in validation and handled them before they mattered.

**R —** The pilot passed end-to-end, and the issues it surfaced were exactly the kind that would've been painful to hit later. That convinced my peer — we adopted the combined migrate-and-clean approach for the rest of our services. Beyond the time saved, it meant we weren't leaving known tech debt behind us as we migrated.

**🪞 Reflection —** When I disagree with a peer, the move isn't to argue harder — it's to lower the stakes of being wrong. The pilot let us test the disagreement cheaply instead of debating it in the abstract. If I'd been wrong, we'd have found out on one service, not ten.

> ⚠️ **CONFIRM:** The most probeable line is *"what were the 1–2 issues the pilot surfaced?"* I wrote "config values assumed unused but still referenced." Keep that only if true — otherwise swap in the real detail.

⏱️ Target: ~3.5 minutes

---



## STORY 4 — Production Incident: Auth-Failure Spike 🚨

**Primary:** Handling pressure / a production incident
**Also reuses for:** Debugging a prod issue · Customer-facing problem · Initiative · Cross-team collaboration

**S —** At Ford, I was on-call for our Connected Vehicle backend services. One endpoint — the one fleet customers hit to get vehicle status — suddenly started throwing a wave of 401 authentication errors. Traffic on that endpoint had jumped well above its normal baseline overnight, and real fleet customers were affected: they couldn't pull vehicle data. Customer-facing and live.

**T —** Three jobs, in order: stop the customer impact, find the actual root cause, and make sure this class of problem didn't just come back next week.

**A —** My first move was to segment the logs rather than read them top-to-bottom. I broke the traffic down by customer ID and by endpoint in Splunk, looking for the shape of the failure. The pattern showed up fairly quickly — the 401s weren't random or malicious; they clustered around a specific set of customers who all had one thing in common.

That pointed at the root cause: those customers had just been enabled through a rollout on another team's side, but our backend was still only recognizing the original, smaller authorized set. Legitimately-enabled customers were hitting our endpoint and getting rejected because our service didn't know about them yet — the two sides had drifted out of sync.

For the immediate fix, I coordinated with the other team to confirm the correct set of newly-activated customers, updated our side, got it reviewed, and deployed — which cleared the errors. From alert to root cause was a few hours; finding the pattern in the logs was fast, confirming the customer list across teams was the slower part.

**R —** Customer impact was resolved the same day once we corrected the mismatch. The part I cared about more was preventing the repeat: this happened because one team's rollout and our backend's authorized list weren't coordinated, and nothing caught the gap. So I pushed for a coordination step between the teams before that kind of rollout goes out, and flagged that we should alert on sudden traffic spikes on that endpoint so we'd see it *before* customers did.

**🪞 Reflection —** The real fix for an incident usually isn't the hotfix — it's closing the gap that let it happen. Patching the customer list solved that morning. The coordination step and the alert solved the actual problem: two teams could drift out of sync silently.

> ⚠️ **CONFIRM (4 spots — these are your own memory, fill them truthfully):**
>
> 1. **Root cause** — I wrote "another team enabled customers our backend didn't recognize." If the real cause was different (token/cert/gateway config), swap it for what you can explain.
> 2. **Immediate fix** — I wrote "updated the authorized customer list." Confirm the real remediation.
> 3. **Systemic fix** — you told me the real fix was *improved monitoring/alerting*, so the alert is solid. Confirm the cross-team coordination piece is real, or drop it.
> 4. **The spike** — I wrote "well above baseline," NOT "26x." Only say a specific multiple if you genuinely remember measuring it.

⏱️ Target: ~3.5–4 minutes

---



## STORY 5 — README Automation with MCP 🤖 *(high-reward, high-risk — see warning)*

**Primary:** AI/automation initiative · Differentiator story
**Also reuses for:** Improving developer productivity · Owning something novel end-to-end

> 🚨 **READ THIS FIRST.** This is your highest-*ceiling* story (MCP is current and rare) and now that it's framed accurately, it's defensible. Be precise about the boundary of what you did: **you *used* existing MCP servers to connect AI tooling to internal APIs — you did NOT build an MCP server from scratch.** Say "used," never "built one," or you'll invite implementation questions you can't answer. The real strength here is your *workflow design, prompt engineering, and validation layer* — lead with those.

**S —** At Oracle, engineers had a manual, repetitive workflow to package and validate configurations for a README process — running scripts, generating evidence files, and hand-entering data into an internal tracking tool. It was slow, error-prone coordination work across several systems.

**T —** I wanted to reduce that manual effort by automating the workflow end-to-end, using AI as the integration layer rather than writing one brittle script.

**A —** I designed it as a guided, step-by-step workflow rather than a single script, because it needed to parse input documents, make decisions, and coordinate across systems. I used MCP servers to connect the AI tooling to our internal APIs — so the AI agent could actually call our systems rather than just generate text suggestions. I built the workflow around that: Cline-based workflows with embedded rules and validation checkpoints at each step, and structured prompts for each stage — parsing the README document, converting it to the exact JSON schema our internal tracker's API expected, and generating the final evidence output.

The trickiest part was making the AI reliably turn an unstructured document into the *exact* schema the API required. I handled that with few-shot examples and strict output constraints in the prompt, then added a validation layer that checked the output *before* any API call was made — so a malformed AI response could never reach a live internal system. I also wired it into a Jenkins pipeline so it could trigger automatically.

**R —** The workflow cut out most of the manual tool-switching and hand-entry, and made the process more consistent and repeatable. I presented it to the team as a proof-of-concept for broader AI-assisted automation, and my manager flagged it as aligned with where the team wanted to go.

**🪞 Reflection —** What I took away is that giving an AI agent real access to backend systems is powerful, but the guardrails matter as much as the capability — the validation layer before any API call was the part that made it safe to actually use.

> ⚠️ **CONFIRM / HOMEWORK (mandatory before using):**
>
> 1. **The critical distinction:** you *used* MCP servers, you did NOT build one. Phrase it as "I used MCP to connect the AI tooling to our internal APIs." Be ready to explain, at a *user's* level, what MCP does (lets an AI agent call real tools/APIs through a standard interface) — but don't claim server-implementation depth you don't have.
> 2. Be ready for **"how did you validate LLM output before the API call?"** — schema validation is the honest answer; know how it worked. This is now your strongest defensible technical point, so lean into it.
> 3. **Scope honesty:** you started Oracle ~Oct 2025. Frame this as a *proof-of-concept you built*, NOT a fully-adopted production framework.
> 4. Drop the specific "70–80% reduction" — say "cut out most of the manual steps" unless you truly measured it.

⏱️ Target: ~4 minutes

---



## Migration vs README — which to lead with 🥊


|                           | SVN→Maven Migration           | README Automation                                        |
| ------------------------- | ----------------------------- | -------------------------------------------------------- |
| Defensibility             | ✅ High (concrete: POMs, deps) | ✅ Good now it's accurate (design + validation are yours) |
| Timeline credibility      | ✅ Fine                        | ⚠️ Tight (started Oct 2025) — frame as proof-of-concept  |
| "Is it real engineering?" | ✅ Unambiguous                 | ✅ Yes (workflow design + schema validation)              |
| Wow factor                | Solid                         | ✅ High (MCP is rare)                                     |


**Verdict:** Migration = your reliable **floor** (never collapses). README = high-ceiling and now safe to tell, *as long as you say "used MCP," not "built an MCP server."* Both are usable; lead with Migration for "ambiguous/learning-fast," deploy README when you want to stand out.

---



## "WHY THIS COMPANY / WHY THIS ROLE" 🎯



### Reusable Template (works for any company)

Fill the `[brackets]` with 10 minutes of research on the specific company:

> "Three things draw me to `[company]`.
>
> First, **the problem space** — `[company]` is `[one specific thing about what they build / who they serve]`, and I like that the backend work there has real weight: `[reliability / scale / the systems are core to how customers operate]`.
>
> Second, **the fit with how I work.** The kind of engineering I do best is `[owning backend systems end-to-end / building tooling that makes other engineers faster / making distributed systems observable and reliable]` — and from the role, that's `[exactly what this team does / a big part of this platform work]`.
>
> Third, **where I want to grow.** I want to keep working on `[systems at scale / platform-level problems / reliability]`, and this role puts me right in that path rather than adjacent to it.
>
> So it's not a generic backend role to me — it lines up with both what I've done and where I want to go."

**Rules for using it:**

- **Lead with the company, not yourself.** Show you researched *them* before you pivot to fit.
- **One specific, non-obvious detail** proves you did homework. Avoid "I love your mission" (everyone says it).
- **Keep it ~45–60 seconds.** This isn't a story; it's a signal of genuine interest + self-awareness.



### Worked Example — Procore Technologies

*Spine: reason #3 (building tools that multiply engineers), with #1/#2 as support.*

> "A few things draw me specifically to Procore.
>
> First, the mission resonates — Procore builds the software that builds the physical world, for an industry that historically hasn't had great tooling. That's meaningful backend work: the systems have to be reliable because real construction projects depend on them.
>
> Second, and most important to me — I noticed the platform work here is about empowering other teams to build on top of a common core. That maps directly to the work I care about most. At Ford, the thing I was proudest of wasn't a customer feature — it was building observability that made every engineer on my team debug faster. I like being the person who builds the thing that makes *other* engineers more effective, and that's exactly what platform and internal-tooling work at Procore is.
>
> Third, the scale is a genuine draw — core systems serving a large user base, a real microservices footprint, and a serious investment in observability. I've worked on distributed services and spent real time on the reliability side, so that's a challenge I want more of, not less.
>
> I'll be honest that my core stack has been Java and Spring rather than Ruby on Rails — but the backend and distributed-systems thinking transfers directly, and I'd be genuinely excited to pick up the Rails side."

**Why this version works:**

- Names a **real, specific** Procore trait (platform/common-core empowering other teams) — proves research.
- **Ties it to your actual OTEL story** — motivation backed by evidence, not just enthusiasm.
- **Proactively addresses the Ruby/Rails gap** honestly — Procore's stack is Ruby/Rails/Postgres/AWS, not Java. Naming it first shows self-awareness and defuses their concern before they raise it.

> ⚠️ **CONFIRM before your Procore interview:** Re-check the *specific* role/JD you apply to — team, product area, and whether it's backend/platform/tooling. Swap in one detail from *that* JD so this doesn't sound like a company-level generic.

---



## GENERAL DELIVERY REMINDERS

- **When you don't remember a detail:** say "I'd have to check the exact figure, but roughly..." — hedging is *credible*, not weak. Never invent precision on the spot.
- **Every story ends on the Reflection** — don't let it trail off after the Result.
- **If pressed beyond what you recall**, pivot to what you learned or what you'd do now. That's always defensible.

