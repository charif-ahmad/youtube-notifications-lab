# Stage 1: Synchronous Fan-out Baseline

**Status:** Planned; implementation and measurements are pending.

## Question

What becomes the first bottleneck when one API request creates notifications synchronously for a creator's subscribers?

## Hypothesis

Doing subscriber lookup and many notification inserts before responding will increase request latency and may saturate the API, database connection pool, or PostgreSQL write capacity. The first measured bottleneck is not known yet.

## Planned workload

- One API instance and one PostgreSQL instance.
- Seed a dataset with 50,000 subscribers for the target creator.
- Publish an event and synchronously create subscriber notifications before returning the response.
- Define and record the k6 arrival rate, virtual users, duration, and thresholds before each baseline run.

## Capture for each run

- Git commit, app/runtime versions, database configuration, and container limits.
- k6 request rate, p95/p99 latency, error and timeout rates, and virtual users.
- Host/container CPU and memory, swap, and load average.
- PostgreSQL connection usage, query/insert latency, insert rate, locks, and waits.
- Screenshots or exported outputs, timestamped and linked from the run record.

## Findings

Pending implementation and measurement. Do not report a bottleneck until the run evidence supports it.
