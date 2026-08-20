# Module 04 — Tracing and MLflow

## Brief Overview

This module is the observability capstone for the core lab path. Participants access the RHOAI-managed MLflow tracking server and trace the CEO agent conversations they generated in Module 1 — end-to-end, from the initial HTTP request through input/output shields, LLM calls (with full prompt and completion text), tool authorization decisions, and tool results. One `mlflow.langchain.autolog()` call in the application code provides the complete trace; participants drill into each span without writing any code. The module closes with MLflow's user-session grouping feature, which aggregates multi-turn conversations for analysis.

## Audience and Time

- **Target personas:** SREs, Platform Engineers, AI Developers/Engineers
- **Prerequisites for this module:** Completion of Modules 1–3; the CEO conversation from Module 1 is the primary artifact analyzed here
- **Estimated duration:** 35 minutes

## Learning Objectives

- Configure and use MLflow tracing (via `mlflow.langchain.autolog()`) to understand how a single API call enables full end-to-end span capture for LangGraph agents
- Analyze a CEO agent trace end-to-end, drilling into input/output shield spans, LLM call spans (with prompts and completions), tool authorization spans, and tool result spans
- Explore multi-turn conversation tracing using MLflow user-session grouping to analyze conversation context across turns

## Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Access MLflow tracking server | 5 min |
| 2 | How tracing works: autolog and OpenTelemetry | 5 min |
| 3 | Find your first trace | 5 min |
| 4 | Analyze CEO trace end-to-end | 15 min |
| 5 | Explore user sessions | 5 min |

## Detailed Steps

**Exercise 1: Access MLflow tracking server**

1. Open the MLflow tracking server URL (retrieved in Module 1 via `oc get route -n <rhoai-namespace>`).
2. Log in with the credentials provided in the lab guide. Confirm you see the MLflow Experiments list.
3. Navigate to the experiment named `fed-aura-capital` (or the experiment pre-configured for the workshop).
4. Note the experiment structure: each run represents one request to the application.

**How tracing works: autolog and OpenTelemetry**

5. Read the code snippet in the lab guide showing the single instrumentation line in the application: `mlflow.langchain.autolog()`. No additional code is required — autolog captures all LangGraph spans automatically.
6. Read the explanation of how MLflow uses OpenTelemetry under the hood: each LangGraph node becomes an OpenTelemetry span; the span tree is exported to the MLflow tracking server.
7. Note the OpenTelemetry compatibility benefit: the same trace data can be exported to other OTLP-compatible backends (Jaeger, Tempo) without application changes.

**Exercise 2: Find your first trace**

8. In MLflow, filter the run list by the timestamp range of the Module 1 CEO conversation.
9. Click the first run to open its trace detail view.
10. Observe the span tree at the top level: identify the root span (the HTTP request to the FastAPI endpoint) and its immediate children.

**Exercise 3: Analyze CEO trace end-to-end**

11. Expand the input shield span. Observe: what content was checked, what policy was applied, whether the check passed.
12. Expand the first LLM call span. Note these fields: model name, prompt tokens, completion tokens, latency, the full prompt text, and the full completion text. This is the information that was invisible in Modules 1–2 standard monitoring.
13. Expand the tool authorization span (if present). Observe: which tool the agent requested, what the authorization decision was, and whether the tool was invoked or blocked.
14. Expand the tool result span. Observe: the tool's input parameters and its returned data.
15. Expand the output shield span. Observe: what the post-generation content check found and whether any content was filtered.
16. Calculate the total duration of the CEO agent's response by summing the major span durations and comparing to the root span duration — identify where time was spent.
17. Identify one specific span that accounts for the largest share of latency. Discuss what optimization would target that span.

**Exercise 4: Explore user sessions**

18. In MLflow, navigate to the Sessions view (or filter by the `mlflow.session_id` tag if sessions are shown as tagged runs).
19. Find the session corresponding to the multi-turn CEO conversation from Module 1 (use the timestamp or session ID from the lab guide).
20. Expand the session to see all turns (runs) in the conversation. Observe how context accumulates across turns: the second and third turns include prior conversation history in their LLM call prompts.
21. Compare prompt token counts across turns — note how they grow as conversation context is appended.

## Key Takeaways

- `mlflow.langchain.autolog()` provides zero-code tracing for LangGraph applications: all agent spans, LLM calls (with prompts and completions), and tool invocations are captured automatically.
- MLflow traces make visible what standard platform monitoring cannot: the exact prompt sent to the LLM, the agent's reasoning chain, which tools were called and with what parameters, and where latency was incurred.
- OpenTelemetry compatibility means MLflow tracing integrates with the broader observability ecosystem without vendor lock-in.
- MLflow user-session grouping is essential for multi-turn conversation analysis: it aggregates per-turn traces into a coherent conversation view, revealing how context grows and how each turn's behavior depends on prior turns.
- The combination of Modules 3 and 4 (metrics/logs + traces) gives SREs and AI developers complementary views: aggregate health signals plus per-request causal analysis.

## Infrastructure Notes

- The RHOAI MLflow tracking server must be deployed and accessible before the lab starts.
- The Fed Aura Capital application must have `mlflow.langchain.autolog()` enabled in its startup code and the `MLFLOW_TRACKING_URI` environment variable pointing to the RHOAI MLflow server.
- MLflow session grouping requires the application to pass a consistent `mlflow.session_id` tag per user conversation — this must be implemented in the application and verified during setup.
- No GPU access is needed; the LLM backend is the in-cluster MaaS endpoint.
