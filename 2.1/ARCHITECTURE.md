# Intelligent Assistant configuration boundaries

> **Draft status:** Review draft for RHDH 2.1.

## What RHDH connects to

The Intelligent Assistant frontend and backend plugins connect to the Lightspeed Core sidecar in the RHDH pod. RHDH does not require customers to deploy, configure, or call a separate Llama Stack service.

Lightspeed Core uses an embedded runtime internally. The `llama_stack` section in `lightspeed-stack.yaml` selects the `byo-llm` baseline for that runtime. This internal detail does not create a second customer configuration file or a customer-facing Llama Stack API dependency.

```text
User
  -> Intelligent Assistant UI
  -> Intelligent Assistant backend
  -> Lightspeed Core sidecar
       -> inference provider
       -> question-validation shield
       -> MCP servers
       -> vector store
```

Configure Lightspeed Core. Do not edit the synthesized `/tmp/.generated/run.yaml` file. Lightspeed Core creates that file at runtime from `lightspeed-stack.yaml`.

## Three different configuration types

### RHDH app-config

RHDH `app-config` configures the Backstage application and the Intelligent Assistant plugin.

This file does not define inference providers, vector stores, shields, or MCP server endpoints for Lightspeed Core.

### Lightspeed Core runtime configuration

`/app-root/lightspeed-stack.yaml` configures Lightspeed Core. It contains the `inference`, `vector_store`, `shields`, `skills`, and `mcp_servers` sections covered by this guide.

RHDH 1.10 used both `lightspeed-stack.yaml` and `config.yaml`. RHDH 2.1 consolidates that information into `lightspeed-stack.yaml`. The old `llama_stack.library_client_config_path` setting no longer belongs in the file.

The shipped `rhdh-profile.py` remains a separate prompt profile. Most installations should use the bundled profile and replace only `lightspeed-stack.yaml`.

### Kubernetes Secrets

Kubernetes Secrets supply credentials and environment-specific values to the `lightspeed-core` container. For example, a Secret can hold `OPENAI_API_KEY`, `VLLM_URL`, or validation selections.

A Secret does not define or enable an inference provider. The corresponding entry must exist under `inference.providers` in `lightspeed-stack.yaml`. The `api_key_env` field tells Lightspeed Core which environment variable contains a provider key without putting that credential in the ConfigMap.

For Vertex AI, `GOOGLE_APPLICATION_CREDENTIALS` must contain the path inside the `lightspeed-core` container. Mount the credential file separately. A host path or the JSON credential itself is not a valid value for this environment variable.

## Installation ownership

The configuration content is common, but the installation method mounts it differently:

- The Helm chart creates stack and profile ConfigMaps by default. Set `intelligentAssistant.config.stack.existingConfigMap` to use a customer-managed stack ConfigMap. Set `intelligentAssistant.existingSecret` to load a customer-managed Secret.
- The Operator flavour owns its bundled ConfigMaps. Mount a customer-managed stack ConfigMap through `spec.application.extraFiles.configMaps` and inject a Secret through `spec.application.extraEnvs.secrets`.

See [Helm installation](install/HELM.md) or [Operator installation](install/OPERATOR.md) for the exact resource shape.

## Sources

- [RHDHPLAN-1190](https://redhat.atlassian.net/browse/RHDHPLAN-1190)
- [Pinned 2.x configuration](https://github.com/redhat-developer/rhdh-intelligent-assistant-configs/blob/e85128ed014a1ce18431fa27d19b5cbb365d9dcc/lightspeed-core-configs/lightspeed-stack.yaml)
- [Helm 2.1 deployment template](https://github.com/redhat-developer/rhdh-chart/blob/b60cc88b47abe9be981b969e884747d7f359992c/charts/rhdh/templates/deployment.yaml)
- [RHDH 2.1 Operator guidance](https://github.com/redhat-developer/rhdh-operator/blob/41b057ba97a7c15ba8374b42397c1ba315c6b9d4/docs/intelligent-assistant.md)
