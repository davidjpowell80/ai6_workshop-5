# Activity 4: Scale Up & Compare

**Primary KSB:** S19 — Ensure the model capacity is scaled in proportion to the operating requirements; S22 — Identify architecture to solve computational problems

🎯 **Learning Objective:** Apply horizontal scaling by increasing parallel capacity and measure the improvement

## AWS Docs (Core Services)

See [AWS service docs and key quotes](../../docs/aws_service_docs.md).

## 📋 Expected Outputs

- A faster batch run at higher concurrency
- Side-by-side comparison of MAX_CONCURRENCY=2 vs MAX_CONCURRENCY=10
- One-sentence explanation of what changed and why
- Thinking about the next bottleneck beyond concurrency

---

## 📝 Task 1 — Run the Same Burst at Higher Concurrency

Run the burst-load script again with the same 500 tickets, but now allow **10 parallel executions**:

⌨️ **Terminal:**

```bash
N=500 MAX_CONCURRENCY=10 ./scripts/03_burst_load.sh
```

Wait for completion and record the total batch duration.

✅ **Checkpoint:** You can see a measurable change in total batch duration compared to Activity 3.

---

## 📝 Task 2 — Record the New Duration

| Metric              | Your Value |
|---------------------|------------|
| Total batch duration |     36 seconds      |
| Tickets processed    | 500       |
| Max concurrency      | 10        |

---

## 📝 Task 3 — Compare Side-by-Side

Fill in the comparison table using your results from Activity 3 and this activity:

| Metric                | MAX_CONCURRENCY=2 | MAX_CONCURRENCY=10 |
|-----------------------|--------------------|---------------------|
| Total batch duration  |       149             |         36            |
| Tickets processed     | 500                | 500                 |
| Throughput (tickets/s)|               3.35     |           13.8          |

💡 **Tip:** Calculate throughput as `tickets / duration`. For example: 500 tickets / 126 s = ~4.0 tickets/s vs 500 tickets / 41 s = ~12.2 tickets/s.

---

## 📝 Task 4 — Observe the Dashboard Difference

💻 **Console:**

1. Open the **CloudWatch Dashboard**.
2. Compare the metrics from the two burst runs:
   - **ConcurrentExecutions** — does it now reach a higher value?
   - **Embed Duration p95** — is the per-invocation time similar or different?
   - **Execution count** — you should see a second cluster of executions.

✅ **Checkpoint:** You can see how changing concurrency affects throughput and overall batch duration.

---

## 📝 Task 5 — Write Your Explanation

In one sentence, explain what changed and why:

> "We increased __The number of tickets processed per per second ________ from 3.35 to 13.8, which meant we are servicing our customers quicker."

💡 **Tip:** Focus on the difference between making each individual execution faster (vertical scaling) vs running more executions at the same time (horizontal scaling). Which one did we do?

We horizontally scalled (Scaled out) . increased costs for a short period of time, managed by AWS managed service

---

## 📝 Task 6 — Think About the Next Bottleneck

Consider this question (discuss with your coach or group):

> "If we went to MAX_CONCURRENCY=500 (one slot per ticket), what would happen? What would be the next bottleneck?"

Think about:
- Lambda cold starts when 500 functions spin up simultaneously
- AWS account-level Lambda concurrency limits
- Memory and CPU contention
- Cost implications

<details>
<summary><strong>Hint: what you should observe</strong></summary>

- The batch should complete **significantly faster** than the `MAX_CONCURRENCY=2` run.
- **ConcurrentExecutions** should rise toward your configured `MAX_CONCURRENCY`.
- The per-item Embed duration usually stays in a similar range; the improvement comes from **more parallelism**, not faster single calls.

</details>

<details>
<summary><strong>Example answer (optional)</strong></summary>

> "We increased **MaxConcurrency** from 2 to 10, which meant more tickets were processed in parallel, reducing total batch duration even though each Embed call took roughly the same time."

</details>

---

## 🚀 Extension

Try an intermediate concurrency value to see how throughput scales:

⌨️ **Terminal:**

```bash
N=500 MAX_CONCURRENCY=5 ./scripts/03_burst_load.sh
```

Record the results:

| MAX_CONCURRENCY | Duration | Throughput (tickets/s) |
|-----------------|----------|------------------------|
| 2               |          |                        |
| 5               |          |                        |
| 10              |          |                        |

At what point do returns start to diminish?

⚠️ **Sandbox limit:** The Pluralsight AWS sandbox restricts account-level Lambda concurrency to **10**. Do not exceed `MAX_CONCURRENCY=10` — values above this will cause executions to fail with a `TooManyRequestsException` (429). In a standard AWS account the default limit is 1,000 — see [Lambda quotas](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html#compute-and-storage). You can check your current limit at any time with `aws lambda get-account-settings`.

---

🎓 **Complete** — proceed to [Activity 5](../activity-5/activity-5_start.md)
