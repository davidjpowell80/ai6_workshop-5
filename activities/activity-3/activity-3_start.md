# Activity 3: Hit the Wall (Burst Load at Low Concurrency)

**Primary KSB:** S19 — Ensure the model capacity is scaled in proportion to the operating requirements

🎯 **Learning Objective:** Create a burst load that demonstrates the throughput bottleneck and collect evidence

## AWS Docs (Core Services)

See [AWS service docs and key quotes](../../docs/aws_service_docs.md).

## 📋 Expected Outputs

- Batch execution completed with measurable duration
- CloudWatch Dashboard evidence of the concurrency wall
- A completed Scaling Map identifying the bottleneck step
- Answer to: "Which step is the bottleneck? What evidence proves it?"

---

## 📝 Task 1 — Send a Burst of 500 Tickets at Low Concurrency

📘 **Note:** In this workshop, the scaling knob you are tuning is Step Functions Map `max_concurrency` (passed as `MAX_CONCURRENCY` in the script), rather than Lambda reserved concurrency.

Run the burst-load script with 500 tickets and a maximum concurrency of 2:

⌨️ **Terminal:**

```bash
N=500 MAX_CONCURRENCY=2 ./scripts/03_burst_load.sh
```

This sends 500 tickets through the pipeline, but only allows **2 executions to run in parallel** at any time.

✅ **Checkpoint:** The script completes and prints the total batch duration.

---

## 📝 Task 2 — Record the Batch Duration

When the script finishes, it prints a summary. Record the total duration:

| Metric              | Your Value |
|---------------------|------------|
| Total batch duration | 149 Seconds          |
| Tickets processed    | 500        |
| Max concurrency      | 2         |

---

## 📝 Task 3 — Observe the Dashboard

💻 **Console:**

1. Navigate to **CloudWatch** > **Dashboards** > open your dashboard. (If you've left this open from a previous activity, you may need to click the refresh button in the top-right.)
2. Look at the following widgets:
   - **Embed Duration p95** — how long is each Embed invocation taking?
   - **ConcurrentExecutions** — does it plateau at 2?
3. Notice the pattern: ConcurrentExecutions is flat at 2 — the chart shows the ceiling directly. From that you can *infer* that any additional tickets must wait their turn; the queuing itself is not visible, but it is the logical consequence of the cap.

💡 **Tip:** The combined widget uses **two y-axes**: duration (ms) on the left, and ConcurrentExecutions on the right. The ConcurrentExecutions line reads against the right axis — so a flat line that appears mid-chart is actually sitting at 2, not at whatever the left axis says at that height. If you are unsure which line is which, hover over any line to reveal a tooltip identifying it.

📘 **Why so few data points?** The widget is configured with a **60-second period** — each point on the chart aggregates all invocations within that minute. Duration shows the p95 across those invocations; ConcurrentExecutions shows the maximum reached (hence "(max)" in the widget title). 500 tickets running over ~70 seconds will produce just 2–3 data points, which is expected. You can see exactly how this is defined in [`infra/ai6_u5w_scale_or_fail.yaml`](../../infra/ai6_u5w_scale_or_fail.yaml) — search for `"All steps: Duration p95"`.

💡 **Tip:** The dashboard may take **1–2 minutes** to update after the burst completes. Refresh the page if metrics appear stale.

📘 **Step duration vs Duration p95:** In the previous activity, you read the duration of individual iterations of steps in the state machine graph. These durations will differ from the logged p95 durations in the dashboard. The origins of these two values are different; the latter comes directly from calculations in the lamda, while the former is AWS-determined. It's *not* the case that one is correct and the other is wrong, and it's also *not* the case that one is more useful than the other; they serve slightly different purposes.

✅ **Checkpoint:** The dashboard shows concurrency and duration signals you can use as evidence.

---

## 📝 Task 4 — Complete the Scaling Map

Work through the Scaling Map exercise below. Writing your answers in your own words — even briefly — forces you to articulate what you actually understand, which is different from recognising a correct answer when you see one.

📘 **Note on the scaling knob:** In this workshop, the effective scaling knob for all three steps is the **Step Functions Map `max_concurrency`** setting — because the three steps run sequentially within each Map iteration, this single setting governs how many complete pipeline runs happen in parallel. Each Lambda *could* be scaled independently in AWS, but that's not the knob being turned here. Write it in your own words anyway — making it concrete is the point.

💡 For the failure question, you are speculating — use the hint to reason from what you know so far. You will see some of these failure modes for real in later activities.

#### Preprocess (pre-model)
- **What does this step do?**
  _Your answer:_

- **What is the scaling knob?**
  _Your answer:_

- **Speculate: what might failure under load look like?**
  _Your answer:_

  <details>
  <summary>Hint</summary>
  This step validates and cleans input. Think about what happens if a ticket arrives with unexpected or oversized data.
  </details>

#### Embed / Model (inference)
- **What does this step do?**
  _Your answer:_

- **What is the scaling knob?**
  _Your answer:_

- **Speculate: what might failure under load look like?**
  _Your answer:_

  <details>
  <summary>Hint</summary>
  This step runs an ML model — it is compute-intensive. Think about what happens to duration as more requests pile up, and what Lambda does when it cannot keep up with demand.
  </details>

#### Postprocess (post-model)
- **What does this step do?**
  _Your answer:_

- **What is the scaling knob?**
  _Your answer:_

- **Speculate: what might failure under load look like?**
  _Your answer:_

  <details>
  <summary>Hint</summary>
  This step applies business rules and may call external services. Think about what happens if those dependencies are slow or unavailable under load.
  </details>

### Decide: which step do you scale FIRST?

Pick the step that is both:
1. **Slowest** (duration p95)
2. And/or **rejecting work** (throttles/errors)

> "I would scale **___** first because **___**."

---

## 📝 Task 5 — Answer the Bottleneck Question

Using your dashboard evidence and scaling map, answer:

> "Which step is the bottleneck? What evidence proves it?"

Write 2–3 sentences referencing specific metrics from the dashboard.

<details>
<summary><strong>Hint: what you should observe</strong></summary>

- With `MAX_CONCURRENCY=2`, the batch duration will be noticeably longer than a single execution — only 2 iterations can run at once, so the rest must wait.
- On the dashboard, **ConcurrentExecutions** should flatten at (or near) 2. This is the evidence; the waiting is what you infer from it.

</details>

<details>
<summary><strong>Example answer (optional)</strong></summary>

- "The bottleneck is **Embed**, because it has the highest Duration p95 and ConcurrentExecutions plateaus at 2 — the ceiling is visible in the chart, from which we can infer that additional tickets must wait their turn."
- "I would scale **Embed** first because it dominates per-item latency and `max_concurrency` on the Map state controls how many pipeline iterations run in parallel."

</details>

---

🎓 **Complete** — proceed to [Activity 4](../activity-4/activity-4_start.md)
