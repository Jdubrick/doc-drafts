# Intelligent Assistant documentation drafts for RHDH 2.1

> **Draft status:** Review draft for RHDH 2.1.

These drafts describe how to configure and migrate the Intelligent Assistant plugin for Red Hat Developer Hub 2.1. They focus on [RHDHPLAN-1190](https://redhat.atlassian.net/browse/RHDHPLAN-1190): RHDH talks to Lightspeed Core, and Lightspeed Core owns the model, validation, MCP, and vector-store configuration.

## Read these drafts

- [Architecture](ARCHITECTURE.md) explains the configuration boundaries.
- [Configure Lightspeed Core](CONFIGURING.md) documents the single `lightspeed-stack.yaml` file.
- [Migrate from RHDH 1.10](MIGRATION.md) gives a common migration checklist, followed by Helm and Operator work.
- [Install with Helm](install/HELM.md) covers the standalone 2.1 chart.
- [Install with the Operator](install/OPERATOR.md) covers the 2.1 `Backstage` custom resource.

## Source snapshot

The drafts use these pinned sources:

- The 1.10 comparison uses [`lightspeed-configs` tag `1.10.4`](https://github.com/redhat-ai-dev/lightspeed-configs/tree/1.10.4), the chart `release-1.10` branch, and the Operator `release-1.10` branch.
- The 2.x configuration source moved to [`rhdh-intelligent-assistant-configs` commit `e85128e`](https://github.com/redhat-developer/rhdh-intelligent-assistant-configs/tree/e85128ed014a1ce18431fa27d19b5cbb365d9dcc). The old repository was retired in [commit `e9ee310`](https://github.com/redhat-ai-dev/lightspeed-configs/commit/e9ee310ec12e1cd411f548d480efc8827b3cf1fe).
- The unified configuration landed in [commit `bee6d08`](https://github.com/redhat-developer/rhdh-intelligent-assistant-configs/commit/bee6d08114aee054b6bc0ffefc1aefb4b8534245). The configuration sync then stopped copying a second Llama Stack ConfigMap in [commit `8b61eaa`](https://github.com/redhat-developer/rhdh-intelligent-assistant-configs/commit/8b61eaa01c91ec30325856b67addb72c0a4a02d2).
- Helm guidance uses chart tag [`redhat-developer-hub-2.1.0`](https://github.com/redhat-developer/rhdh-chart/tree/redhat-developer-hub-2.1.0/charts/rhdh), commit [`b60cc88`](https://github.com/redhat-developer/rhdh-chart/commit/b60cc88b47abe9be981b969e884747d7f359992c), and [PR #519](https://github.com/redhat-developer/rhdh-chart/pull/519).
- Operator guidance uses [commit `41b057b`](https://github.com/redhat-developer/rhdh-operator/tree/41b057ba97a7c15ba8374b42397c1ba315c6b9d4) and [PR #3475](https://github.com/redhat-developer/rhdh-operator/pull/3475).

## Release notes

- The Helm chart facts are release facts for chart 2.1.0. Do not substitute files from current `main` when following these procedures.
- The Operator procedure uses the RHDH 2.1 Intelligent Assistant flavour from [commit `41b057b`](https://github.com/redhat-developer/rhdh-operator/tree/41b057ba97a7c15ba8374b42397c1ba315c6b9d4).
