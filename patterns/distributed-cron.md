# ⏰ Distributed Cron Jobs

> Regular cron runs on one server. What happens when you have 10 servers? Each runs the job 10 times. That's a problem.

---

## The Problem

```
Single server cron: ✅ Job runs once at 2am
10-server fleet:    ❌ Job runs 10 times at 2am — duplicate emails, double charges!
```

---

## Solutions

### 1. Dedicated Scheduler Node

- One designated machine runs all cron jobs
- ✅ Simple
- ❌ Single point of failure — if this node dies, no cron jobs run

### 2. Leader Election + Cron

- Cluster elects a leader (using etcd/ZooKeeper)
- Only the leader runs cron jobs
- If leader dies → new leader elected → cron resumes
- ✅ No SPOF | ❌ Leader election complexity

### 3. Distributed Lock Before Job Execution

```python
def run_job():
    lock = redis.set("cron:daily-email", "locked", nx=True, ex=300)  # 5min TTL
    if not lock:
        return  # another node is running it
    try:
        send_daily_emails()
    finally:
        redis.delete("cron:daily-email")
```

- ✅ Simple, works with existing Redis
- ❌ Lock expiry must be > job duration, or use heartbeat extension

### 4. Dedicated Scheduling Services

- **AWS EventBridge Scheduler** — cron without servers
- **Temporal.io** — workflow orchestration with scheduling
- **Airflow** — DAG-based job scheduling
- **Sidekiq-Cron** (Ruby), **Bull** (Node.js)

---

## Exactly-Once Execution

For critical jobs (billing, emails):

1. Use idempotency keys in the job itself
2. Record execution in DB before running: `INSERT INTO job_runs (job_id, run_date) ON CONFLICT DO NOTHING` → if 0 rows inserted, skip
3. Use a distributed lock with heartbeat

---

## Observability for Cron

- Alert if a job DIDN'T run (missed heartbeat)
- Track job duration — sudden increase = something is wrong
- Log job result (success/failure, records processed)
- Tool: **Healthchecks.io** — cron monitoring via ping

---

## 🎨 Diagram

See: [`../diagrams/distributed-cron.excalidraw`](../diagrams/distributed-cron.excalidraw)
