# Behavioral Interview Prep — Manisha B

---

## SELF-INTRODUCTION 🎤

 "Hi, I’m Manisha. I’m a backend-focused full-stack software engineer with over seven years of experience building scalable and reliable systems across the healthcare and automotive industries.

My primary expertise is in Java, Spring Boot, REST APIs, microservices, distributed systems, databases, and CI/CD. I also have frontend experience with React, Angular, which helps me collaborate effectively across the full application stack.

Most recently, I'm with Oracle Health where I worked on internal engineering tools, application modernization, and AI-enabled workflow automation. One of my key projects involved building an AI-assisted documentation workflow using Cline, MCP servers, scripts, structured prompts, and internal APIs. It automated several repetitive validation and documentation steps, improving developer productivity and consistency. I also modernized a legacy SVN application by migrating it to GitHub and restructuring it as a standardized multi-module Maven project.

Before Oracle, I spent about four years at Ford working on Ford Pro Intelligence, a connected-vehicle and fleet-management platform. I developed and supported Java and Spring Boot microservices, APIs, and event-driven data pipelines using technologies such as Kafka, Azure Event Hubs, and GCP Pub/Sub to process large volumes of vehicle telemetry data. I led an observability initiative where I implemented end-to-end distributed tracing across our microservices, improving system visibility and significantly reducing the time required to troubleshoot production issues.

What I enjoy most is owning problems end-to-end, working closely with cross-functional teams, and building reliable, maintainable systems that are easy to support in production."

---

## STORY 1 — OTEL Observability 🎯 *(your anchor)*

**Primary:** Hardest technical problem
**Also reuses for:** Mistake/would-redo · Tough feedback · Learning quickly

**S —** At Ford, my team supported several backend services running on GCP Cloud Run and processing real-time vehicle events through Pub/Sub. At the time, our observability was limited mainly to application logs in Splunk, and we didn’t have distributed tracing. When an issue occurred, debugging meant searching logs across multiple services and manually matching timestamps. It was especially difficult with asynchronous Pub/Sub flows because there was no clear connection between the publishing service and the consuming service.

**T —** I was asked to own the observability work for these services, mainly to improve latency visibility and provide end-to-end distributed tracing.

**A —** My initial approach was to instrument each service manually. I added Micrometer timers to a couple of services, exported the metrics to Prometheus, and created dashboards. The dashboards worked well, but my manager asked whether there was a simpler and more scalable way to onboard future services without modifying their application code each time.

I went back, researched different options, and proposed an OpenTelemetry-based approach. I configured an OTEL Collector as a sidecar alongside each Cloud Run service and used Spring Boot auto-instrumentation, so new services could be onboarded with minimal code changes.

One of the main reasons I chose the OTEL Collector approach was its ability to fan out telemetry to multiple destinations. It allowed me to send telemetry to our existing Prometheus and Grafana setup while also exporting it to the new Grafana Cloud stack we were migrating to. That made the transition smoother because both platforms could continue to work in parallel.

The most challenging part was maintaining trace continuity across Pub/Sub. The trace was getting lost when one service published a message and another service consumed it. I fixed that by propagating the W3C trace context through Pub/Sub message attributes, allowing the same trace ID to continue across the asynchronous flow.

I also handled several supporting changes, including upgrading the services to Spring Boot 3.2 for OpenTelemetry compatibility, updating the GCP Secrets Manager dependency, and migrating the Pub/Sub integration to Spring Cloud GCP so tracing could work correctly. I documented the required setup and updated the onboarding guide for other teams.

**R —** I successfully enabled end-to-end tracing across seven services. On-call debugging time dropped from around twenty minutes of searching through logs to finding the issue through a trace within a few minutes. I also demonstrated the solution to other teams, helped them onboard their services, and became one of the main engineers people reached out to for OpenTelemetry and tracing-related questions.

**🪞 Reflection —** The lesson I keep: before I start building, ask what this looks like at the 10th case, not the 1st. My manager's question saved me from shipping something that didn't scale, and I now ask it myself by default.

---

## STORY 2 — SVN→Maven Migration 🔧

**Primary:** Ambiguous / poorly-scoped problem
**Also reuses for:** Learning quickly · Ownership · Complex technical challenge

**S —** Early at Oracle I was asked to migrate an internal application from SVN to GitHub. On paper, a lift-and-shift. When I opened the repo, it was clearly more — a flat Maven structure with tightly coupled modules, duplicated version-specific submodules, conflicting dependency declarations in individual POMs, and no parent POM enforcing anything. Nobody had run it cleanly from a fresh clone in a while.

**T —** The stated task was "move it to GitHub." The real task was "move it so it's actually maintainable" — and I had to figure that out while still onboarding, with limited context.

**A —** My first plan was one big pass — restructure everything, one PR. The deeper I went, the more hidden dependencies surfaced, so I switched to going module by module and validating each increment. I introduced a parent POM using Oracle's standard base POM so plugin versions and build standards were enforced in one place instead of drifting per module. The trickiest part was internal artifact references with hardcoded URLs that broke in the new environment — I traced each one, mapped it to the right repository in settings.xml, and validated resolution module by module. I also fixed the local dev setup end-to-end and documented the clean-clone-to-running steps.

**R —** By the end, all modules built successfully, and the standardized Maven structure made the application more reliable and easier to maintain. It simplified local setup, reduced build and dependency issues, and made future changes, refactoring, and feature development easier.

**🪞 Reflection —** The lesson was scope honesty — I underestimated because I judged the surface, not the system's health. My fix was to stop trying to understand everything upfront and instead work incrementally and let the real scope reveal itself.

---

## STORY 3 — Config Cleanup Push-Back ⚖️

