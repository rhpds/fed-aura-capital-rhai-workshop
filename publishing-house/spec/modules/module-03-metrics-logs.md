# Module 03 — Metrics and Logs for Agentic Applications

## Brief Overview

This module moves participants from concept to hands-on tooling across two exercises. In the first, participants navigate the pre-deployed Grafana dashboard for the Fed Aura Capital application and interpret both standard HTTP metrics and AI-specific agentic metrics — LLM token usage, inference latency, and tool-call success rates. In the second, they use LokiStack's log viewer with LogQL to scope, filter, and extract structured JSON application logs from the multi-agent service. The module closes by demonstrating how logs and traces are complementary: logs provide the event record; traces provide the causal chain.

## Audience and Time

- **Target personas:** SREs, Platform Engineers, AI Developers/Engineers
- **Prerequisites for this module:** Completion of Modules 1–2; access to Grafana URL and OpenShift web console
- **Estimated duration:** 30 minutes

## Learning Objectives

- Monitor AI-specific metrics — LLM token usage, inference latency, tool-call success rates — using a pre-deployed Grafana dashboard
- Analyze structured application logs for a multi-agent system using LokiStack and LogQL queries
- Demonstrate how log data and trace data are complementary observability signals for agentic applications

## Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Explore the Mortgage-AI Grafana dashboard | 15 min |
| 2 | Explore application logs with LokiStack | 12 min |
| 3 | Logs and traces are complementary | 3 min |

## Detailed Steps

**Exercise 1: Explore the Mortgage-AI Grafana dashboard**

1. Open the Grafana URL retrieved in Module 1. Log in with the credentials provided in the lab guide.
2. Navigate to Dashboards and open the "Mortgage-AI" dashboard.
3. Examine the Overview / HTTP panels: note the request rate, error rate, and P95 latency across all API endpoints.
4. Run `oc get grafanadashboard -n <monitoring-namespace>` to see the GrafanaDashboard CRD resource that defines this dashboard — understanding that dashboards are managed as OpenShift resources.
5. Scroll to the AI agent metrics section. Identify these panels and what they measure:
   - LLM token usage per request (prompt tokens, completion tokens)
   - Inference latency (time-to-first-token, total generation time)
   - Tool-call success rate (ratio of successful tool invocations)
   - Agent invocation frequency (which agents are called most often)
6. Adjust the time range to cover the period of the CEO conversation from Module 1 and observe whether the conversation's activity is visible in the metrics.
7. Note which metrics are most relevant to the SRE persona vs. the AI Developer persona (reinforcing Module 2 concepts).

**Exercise 2: Explore application logs with LokiStack**

8. In the OpenShift web console, navigate to Observe > Logs (or the Logging section powered by LokiStack).
9. Scope the log stream to the `fed-aura-capital` application in your namespace using a LogQL label selector: `{namespace="<user-namespace>", app="fed-aura-capital"}`.
10. Apply a time filter covering the Module 1 conversation window.
11. Examine a sample log entry and identify these JSON fields: timestamp, agent_name, request_id, message, duration_ms, tool_called (if present), llm_tokens (if present).
12. Narrow the log stream to a single agent using a more specific label: `{namespace="<user-namespace>", agent="ceo"}`.
13. Run a LogQL filter to extract only entries where a tool was called: `{...} | json | tool_called != ""`.
14. Run a LogQL metric query to count log entries per agent over time: `sum by (agent) (count_over_time({namespace="<user-namespace>", app="fed-aura-capital"}[5m]))`.
15. Read the section in the lab guide on how AI can help analyze log patterns — LLMs can summarize structured log anomalies, but this capability requires caution about log volume and PII.

**Closing: logs and traces are complementary**

16. Review the diagram in the lab guide showing how a single request ID appears in both logs (individual events per agent) and traces (the full causal span tree). Note that logs answer "what happened in this event" while traces answer "how did this event relate to all other events in the request."

## Key Takeaways

- AI-specific Grafana panels (LLM tokens, inference latency, tool-call success rates) provide aggregate signals that standard HTTP metrics cannot capture — they are essential for SLO definition on agentic workloads.
- LokiStack with LogQL enables scoped, structured log querying for multi-agent applications; JSON log fields (agent name, request ID, tool calls) are first-class query dimensions.
- The GrafanaDashboard CRD pattern means observability configuration is version-controlled and cluster-native — dashboards are not hand-managed in the Grafana UI.
- Logs and traces answer complementary questions; neither alone is sufficient for root-cause analysis of agentic failures.

## Infrastructure Notes

- Grafana operator and GrafanaDashboard CRD must be deployed before the lab starts; the Mortgage-AI dashboard must be pre-loaded.
- LokiStack and OpenShift Logging must be configured to collect logs from the `fed-aura-capital` namespace.
- User Workload Monitoring (Prometheus) must be enabled and scraping the FastAPI service's `/metrics` endpoint.
- PromQL examples referenced in the lab guide are provided as copy-paste snippets; students are not expected to write raw PromQL.
