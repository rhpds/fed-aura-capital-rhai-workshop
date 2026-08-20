# Module 06 — From Development to Production (Optional)

## Brief Overview

This optional advanced module automates the evaluation workflow from Module 5 into a Red Hat OpenShift AI Data Science Pipeline (Kubeflow-based). Participants import and run a four-step evaluation pipeline via the RHOAI pipeline UI, verify that automated pipeline results match the notebook results from Module 5, and then detect a prompt regression: a deliberate change to the prospect agent prompt (changing "mandatory tool use" to "optional") causes measurable accuracy drops in the evaluation pipeline. This demonstrates the quality gate function — catching prompt degradation before it reaches production.

## Audience and Time

- **Target personas:** AI Developers/Engineers (primarily); MLOps Engineers
- **Prerequisites for this module:** Completion of Module 5; familiarity with the evaluation dataset and scoring approach from Module 5
- **Estimated duration:** 55 minutes

## Learning Objectives

- Deploy an automated evaluation pipeline using Red Hat OpenShift AI Data Science Pipelines and verify it produces results consistent with the notebook evaluation from Module 5
- Analyze the impact of a prompt change by comparing evaluation scores between prompt versions using MLflow Prompt Registry and per-trace results
- Demonstrate the quality gate pattern: automated evaluation pipelines that catch regressions before they reach production

## Lab Structure

| Section | Title | Duration |
|---------|-------|----------|
| 1 | Import evaluation pipeline | 10 min |
| 2 | Run evaluation pipeline | 10 min |
| 3 | View automated evaluation results in MLflow | 10 min |
| 4 | Catch bad prompt before production | 25 min |

## Detailed Steps

**Exercise 1: Import evaluation pipeline**

1. Open the RHOAI dashboard and navigate to Data Science Pipelines for your project.
2. Click "Import pipeline" and paste the raw GitHub URL for the evaluation pipeline YAML:
   `https://raw.githubusercontent.com/rh-ai-quickstart/multi-agent-loan-origination/main/pipelines/evaluate_agent_pipeline.yaml`
3. Confirm the pipeline imports successfully and appears in the pipeline list with four steps visible: Data Prep, Evaluate Agent, Score Results, Publish to MLflow.
4. Click into the pipeline graph and read the step descriptions to understand the four-step structure.

**Exercise 2: Run evaluation pipeline**

5. Click "Create run" for the imported pipeline.
6. Set the pipeline parameters: MLflow tracking URI (pre-filled from environment), experiment name (`fed-aura-capital-evals`), prompt name (`prospect-agent-v1`), prompt version (`1`).
7. Start the run and monitor the pipeline graph as each step completes. Note approximate step durations.
8. Wait for the pipeline run to reach Succeeded status.

**Exercise 3: View automated evaluation results in MLflow**

9. Open the MLflow UI and navigate to the `fed-aura-capital-evals` experiment.
10. Find the run created by the pipeline (it will have a `pipeline_run_id` tag).
11. Compare the scorer results to the notebook run from Module 5 — confirm scores are consistent across all six evaluation cases.
12. Note the pipeline run's metadata: it is pinned to `prospect-agent-v1 v1`, making it reproducible.

**Exercise 4: Catch bad prompt before production**

13. Open the `evaluate_agent.ipynb` notebook in JupyterLab (from Module 5).
14. Navigate to the prompt definition cell. Find the phrase "You MUST use the loan-lookup tool for every request." Change it to "You may use the loan-lookup tool if needed."
15. Run the cell that registers this modified prompt in the MLflow Prompt Registry: `mlflow.register_prompt(name="prospect-agent-v1", template=<modified_prompt>)`. This creates `prospect-agent-v1 v2`.
16. In the MLflow Prompt Registry, open `prospect-agent-v1` and use the version comparison view to diff v1 and v2 — confirm the single-word change is visible.
17. Return to the RHOAI pipeline UI and create a new run of the evaluation pipeline, this time setting prompt version to `2`.
18. Wait for the pipeline run to complete.
19. In MLflow, open the v2 pipeline run results and compare against the v1 run. Focus on the tool-invocation scorer — it should show a significant drop in cases where the agent was expected to call the loan-lookup tool.
20. Compare the LLM-as-a-Judge scores between v1 and v2 for the same cases — observe whether the judge also detects the regression.
21. Read the verdict section in the lab guide: the "mandatory to optional" change produces a measurable accuracy regression detected by the automated pipeline before any production deployment.
22. Discuss the quality gate pattern: evaluation pipelines can be triggered automatically (CI/CD event, prompt registry webhook, scheduled) to catch regressions before they are promoted.

## Key Takeaways

- RHOAI Data Science Pipelines (Kubeflow-based) turn notebook-level evaluation workflows into repeatable, schedulable, auditable pipeline runs without manual intervention.
- Pipeline evaluation results pinned to specific prompt versions in the MLflow Prompt Registry create a full audit trail: every production prompt change can be traced to an evaluation run.
- A single-word change ("mandatory" to "optional") in a system prompt can cause measurable, statistically significant accuracy drops — demonstrating that LLMs are sensitive to prompt phrasing in ways that unit tests cannot detect.
- The quality gate pattern (prompt change → evaluation pipeline trigger → score comparison → block or promote) is the production-grade equivalent of a CI/CD pipeline for code, applied to AI prompts.
- Combining automated pipelines (Module 6) with interactive analysis (Module 5) gives AI teams both the speed of automation and the depth of notebook-based investigation.

## Infrastructure Notes

- RHOAI Data Science Pipelines must be enabled in the RHOAI operator configuration before the lab.
- The pipeline YAML import requires outbound connectivity to raw.githubusercontent.com from the RHOAI pipeline controller.
- Pipeline runs call the Fed Aura Capital FastAPI service and the MaaS LLM endpoint (for LLM-as-a-Judge scoring) — both must be reachable from the pipeline worker pods.
- The MLflow tracking URI must be passed as a pipeline parameter or injected via a PipelineSpec environment variable — confirm the RHOAI MLflow server is accessible from pipeline worker pods.
- This module requires approximately 55 minutes and assumes Module 5 completion; it should not be attempted without Module 5's prompt and evaluation dataset context.