**Primary:** Conflict / disagreement with a coworker
**Also reuses for:** Influence without authority · Tech-debt vs speed judgment · De-risking a decision · Quality vs delivery pressure

**S —** During a Cloud Run migration at Ford, configuration was spread across multiple places—separate repositories for pre-production and production, along with local configuration files inside each application repository. The migration gave us an opportunity to consolidate everything into the main application repository.

**T —** I suggested cleaning up stale and duplicate configuration values during the migration so the configuration could be reviewed and validated once. My manager preferred migrating everything first and handling the cleanup later because he was concerned that doing both together could affect the delivery timeline. I needed to address that concern and get alignment on the approach.

**A —** I set up a working session and walked through the amount of duplicate and outdated configuration across the repositories. I explained that postponing the cleanup would mean revisiting the same files later, repeating validation, and spending additional time rebuilding context.

To validate my approach without adding unnecessary delivery risk, I applied it to one service first. I migrated and cleaned up the configuration, completed end-to-end validation, and documented the results. During testing, I found a few values that appeared unused but were still referenced, and I resolved those issues before rollout.

I then presented the results and validation report to my manager and the rest of the team. The pilot showed that the approach was manageable and helped identify issues early.

**R —** Based on the pilot results, my manager agreed to use the combined migration-and-cleanup approach for the remaining services. This reduced duplicate configuration, avoided carrying known technical debt into the new environment, and made the configuration easier to maintain.

**🪞 Reflection —** When I disagree with a peer, the move isn't to argue harder — it's to lower the stakes of being wrong. The pilot let us test the disagreement cheaply instead of debating it in the abstract. If I'd been wrong, we'd have found out on one service, not ten.

---

## STORY 4 — Production Incident: Auth-Failure Spike 🚨

**Primary:** Handling pressure / a production incident
**Also reuses for:** Debugging a prod issue · Customer-facing problem · Initiative · Cross-team collaboration

**S — Situation**

At Ford, I was on call for our connected-vehicle backend services. One of our customer-facing endpoints, which fleet customers used to retrieve vehicle status, suddenly started returning a high number of 401 errors.

The traffic had increased because a new group of legitimate customers had recently been enabled, but they were unable to retrieve their vehicle data. Since this was affecting customers in production, it needed immediate attention.

**T — Task**

My priorities were to understand the impact, identify the root cause, restore customer access, and prevent the same issue from happening again.

**A — Action**

I started by analyzing the Splunk logs and breaking down the failures by endpoint and customer ID instead of reviewing the requests one by one.

I noticed that the errors were concentrated around a specific group of customers. After comparing that pattern with recent rollout activity, I found that another team had enabled those customers on their side, but our backend was still using the older customer allowlist in the application configuration.

The customers had valid credentials, but because their IDs were not yet included in our allowlist, our service did not recognize them as authorized and rejected their requests.

I coordinated with the other team to confirm the correct set of newly enabled customer IDs. I then updated the customer allowlist in our application configuration, validated the affected customer flows, completed the review process, and deployed the change.

After deployment, I monitored the 401 error rate and successful request volume to confirm that customer access had been restored.

The deeper issue was that customer-enablement information was maintained separately by two teams, and there was no reliable process to make sure both systems were updated together. To prevent the issue from recurring, I recommended adding a rollout checklist where both teams confirmed the customer list and validated a customer flow before completing future rollouts. I also recommended alerting on unusual increases in authentication or authorization errors.

**R — Result**

The customer impact was resolved the same day, and the 401 error rate returned to its normal level.

The new rollout-verification step reduced the risk of the two systems becoming out of sync again. The incident also highlighted the longer-term need for a shared source of truth or an automated process for keeping customer authorization information synchronized.

---

## STORY 5 — README Automation with MCP 🤖 *(high-reward, high-risk — see warning)*

**Primary:** AI/automation initiative · Differentiator story
**Also reuses for:** Improving developer productivity · Owning something novel end-to-end

**S — Situation**

At Oracle Health, engineers followed a manual process to package and deliver configurations through an internal README workflow. They had to validate scripts through SSH, review README content, generate evidence files, enter information into the Feature Tracker, and create records across multiple internal systems.

The process involved a lot of repetitive work and switching between tools, which made it slow and prone to errors.

**T — Task**

I took ownership of simplifying this process by building an AI-assisted workflow that could automate the repetitive steps while still keeping validation and human review in place.

**A — Action**

I used Cline workflows to define the process step by step, with clear rules and validation checks at each stage.

I also used existing MCP servers to connect the workflow with our internal APIs and tools. This allowed the workflow to retrieve information, run supported actions, and submit validated data instead of only generating text instructions.

The workflow handled several steps, including reviewing PR scripts, parsing the README document, converting the required information into structured JSON, calling the Feature Tracker API, running the README scanner, and generating the final evidence files.

The main challenge was making the AI-generated output consistent and reliable. README content was not always structured in the same way, but the Feature Tracker API expected a specific JSON format. I used clear prompts, examples, and strict formatting rules to guide the output.

I also added validation before any API call was made, so incomplete or incorrectly formatted information could not be submitted to the internal system. If something was missing or invalid, the workflow stopped and asked the engineer to review it.

I tested the workflow with different inputs, refined the prompts and validation rules, and documented how engineers could use and troubleshoot it.

**R — Result**

The workflow removed much of the manual data entry, command execution, and switching between tools. It made the process faster, more consistent, and easier to track.

I presented it to the team as a proof of concept for broader AI-assisted engineering automation, and my manager recognized it as aligned with the direction the team wanted to pursue.

**Reflection**

The main lesson for me was that AI automation is useful only when it has strong validation and human checkpoints. The goal was not just to automate the process, but to make it reliable and safe when interacting with internal systems.

---



## "WHY THIS COMPANY / WHY THIS ROLE" 🎯







---



