# Activity 5: Understand Orchestration (Reading the Logs)

**Primary KSB:** K12 — Deployment approaches; S10 — Monitoring in live environment

🎯 **Learning Objective:** Understand how Step Functions orchestration provides visibility and control over parallel processing

## AWS Docs (Core Services)

See [AWS service docs and key quotes](../../docs/aws_service_docs.md).

---

## 📋 Expected Outputs

- Three CloudWatch Logs Insights queries executed successfully
- Understanding of which pipeline step takes the most time
- Written answer: "How does orchestration help you see where time is spent?"

---

## 📝 Task 1 — Open the State Machine

1. Open **Step Functions** > **State machines** in the AWS Console.
2. Click **AI6-Unit5W-ScaleOrFail-state-machine** to open the state machine (or use the `StateMachineArn` output from Activity 1).

💻 **Console:** Step Functions > State machines > AI6-Unit5W-ScaleOrFail-state-machine

---

## 📝 Task 2 — Examine the State Machine Definition

1. View the state machine definition by selecting an execution and inspecting the Graph view.
2. Identify the **Map state** — this is the parallel processing construct.
3. Find the `max_concurrency` parameter on the Map state. Do this by selecting the Map state (ProcessTickets) in the graph view, then clicking "View Map state overview".

✅ **Checkpoint:** You can see the Map state and its `max_concurrency` value in the definition.

💡 **Tip:** The Map state iterates over an array of inputs and runs a sub-workflow for each item. `max_concurrency` controls how many run in parallel.

---

## 📝 Task 3 — Compare Execution Histories

1. Go to the **Executions** tab.
2. Open the execution from Activity 3 (low concurrency run) in one browser tab.
3. Open the execution from Activity 4 (high concurrency run) in another browser tab.
4. Compare side-by-side:
   - How long did each batch take?
   - How many items processed in parallel?
   - Where did time stack up?

✅ **Checkpoint:** You can see the difference in execution duration and parallelism between the two runs.

---

## 📝 Task 4 — Open CloudWatch Logs Insights

1. Open **CloudWatch** > **Logs Insights** in the AWS Console.

💻 **Console:** CloudWatch > Logs > Logs Insights

2. Click in the box that says "Select up to 50 log groups" and select the following log groups (tick all three):
   - `/aws/lambda/AI6-Unit5W-ScaleOrFail-preprocess`
   - `/aws/lambda/AI6-Unit5W-ScaleOrFail-embed`
   - `/aws/lambda/AI6-Unit5W-ScaleOrFail-postprocess`

⚠️ **Warning:** Make sure all three log groups are selected before running queries, or you will only see partial results.

---

## 📝 Task 5 — Query 1: Recent Log Lines

You can filter logs by time by using the time filter at the top of the screen. By default it's set to 1h, which will be fine for our purposes, but you may wish to constrain it to get a finer view of the situation.

The default query works fine, so click **Run query**. (You may wish to limit the number of log messages, but it's not necessary.)

Once you've run the query, the raw logs appear at the bottom of the screen. You can view patterns (shared text structures that recur in your logs) by click the "Patterns" tab under the query box.

✅ **Checkpoint:** You see log entries from preprocess, embed, and postprocess functions. If you're missing any of these, you might need to extend the time filter (longer time) or increase the query limit.

---

## 📝 Task 6 — Query 2: Extract Structured Fields (Try It First)

Write a Logs Insights query that extracts:
- `step`
- `ticket_id`
- `duration_ms`

Then display those fields in a table.

💡 **Read the docs!** You will need to consult authoritative, trustworthy sources online to complete this task, like those listed [here](../../docs/aws_service_docs.md). Some limited hints are given [here](../../observability/cloudwatch_logs_insights_queries.md), but you will need to look elsewhere, too.

✅ **Checkpoint:** You can see columns for step, ticket_id, and duration_ms in the results.

---

## 📝 Task 7 — Query 3: Aggregate Duration by Step (Try It First)

Write a Logs Insights query that calculates:
- average duration by `step`
- max duration by `step`
- count by `step`

✅ **Checkpoint:** You can see the slowest step when you sort by average duration.

---

## 📝 Task 8 — Answer the Question

Write your answer to this question:

> **"How does orchestration help you see where time is spent?"**

We are able to analyse the logs to calculate time spent in each stage of the process

Think about:
- How the Map state separates each ticket's processing into distinct steps
- How MaxConcurrency controls throughput
- How Logs Insights lets you aggregate and compare step durations

<details>
<summary><strong>Example answer (optional)</strong></summary>

> "Orchestration breaks the work into named steps and makes parallelism explicit (Map + MaxConcurrency). That lets you measure where time is spent per step (logs/metrics) and change throughput by tuning concurrency, without guessing."

</details>

---

🚀 **Extension:** Write your own Logs Insights query to find the ticket with the longest Embed duration. What ticket_id had the slowest embed step?

---

🎓 **Complete** — Proceed to [Activity 6: Controlled Failure — Bad Input](../activity-6/activity-6_start.md)
