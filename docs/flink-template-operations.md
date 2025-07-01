# Flink Template Operations

## Autoscaling Model

This template assumes two separate control loops:

- `Flink job autoscaler` adjusts parallelism
- Kubernetes and node autoscaling provide compute capacity so new `TaskManager` pods can land

Do not treat HPA as the primary control loop for steady-state TaskManager scaling when the Flink job autoscaler is enabled. That creates conflicting feedback.

## Autoscaler Guardrails

Recommended defaults:

- bounded scale-up and scale-down factors
- stabilization windows to avoid oscillation
- target utilization tuned by workload class
- minimum and maximum parallelism bounds per job

A good starting policy is:

- target utilization around `0.7`
- scale-up factor cap around `2.0`
- scale-down factor floor around `0.5`
- metrics window of several minutes, not seconds

These defaults are encoded in the sample values files and can be tightened for cost-sensitive low-throughput topics.

## Scale To Zero

Scale-to-zero is not a default for stateful low-latency jobs in this template. It increases cold-start latency, complicates state warmup, and can break operational expectations around checkpoint cadence and lag recovery. Treat it as a separate architecture, not a routine toggle.

## Cost Controls

Focus on these cost levers first:

- right-size TaskManager slots against CPU and memory instead of maximizing slot density blindly
- avoid over-parallelizing low-throughput topics
- retain checkpoints, savepoints, and Paimon snapshots intentionally instead of indefinitely
- tune Kafka retention and compaction by topic criticality
- use spot or preemptible capacity only for tolerant or replayable workloads
- isolate heavy batch backfills from always-on low-latency pipelines

## Local Smoke Test

The local smoke test is designed to exercise the workflow rather than prove benchmark-grade throughput:

1. verify the operator, Kafka, MinIO, and `FlinkDeployment` resources exist
2. submit the sample Kafka producer job
3. confirm TaskManager and JobManager pods stay healthy
4. inspect checkpoint-related configuration on the deployment
5. optionally restart a TaskManager pod to confirm recovery behavior

The helper script for this flow lives at [scripts/flink-local-smoke-test.sh](/Users/harpreetsingh/Documents/flink-streaming-infra-template/scripts/flink-local-smoke-test.sh).

## Rendering And Validation

At minimum, validate:

- local values render without production-only placeholders
- prod-example values render without demo dependencies
- checkpoint/savepoint paths point to durable object storage in production mode
- autoscaler settings remain bounded
- ingress exposure is disabled unless intentionally enabled

## Failure Playbooks

Every production deployment should have runbooks for:

- Kafka broker outage or elevated lag
- object store unavailability or increased latency
- sustained backpressure and checkpoint timeout growth
- autoscaler churn or repeated scale reversals
- failed sink commits or warehouse metadata contention
