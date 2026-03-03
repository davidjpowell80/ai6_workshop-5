# Activity 6: Controlled Failure — Bad Input (RCA Incident 1)

**Primary KSB:** S26 — Independent, impartial decision-making; B4 — Integrity in technical decisions

🎯 **Learning Objective:** Trigger a data validation failure, classify it using the RCA Tree, and distinguish it from a scaling issue

## AWS Docs (Core Services)

See [AWS service docs and key quotes](../../docs/aws_service_docs.md).

---

## 📋 Expected Outputs

- A FAILED execution visible in Step Functions
- A written mini incident report with evidence, classification, and first safe action

---

## 📝 Task 1 — Trigger the Bad Input

Run the script that sends an oversized payload to the pipeline:

⌨️ **Terminal:**

```bash
./scripts/05_trigger_bad_input.sh
```

💡 **Tip:** This script sends a single ticket with a payload that exceeds the Lambda input size limit. It is designed to fail.

---

## 📝 Task 2 — Examine the Failed Execution

1. Open **Step Functions** > **Executions**.
2. Find the execution with status **FAILED** (it should be the most recent one).
3. Click into the execution to see the execution graph.

💻 **Console:** Step Functions > State machines > AI6-Unit5W-ScaleOrFail-state-machine > Executions

✅ **Checkpoint:** You can see a FAILED execution with a red indicator in the execution list.

---

## 📝 Task 3 — Read the Error Message

1. In the execution detail view, click on the failed state (it will be highlighted in red).
2. Read the **Reason** for failure in the Input/Output tab.

What does the error message say? Write it down.

---

## 📝 Task 4 — Classify Using the RCA Tree

Use the Scaling RCA Tree to classify this incident. Here is the tree for reference:

### Scaling RCA Tree (4 leaves)

When your pipeline is slow or failing under load, classify it:

**1) THROTTLED (hard limit)** — Throttles > 0, TooManyRequests errors. Cause: concurrency too low.

**2) EXHAUSTED (resource)** — High p95 at low traffic, OutOfMemory. Cause: model too heavy.

**3) TIMED OUT (dependency)** — Duration grows, retries. Cause: external API slow.

**4) BAD INPUT (data)** — Immediate failures, validation errors. Cause: no validation gate.

You can also view an image of this tree [here](../../diagrams/RCA_tree_example.jpg).

---

## 📝 Task 5 — Rule Out the Other Leaves

Which of the 4 leaves matches this incident? For each of the other 3, explain why you can rule it out:

| RCA Leaf | Matches? | Why / Why not? |
|---|---|---|
| THROTTLED | No | No requests were made  |
| EXHAUSTED | No | Failed at the preprocess step, no concurrent executions recorded |
| TIMED OUT | No | Failed fast with a pre processing check error message |
| BAD INPUT | Yes | Error message indicates the payload is too large - "errorMessage": "PayloadTooLarge: text length 5100 > 5000",  |

---

## 📝 Task 6 — CloudWatch Logs Insights: Find the Error

Open **CloudWatch** > **Logs Insights** and select the preprocess log group:
- `/aws/lambda/AI6-Unit5W-ScaleOrFail-preprocess`

Run **Query 4** to find the error:

```sql
fields @timestamp, @message
| filter @message like /PayloadTooLarge/
| sort @timestamp desc
```

✅ **Checkpoint:** The query returns log entries containing the PayloadTooLarge error.

---

## 📝 Task 7 — Write a Mini Incident Report

Complete the following incident report template:

- **What happened:** _______________________________________________
- **Evidence:** _______________________________________________
- **Classification:** _______________________________________________
- **First safe action:** _______________________________________________

⚠️ **Warning:** Be precise with your evidence. Point to the specific error message and the specific metric or log entry, not just "it failed."

<details>
<summary><strong>Hint: what you should observe</strong></summary>

- This failure should happen **immediately** on a single request (not under burst load).
- The error evidence should point to a **data/validation** problem (for example, payload size), not a capacity issue.
- Classification should map to the **BAD INPUT (data)** leaf of the RCA Tree.

</details>

<details>
<summary><strong>Example answer (optional)</strong></summary>

- **What happened:** "A single execution failed immediately when given an oversized payload."
- **Evidence:** "Step Functions shows FAILED with an error indicating the payload was too large; Logs Insights returns `PayloadTooLarge`."
- **Classification:** "BAD INPUT (data)."
- **First safe action:** "Add a validation gate early (preprocess) to reject oversized inputs with a clear error."

</details>

---

🚀 **Extension:** What validation rule would you add to the Preprocess function to catch this earlier? Think about: at what point in the pipeline should you check payload size, and what should the error response look like?

---

🎓 **Complete** — Proceed to [Activity 7: Controlled Failure — Throttling (RCA Incident 2)](../activity-7/activity-7_start.md)
