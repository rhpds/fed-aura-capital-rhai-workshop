# Module 01 — The Agentic App and Why Observability Matters

## Brief Overview

This module introduces participants to the Fed Aura Capital mortgage lending application — a five-agent LangGraph system deployed on Red Hat OpenShift Container Platform. Participants run CLI health checks to confirm the deployment, interact with the CEO agent persona to generate realistic multi-turn conversation data, and experience the observability gap firsthand: the application functions but its internal reasoning, tool calls, and decision paths are invisible to standard platform monitoring. The module establishes the "why" that motivates all subsequent observability work.

## Audience and Time

- **Target personas:** SREs, Platform Engineers, AI Developers/Engineers
- **Prerequisites for this module:** Basic `oc` CLI familiarity; ability to open a web browser and navigate a URL
- **Estimated duration:** 25 minutes

## Learning Objectives

- Explore the Fed Aura Capital multi-agent application architecture and identify the five LangGraph agent personas
- Verify a multi-agent application deployment using `oc` CLI commands and a health-check endpoint
- Demonstrate the observability gap by generating multi-turn conversation data that standard platform metrics cannot explain

## Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Access the application UI | 5 min |
| 2 | Explore the multi-agent architecture | 10 min |
| 3 | Experience the observability gap | 10 min |

## Detailed Steps

1. Retrieve the OpenShift login command from the lab environment and run `oc login` to authenticate to the cluster.
2. Run `oc get pods -n <user-namespace>` to confirm all Fed Aura Capital application pods are Running.
3. Run `oc get route -n <user-namespace>` to retrieve the application URL and the Grafana/MLflow URLs.
4. Run the health-check `curl` against the FastAPI endpoint (`/health`) and confirm a 200 response.
5. Open the React front-end URL in a browser and navigate to the mortgage application UI.
6. Read the architecture diagram in the lab guide explaining the five agent personas: CEO, Prospect, Document Analyzer, Underwriter, and Compliance.
7. Trace a conceptual request flow through the diagram — how a customer query reaches the CEO persona, which downstream agents are invoked, and how results are returned.
8. Verify that each agent microservice pod is running with `oc get pods -l app=fed-aura-capital`.
9. Select the CEO persona in the UI and start a conversation: ask about mortgage pre-qualification for a specific loan amount.
10. Continue the conversation with at least two follow-up questions (e.g., ask about rates, then documentation requirements) to generate multi-turn session data.
11. Open the OpenShift web console and navigate to the pod logs for the FastAPI service — observe that logs show HTTP requests but reveal nothing about which agents ran, what tools they called, or how long each step took.
12. Note the observability gap: the application is working, but the reasoning, tool calls, and per-agent latency are invisible. Record this observation for reference in later modules.

## Key Takeaways

- The Fed Aura Capital application uses five LangGraph agent personas coordinated by a FastAPI service; each agent can invoke tools and call the LLM backend independently.
- Standard platform health checks (`oc get pods`, route health, HTTP status codes) confirm liveness but reveal nothing about agent-internal behavior.
- Multi-turn conversation data is being generated and stored — it can only be analyzed with purpose-built observability tooling introduced in subsequent modules.
- The observability gap for agentic AI is qualitatively different from the observability gap for traditional microservices: agent reasoning, prompt contents, tool authorization decisions, and LLM token budgets are all invisible to conventional APM.

## Infrastructure Notes

- Application must be pre-deployed in per-user namespaces before the lab starts; students do not deploy it.
- MaaS LLM endpoint (`llm-credentials` secret, model `gpt-oss-120b`) must be configured and reachable from the application pods.
- PostgreSQL with pgvector and MinIO PVCs must be provisioned; students do not interact with storage directly in this module.
- Keycloak is present in the application stack but disabled for workshop use — students authenticate via the lab portal, not Keycloak.
