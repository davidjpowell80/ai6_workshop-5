# Activity 2: The Happy Path (Single Execution)

**Primary KSB:** K12 — Deployment approaches for new data pipelines and automated processes

🎯 **Learning Objective:** Verify the pipeline works end-to-end by processing a single ticket, and identify the model step

## AWS Docs (Core Services)

See [AWS service docs and key quotes](../../docs/aws_service_docs.md).

## 📋 Expected Outputs

- One successful Step Functions execution
- Understanding of the three pipeline steps and their relative durations
- Identification of the Embed step as the model (slowest) step
- Answer to: "The model step is ___ because ___"

---

## 📝 Task 1 — Invoke a Single Ticket

1. Run the single-invocation script:

⌨️ **Terminal:**

```bash
./scripts/02_invoke_one.sh
```

2. The script sends one support ticket through the pipeline and prints the execution ARN.
3. Wait for the script to report the execution result.

✅ **Checkpoint:** The script reports `SUCCEEDED` and prints output JSON.

---

## 📝 Task 2 — Inspect the Execution Graph

💻 **Console:**

1. Navigate to **Step Functions** > **State machines** > `AI6-Unit5W-ScaleOrFail-state-machine` (or use the `StateMachineArn` output from Activity 1).
2. Click **Executions** and select the most recent execution. (If you've still got this page open from Activity 1, you may need to click the refresh button in the "Executions" table.)
3. View the **Graph view** — you should see three steps:
   - **Preprocess**
   - **Embed**
   - **Postprocess**
4. Note the **duration** of each step (visible in the execution details or step details when you click on the step).

💡 **Tip:** Click on each step in the graph to see its input, output, and duration in the right-hand panel.

✅ **Checkpoint:** All three steps show green (succeeded). You can see individual durations.

---

## 📝 Task 3 — Identify the Model Step

1. Compare the durations of the three steps:
   - Preprocess
   - Embed
   - Postprocess
2. Based on the evidence you see in the execution graph, decide which step is acting as the “model step” in this workflow.

**Answer the following:**

> "The model step is **Embed** because **the output contains the model name and the confidence score**."

---

## 📝 Task 4 — Examine the Output JSON

Look at the final output from the execution. It should contain fields like:

```json
{
  "route": "billing",
  "route_score": 0.87,
  "priority": "normal",
  "action": "auto-respond"
}
```

The fields are:
- `route` — the predicted support category
- `route_score` — confidence of the classification
- `priority` — business-rule priority assignment
- `action` — recommended next action

✅ **Checkpoint:** Output JSON is visible with `route`, `route_score`, `priority`, and `action` fields.

<details>
<summary><strong>Hint: what you should observe</strong></summary>

- In a typical run, **Embed** is noticeably slower than Preprocess and Postprocess.
- You should see all three steps succeed (green) and be able to click each step to view its input/output and duration.

</details>

<details>
<summary><strong>Example answer (optional)</strong></summary>

> "The model step is **Embed** because it takes the longest and is where the workflow performs the route/priority inference."

</details>

---

## 🚀 Extension

1. Navigate to **CloudWatch** > **Logs** > **Log Management** > **Log groups**.
2. Find the log group for the Embed function (it will contain `Embed` in the name).
3. Open the most recent log stream and read a structured log entry.
4. What information does the Embed function log? (e.g. input length, model load time, inference time)

---

🎓 **Complete** — proceed to [Activity 3](../activity-3/activity-3_start.md)
