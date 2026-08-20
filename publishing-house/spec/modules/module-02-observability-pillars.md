# Module 02 — Observability Pillars, Concepts, and Personas

## Brief Overview

This conceptual module builds the mental model that participants need to interpret everything they will do in Modules 3–6. Starting from the CEO conversation generated in Module 1, participants map the three observability pillars — metrics, logs, and traces — to specific operational questions in the Fed Aura Capital mortgage system. The module also distinguishes two distinct personas — SRE and AI Developer — and the different observability questions each role asks. No hands-on exercises with tooling; this module is entirely reading, discussion, and a structured mapping activity.

## Audience and Time

- **Target personas:** SREs, Platform Engineers, AI Developers/Engineers
- **Prerequisites for this module:** Completion of Module 1; the CEO conversation from Module 1 is used as a reference throughout
- **Estimated duration:** 15 minutes

## Learning Objectives

- Analyze the three observability pillars (metrics, logs, traces) and map each to specific operational questions for the Fed Aura Capital mortgage AI system
- Demonstrate how SRE and AI Developer personas require different observability signals from the same application
- Identify which pillar would answer a given operational question (e.g., "why did this conversation take so long?" vs. "is the service healthy?")

## Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Revisit the CEO conversation from Module 1 | 3 min |
| 2 | The three pillars: metrics, logs, traces | 7 min |
| 3 | Two personas, one observability stack | 5 min |

## Detailed Steps

1. Re-open the CEO conversation from Module 1 in the application UI (or review the transcript if preserved). Identify two or three moments where you could not tell from the UI alone what was happening internally.
2. Read the definitions of the three observability pillars as presented in the lab guide:
   - **Metrics:** Quantitative time-series data — request counts, latency percentiles, error rates, LLM token usage, tool-call success rates
   - **Logs:** Discrete event records — structured JSON entries from each agent, request/response bodies, error stack traces
   - **Traces:** Causal chains of spans — end-to-end records of a single request flowing through multiple agents, LLM calls, and tool invocations
3. Work through the pillar-mapping table in the lab guide: for each of five sample operational questions, decide which pillar (or combination) answers it. Example questions: "Is the service healthy?", "Why did this loan application take 45 seconds?", "What prompt did the Compliance agent receive?", "How many tool calls did the CEO agent make this week?"
4. Read the SRE persona section: what an SRE needs (service-level indicators, error budgets, alert thresholds, capacity signals) and which pillars are most relevant (primarily metrics and logs).
5. Read the AI Developer persona section: what an AI developer needs (prompt inspection, per-agent latency breakdown, tool-call result tracing, evaluation scores) and which pillars are most relevant (primarily traces and evaluation results).
6. For the CEO conversation from Module 1, identify at least one question that the SRE persona would ask and one that the AI Developer persona would ask. Note which pillar would answer each.

## Key Takeaways

- Metrics answer "what is happening and how often"; logs answer "what happened in a specific event"; traces answer "why did this specific request behave this way."
- Agentic AI introduces a fourth invisible layer — model reasoning — that traces can partially surface (prompts, completions, tool decisions) but cannot fully explain.
- The SRE and AI Developer personas look at the same application through different lenses: SREs optimize for service reliability; AI developers optimize for agent reasoning quality.
- The Red Hat OpenShift AI observability stack (MLflow, Grafana, LokiStack) addresses all three pillars for agentic workloads — each tool is introduced in subsequent modules.

## Infrastructure Notes

- No cluster access required in this module; it is a reading and discussion activity.
- The CEO conversation from Module 1 must be accessible (either in the browser or as a transcript) — no new application interaction is needed.
- If delivering instructor-led, this module can be condensed to a 10-minute discussion; the mapping exercise can become a group whiteboard activity.
