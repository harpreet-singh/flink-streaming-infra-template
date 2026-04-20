# Flink Template Production Patterns

## Namespace Model

Use separate namespaces for distinct operational concerns:

- `flink-operator`: operator deployment and admission webhooks
- `streaming-platform`: shared platform services such as catalogs, schema helpers, or shared sidecars if needed
- workload namespaces such as `streaming-orders`, `streaming-risk`, or `streaming-ml-features`

This keeps operator blast radius, RBAC, and workload ownership clean.

## Compute Isolation

Prefer dedicated node pools or taints for stateful stream compute:

- isolate `TaskManagers` from bursty batch nodes
- reserve capacity for low-latency or SLA-sensitive jobs
- keep node autoscaler decisions aligned with streaming demand rather than generic platform noise

For multi-tenant clusters, use labels and taints so critical streaming jobs do not share preemptible pools with opportunistic batch workloads.

## Externalized State

Treat all durable state as external:

- checkpoints in object storage
- savepoints in object storage
- high-availability metadata in object storage
- Paimon warehouse data and metadata in durable object storage

Never rely on pod-local filesystems for restart-critical state outside ephemeral dev experiments.

## Upgrades

The default upgrade path should be savepoint-driven:

1. trigger a savepoint
2. confirm the savepoint path is durable and readable
3. roll out the new image or job artifact
4. restore from the saved path when compatibility requires it

For risky schema or state changes, prefer blue/green or canary rollout patterns over a single in-place replacement. Run the new job from a mirrored or replayable topic when business risk is high.

## High Availability

Recommendations for production deployments:

- enable JobManager HA metadata in object storage
- run workloads across multi-AZ node pools when the cluster spans zones
- add pod anti-affinity for JobManager and TaskManagers where cluster size permits
- validate network throughput and object-store latency assumptions before broad rollout

## Secrets And Configuration

Keep sensitive configuration separate from chart values:

- Kafka credentials and TLS material in secrets or external secret managers
- object storage access keys in secrets or workload identity bindings
- catalog credentials and endpoints separated from non-secret Flink config

Use chart values for secret references, not raw credentials.

## Failure Domains

Document and test failure assumptions explicitly:

- broker outage and partition unavailability
- object-store throttling or transient failures
- zone loss or node-pool depletion
- prolonged backpressure and lag growth

Production platforms should define whether jobs fail fast, back off, or continue degraded under each failure type.

## Observability

Operators should baseline dashboards and alerts for:

- consumer lag by topic and partition
- backpressure ratios
- checkpoint duration, alignment time, and failure counts
- restart counts and exception-rate changes
- autoscaler recommendations and scale decisions
- sink commit latency and failed commit counts
- object-store error rates and elevated latency

The application chart leaves room for metrics reporters and annotations, but production monitoring stacks should be layered in separately rather than hard-coded to one vendor.
