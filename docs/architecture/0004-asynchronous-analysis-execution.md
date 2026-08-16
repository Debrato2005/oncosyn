# ADR 0004: Defer asynchronous analysis execution until measurements require it

## Status

Deferred. The initial product uses a synchronous application service. No API, Redis instance, worker, or job implementation exists yet.

## Problem

Candidate generation and portfolio optimization can run longer than a web request is allowed to stay open. Retrying a timed-out request must not produce duplicate analyses or lose the evidence/provenance needed for an escape certificate.

## Constraints

- The API must remain responsive and thin.
- An analysis run must be observable, authorized, versioned, and idempotent.
- Job state is durable in PostgreSQL; a queue is not the source of record.
- Jobs may call external predictors or evidence sources that can fail transiently.
- Job payloads must not place raw sensitive data in logs or unbounded broker messages.

## Selected design

Do not add a worker or queue in the initial build. Execute the candidate-to-certificate workflow synchronously through an application service and measure end-to-end duration, failure modes, and concurrent usage. Preserve the application-service boundary so a future worker can receive a stable analysis-run identifier rather than duplicating domain logic.

Revisit this ADR only when measured request duration approaches platform limits, retries are required, or concurrent analyses impair responsiveness. At that point select a queue technology and document its durability, retry, and operational behavior; PostgreSQL remains the system of record.

## Alternatives rejected

- **Add a queue before evidence of need:** rejected because it adds a broker, worker lifecycle, retry policy, monitoring, and failure modes without improving the initial core workflow.
- **Use Redis as the system of record when a queue is added:** rejected because provenance and auditability require transactional durable storage.
- **Place full genomic payloads in future queue messages:** rejected because it enlarges exposure and makes retries/versioning opaque.
- **Use synchronous background threads in the API process if asynchronous work becomes necessary:** rejected because process restarts and horizontal replicas make work unreliable.

## Intended data flow

```text
validated analysis request -> synchronous application service
  -> candidate generation + optimization
  -> PostgreSQL certificate/result -> API/UI response
```

## Error and security behavior

- Return typed failures for invalid input, unavailable predictor/evidence, and infeasible optimization; do not show a failed/incomplete result as a completed certificate.
- Restrict application and PostgreSQL credentials to least privilege; do not log payloads or secrets.
- If asynchronous execution is adopted later, add bounded retries, idempotency, terminal-failure states, and least-privilege worker/broker credentials before enabling it.

## Testing strategy

- Initial integration tests cover synchronous service execution, predictor/evidence failure, and result persistence/retrieval.
- If asynchronous execution is adopted, add integration tests for enqueue, execution, idempotency, retry, terminal failure, process interruption, and result retrieval, plus queue-payload safety tests.
- Establish queue-depth, oldest-job-age, retry-rate, and failure-rate alerts only when a queue exists.

## Deferred work

Queue selection, workflow orchestration, worker pools, priority queues, scheduled re-analysis, real-time notifications, and horizontal scale-out are deferred until measured workload requires them.
