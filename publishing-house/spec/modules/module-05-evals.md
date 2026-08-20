# Module 05 — Agent and LLM Evaluations (Optional)

## Brief Overview

This optional advanced module introduces systematic LLM evaluation using Red Hat OpenShift AI's Jupyter workbench environment and MLflow's Evaluation and Prompt Registry features. Participants launch a RHOAI workbench, clone the multi-agent-loan-origination repository, and run the `evaluate_agent.ipynb` notebook. The notebook registers the prospect agent's system prompt in the MLflow Prompt Registry, builds a six-case evaluation dataset, and executes two scorer types: deterministic rule-based scorers and an LLM-as-a-Judge scorer. Results are analyzed per-trace with prompt-trace linkage, closing the loop between prompt versioning and observed behavior.

## Audience and Time

- **Target personas:** AI Developers/Engineers (primarily); SREs with Python familiarity
- **Prerequisites for this module:** Completion of Module 4; basic Python literacy; ability to read and run Jupyter notebook cells
- **Estimated duration:** 55 minutes

## Learning Objectives

- Deploy a RHOAI Jupyter workbench and run an evaluation notebook against a live MLflow tracking server
- Build an evaluation dataset and configure deterministic and LLM-as-a-Judge scorers for a LangGraph agent
- Analyze per-trace evaluation results and explore prompt-trace linkage using the MLflow Prompt Registry

## Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Launch Jupyter workbench | 5 min |
| 2 | Clone repo and open evaluation notebook | 5 min |
| 3 | Register prompt in MLflow Prompt Registry | 10 min |
| 4 | Create evaluation dataset | 10 min |
| 5 | Run simple evaluation (deterministic scorers) | 10 min |
| 6 | Run LLM-as-a-Judge evaluation | 10 min |
| 7 | Explore prompt-trace linkage | 5 min |

## Detailed Steps

**Exercise 1: Launch Jupyter workbench**

1. Open the Red Hat OpenShift AI dashboard (URL provided in the lab guide).
2. Navigate to Data Science Projects and select your project namespace.
3. Launch the pre-configured Jupyter workbench. Wait for the workbench pod to reach Running state.
4. Open JupyterLab in the browser.

**Exercise 2: Clone repo and open evaluation notebook**

5. Open a terminal in JupyterLab and run:
   ```
   git clone https://github.com/rh-ai-quickstart/multi-agent-loan-origination.git
   ```
6. Navigate to `multi-agent-loan-origination/notebooks/` and open `evaluate_agent.ipynb`.
7. Read the notebook's introductory cells explaining the evaluation approach and confirming the MLflow tracking URI is pre-set to the shared RHOAI MLflow server.

**Exercise 3: Register prompt in MLflow Prompt Registry**

8. Run the notebook cell that defines the prospect agent's system prompt as a Python string.
9. Run the cell that calls `mlflow.register_prompt(name="prospect-agent-v1", template=<prompt>)`. Observe the registry response confirming version 1 is registered.
10. Open the MLflow UI, navigate to the Prompt Registry section, and verify `prospect-agent-v1 v1` appears.

**Exercise 4: Create evaluation dataset**

11. Read the six evaluation cases defined in the dataset cell — each case has an input (customer query), expected output pattern, and test category.
12. Run the cell that builds the evaluation dataset as an `mlflow.data` object. Observe the dataset summary showing six rows.

**Exercise 5: Run simple evaluation (deterministic scorers)**

13. Read the scorer definitions: a keyword-presence scorer (checks for required loan terminology in the response), a response-length scorer, and a tool-invocation scorer (verifies the agent called the expected tool).
14. Run the cell that sets up the predictor function — a thin wrapper that calls the Fed Aura Capital FastAPI endpoint with the evaluation input.
15. Run `mlflow.evaluate(data=eval_dataset, model=predictor, scorers=[...])`. Watch the progress indicator.
16. Open the MLflow UI and navigate to the evaluation run. Examine the per-case scorer results table. Identify any failing cases.

**Exercise 6: Run LLM-as-a-Judge evaluation**

17. Read the judge prompt defined in the notebook — it asks a second LLM to rate the prospect agent's responses on clarity, accuracy, and appropriate referral behavior (1–5 scale).
18. Run the cell that adds the LLM-as-a-Judge scorer to the scorer list and re-runs `mlflow.evaluate()`.
19. In the MLflow UI, compare the LLM-as-a-Judge scores against the deterministic scorer results. Identify any case where the judge disagrees with the deterministic scorer and discuss why.

**Exercise 7: Explore prompt-trace linkage**

20. In the MLflow UI, click into a per-trace evaluation result. Observe the linked MLflow trace (from Module 4's autolog) showing the actual agent execution for that evaluation case.
21. Navigate to the Prompt Registry and confirm the evaluation run is linked to `prospect-agent-v1 v1` — every evaluation result is pinned to the exact prompt version used.

## Key Takeaways

- RHOAI Jupyter workbenches provide a managed environment for running evaluation notebooks without local setup or dependency management.
- MLflow Evaluation supports both deterministic scorers (fast, reliable, rule-based) and LLM-as-a-Judge scorers (flexible, subjective quality assessment) in a single `mlflow.evaluate()` call.
- The MLflow Prompt Registry pins every evaluation result to the exact prompt version used — enabling reproducibility and future regression comparison.
- Prompt-trace linkage closes the feedback loop: evaluation scores can be drilled down to individual agent execution traces, making it possible to diagnose exactly why a case failed.
- LLM-as-a-Judge evaluation requires its own quality assurance — the judge prompt, the judge model, and the scoring rubric are all variables that affect results.

## Infrastructure Notes

- RHOAI Data Science Projects and Jupyter workbench images must be configured before the lab.
- The Jupyter workbench requires outbound GitHub connectivity to clone `rh-ai-quickstart/multi-agent-loan-origination.git`.
- The MLflow tracking URI environment variable must be pre-set in the workbench environment (or injected via a ConfigMap) so students do not need to configure it manually.
- The LLM-as-a-Judge scorer calls the MaaS LLM endpoint — the `llm-credentials` secret must be accessible from the workbench namespace or injected as an environment variable.
- This module is marked optional; it requires approximately 55 minutes and Python comfort. Instructors may choose to demo rather than have all students complete it.
