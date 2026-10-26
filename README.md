# Fan-out Notifications: A System Design Lab

This project is a hands-on lab for learning system design and distributed systems by building a notification service, measuring where it struggles, and improving it one architectural step at a time.

The example is a video platform: when a creator publishes a video, the system needs to make that update available to many subscribers. The experiments begin with a deliberately simple implementation and gradually introduce techniques used to handle higher load. The goal is to understand the trade-offs through measurements, not to claim a production-ready YouTube replica.

## Learning approach

Each stage follows the same loop:

1. State a hypothesis about the system's bottleneck.
2. Run a repeatable load test and record the baseline.
3. Observe application, database, queue, and host metrics.
4. Change one part of the architecture.
5. Run the same workload again and compare the results.

This makes failures useful evidence: each bottleneck motivates the next design change.

## Planned architecture stages

| Stage | Design | What the experiment explores |
| --- | --- | --- |
| 1. Synchronous baseline | One API and PostgreSQL database. A video publish request finds 50,000 subscribers and writes their notifications before responding. | Request latency and timeouts, database connection-pool pressure, write throughput, and CPU use. |
| 2. Scale the API | Put Nginx in front of three API replicas, with the same synchronous database work. | Whether adding API capacity helps when the database remains the bottleneck, and how a failed API instance affects service. |
| 3. Add asynchronous work | Publish an event to RabbitMQ and return from the API while one worker creates notifications. | Faster request responses and the formation of a queue backlog when consumers cannot keep up. |
| 4. Add batching and consumers | Split recipients into batches (initially 1,000) and process them with several workers using bulk inserts. | Consumer throughput, database write capacity, and queue drain time. |
| 5. Split database reads and writes | Configure PostgreSQL streaming replication with a primary and a read replica. | Read/write routing and the effect of moving subscriber reads away from the primary. |
| 6. Measure replication lag | Apply write load and compare data visibility on the primary and replica. | Replication delay and eventual-consistency behavior. |
| 7. Explore hybrid fan-out | Use Redis and a push/pull strategy: push notifications for smaller channels and let subscribers fetch updates for very large channels. | The trade-offs between per-subscriber writes, read-time work, cache use, and fan-out cost. |

The workload sizes and worker counts above are starting points for experiments; they will be adjusted and documented based on the lab's measured limits.

## Technology and lab environment

- **Application:** Node.js
- **Database:** PostgreSQL
- **Messaging:** RabbitMQ
- **Cache:** Redis
- **Proxy/load balancer:** Nginx
- **Containers:** Docker Compose
- **Load testing:** k6

The target is a resource-constrained home server laptop with an Intel Core i3-2310M processor and 4 GB of RAM. Load generation runs from a separate laptop to reduce resource contention. Container limits keep the experiment within the available memory. This small setup is intentional: it makes resource bottlenecks visible and keeps the experiments reproducible on modest hardware.

## Development and deployment workflow

Application changes are developed on a separate laptop and tracked in GitHub. The initial deployment workflow is manual: connect to the home server over SSH, update the checked-out code, and run the relevant Docker Compose services. Once this process is understood and repeatable, CI/CD automation can be added as a separate learning step.

## What will be measured

Measurements will be recorded with the workload and configuration used, so comparisons remain meaningful. Depending on the stage, the lab will track:

- **k6:** request rate, p95/p99 latency, errors, timeouts, and virtual users.
- **Host and containers:** CPU, memory, swap, load average, and per-container resource use.
- **PostgreSQL:** connection usage, query/write latency, insert rate, locks, and waits.
- **RabbitMQ and workers:** queue depth, ready and unacknowledged messages, consume rate, retries, and drain time.
- **Replication:** replica lag and the delay before new data becomes visible on the replica.
- **Redis and fan-out:** cache hit/miss rate, memory use, and the number of per-subscriber writes avoided.

Screenshots, benchmark summaries, architecture diagrams, and lessons learned will be added as the experiments are completed. Results will be presented as observations from this specific lab, not as universal performance claims.

## Project status

The lab plan and deployment access are being established. The application stages and benchmark results are planned work and will be documented here as they are implemented and measured.
