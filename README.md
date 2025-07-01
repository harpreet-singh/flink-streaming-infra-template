# Flink Streaming Infrastructure Template

`flink-streaming-infra-template` is a local-demo-first but production-oriented starter for running Flink-based streaming workloads on Kubernetes.

It centers on:

- `Flink Kubernetes Operator` for operator-managed `FlinkDeployment` resources
- `Kafka` for event ingestion
- `Paimon` for analytical and stateful table outputs
- object storage for checkpoints, savepoints, high-availability metadata, and the warehouse

## Repo Layout

- [docs/flink-template-overview.md](/Users/harpreetsingh/Documents/flink-streaming-infra-template/docs/flink-template-overview.md): architecture, deployment flow, and config contract
- [docs/flink-template-production-patterns.md](/Users/harpreetsingh/Documents/flink-streaming-infra-template/docs/flink-template-production-patterns.md): production deployment guidance
- [docs/flink-template-operations.md](/Users/harpreetsingh/Documents/flink-streaming-infra-template/docs/flink-template-operations.md): autoscaling, smoke testing, and cost controls
- [templates/flink](/Users/harpreetsingh/Documents/flink-streaming-infra-template/templates/flink): Helm charts, values overlays, and sample assets
- [scripts](/Users/harpreetsingh/Documents/flink-streaming-infra-template/scripts): local install, deploy, smoke-test, and teardown helpers

## Quickstart

```bash
make flink-local-install
make flink-local-deploy
make flink-local-smoke-test
```

These commands install the local demo dependencies, deploy the operator-managed Flink application, and submit the sample Kafka producer job.

## What Is Included

- a Helm chart for local Kafka and MinIO demo dependencies
- a Helm chart for the Flink application centered on `FlinkDeployment`
- an operator wrapper chart for namespace bootstrap and opinionated upstream values
- local and production-style values files
- docs for production namespaces, autoscaling, savepoint-driven upgrades, and cost controls

## Notes

- The repo does not bundle the upstream Flink Kubernetes Operator release itself. Install that operator in your cluster before deploying the application chart.
- The application image is intentionally a contract surface. Teams are expected to provide an image containing the Flink job jar and any Kafka or Paimon connectors they require.
