# Migrate Intelligent Assistant from RHDH 1.10 to 2.1

> **Draft status:** Review draft for RHDH 2.1.

Do not merge the 1.10 YAML files and the 2.1 YAML file. Start with a fresh copy of the `lightspeed-stack.yaml` shipped by your RHDH 2.1 installer, then reapply supported customizations one section at a time.

The standalone 2.1 Helm chart does not support an in-place `helm upgrade` from the legacy 1.x `backstage` chart. Plan a fresh Helm release. This guide does not claim support for an in-place chart migration.

## Before migration

1. Record the installed RHDH and Intelligent Assistant versions.
2. Back up the 1.10 Helm values or `Backstage` custom resource.
3. Export customer-managed ConfigMaps, Secrets, app-config, dynamic plugin configuration, RBAC policy, Routes, Services, and persistent-volume claims.
4. Store credentials through your approved secret-management process. Do not add decoded Secret values to the migration workspace.
5. Record enabled providers, model names, MCP endpoints, feedback settings, prompt customizations, storage requirements, and validation behavior.
6. Prepare a separate namespace or release name when that fits your rollback plan. Keep the 1.10 deployment available until 2.1 verification finishes.

## Migrate the common Lightspeed Core configuration

### Start with the shipped 2.1 file

Copy the complete 2.1 `lightspeed-stack.yaml` from the chart or Operator flavour. Do not start with either 1.10 file. Keep this runtime baseline:

```yaml
llama_stack:
  use_as_library_client: true
  config:
    baseline: byo-llm
```

Do not configure generated `/tmp/.generated/run.yaml` output.

### Do not carry the second runtime file forward

RHDH 1.10 used:

```text
lightspeed-stack.yaml
config.yaml
```

RHDH 2.1 uses:

```text
lightspeed-stack.yaml
```

The RHDH 2.1 Helm chart and Operator flavour do not mount `config.yaml`. You do not need to remove a volume or mount from the new deployment. Do not add this old reference to the migrated stack file:

```yaml
llama_stack:
  library_client_config_path: /app-root/config.yaml
```

The separate `rhdh-profile.py` prompt profile still exists. It is not the removed runtime file.

A customer-managed or orphaned `config.yaml` ConfigMap might remain in the namespace after the 2.1 deployment starts. It is not active unless another workload still mounts it. Keep it while the 1.10 deployment remains available for rollback. After the cutover, delete it only after confirming that no workload references it.

### Recreate inference providers

Old 1.10 `config.yaml` provider blocks under `providers.*` do not map line for line to 2.1. Add an explicit entry for each enabled provider under `inference.providers` in the new file.

```yaml
inference:
  providers:
    - type: sentence_transformers
    - type: openai
      id: openai-primary
      api_key_env: OPENAI_API_KEY
```

- Keep `sentence_transformers`.
- Give each provider a unique `id`.
- Replace `ENABLE_OPENAI`, `ENABLE_VLLM`, `ENABLE_VERTEX_AI`, and `ENABLE_OLLAMA` switches with provider definitions.
- Keep credentials and endpoint values in the sidecar Secret.
- Treat Ollama as a vLLM-compatible provider with its `/v1` endpoint.
- Set every vLLM-compatible base URL to the full endpoint ending in `/v1`.
- Use `allowed_models` when the provider should expose only a known set of models.

Setting a Secret key alone does not enable a provider. See [provider examples](CONFIGURING.md#configure-inference-providers).

### Keep the supplied vector store

The 2.1 installer supplies the new `vector_store` section. If you used the standard 1.10 vector-store configuration, leave the supplied 2.1 section alone. There is no manual configuration translation for that case.

This does not migrate existing vector data. The default storage path changed, so decide whether indexed notebook content must survive the cutover. If it must, verify the source and destination volumes and plan the data movement before switching traffic.

If you customized vector storage, translate those requirements into the singular Lightspeed Core section:

```yaml
vector_store:
  default_provider: notebooks
  providers:
    - id: notebooks
      type: faiss
      embedding_model: nomic-ai/nomic-embed-text-v1.5
      embedding_dimension: 768
      config:
        path: /tmp/vector_db/notebooks/faiss_store.db
```

Do not copy old `providers.vector_io`, `registered_resources.vector_stores`, or related storage-provider entries from `config.yaml`. Check whether the installer gives `/tmp` persistent storage. Do not assume that data under this path survives pod replacement.

### Replace the old safety configuration

Do not copy the old safety provider or registered shield resource. Keep the 2.1 top-level `shields` block from the shipped file.

To enable question validation, inject:

```yaml
ENABLE_VALIDATION: question_validity
VALIDATION_PROVIDER: openai-primary
VALIDATION_MODEL_NAME: gpt-4o-mini
```

The provider ID and model must match an enabled inference provider. To keep the shield disabled, leave `ENABLE_VALIDATION` unset or set it to `__disabled__`. Remove old nonempty values such as `true`.

### Keep MCP server settings

The top-level `mcp_servers` schema in `lightspeed-stack.yaml` is unchanged from RHDH 1.10. Leave the shipped RHDH MCP actions entry in place. Copy any customer-defined server entries without changing their schema, and keep tokens out of the ConfigMap. RHDH app-config settings for MCP clients and user tokens are outside this runtime-file migration.

The old `config.yaml` contained an internal `tool_runtime` provider for MCP. The 2.1 runtime supplies that integration. Do not copy the internal provider into `lightspeed-stack.yaml`. See the [official RHDH 1.10 MCP configuration](https://docs.redhat.com/en/documentation/red_hat_developer_hub/1.10/html/interacting_with_model_context_protocol_tools_for_red_hat_developer_hub/configure-mcp-tokens-and-endpoints-to-authorize-client-access_interacting-with-model-context-protocol-tools-for-rhdh) for the previous top-level schema.

### Leave unchanged sections alone

Leave every other section in the shipped 2.1 file unchanged. If you customized feedback collection, the conversation cache, authentication, or the prompt profile in 1.10, compare those custom values with the 2.1 file and reapply only the values you still need. Do not replace a complete 2.1 section with its 1.10 version.

## Complete the Helm migration

Follow the chart's [official RHDH 1.x to 2.x migration guidance](https://github.com/redhat-developer/rhdh-chart/blob/b60cc88b47abe9be981b969e884747d7f359992c/charts/rhdh/README.md#upgrading-from-the-backstage-chart-rhdh-1y) for the full Helm migration. At chart 2.1.0, the upstream guide still marked the detailed procedure as a work in progress. Use the final published RHDH upgrade guide when it becomes available.

The Intelligent Assistant-specific work is:

1. Move `global.lightspeed.*` customizations to the matching root `intelligentAssistant.*` keys.
2. Do not migrate the old `name: server` ConfigMap entry. The 2.1 chart has stack and profile entries only.
3. Pre-create the provider Secret and set `intelligentAssistant.existingSecret`.
4. Supply a custom unified stack ConfigMap through `intelligentAssistant.config.stack.existingConfigMap` only when you need to customize the shipped file.
5. Install a fresh `redhat-developer-hub` chart 2.1.0 release as directed by the chart migration guide. Do not run `helm upgrade` against the 1.x `backstage` release.

## Complete the Operator migration

Follow the [Operator procedure](install/OPERATOR.md). In summary:

1. Change any explicit `lightspeed` flavour name to `intelligent-assistant`.
2. Create the provider Secret and inject it into `lightspeed-core` through `spec.application.extraEnvs.secrets`.
3. Create a complete unified `lightspeed-stack.yaml` ConfigMap when customization is required.
4. Mount that ConfigMap at `/app-root` in `lightspeed-core` through `spec.application.extraFiles.configMaps`.
5. Do not carry a custom `llama-stack-config` mount or `config.yaml` reference into the 2.1 custom resource.
6. Keep the flavour-managed `rhdh-profile.py` unless you intentionally maintain a custom prompt profile.

## Verify the migrated deployment

1. Confirm that the RHDH pod is ready and contains `lightspeed-core`.
2. Confirm that `/app-root/lightspeed-stack.yaml` comes from the intended ConfigMap and that the 2.1 pod has no `/app-root/config.yaml` mount.
3. Review `lightspeed-core` logs for configuration, provider, credential, model, vector-store, and MCP connection errors. Do not print Secret values.
4. Open the assistant and submit a basic development question.
5. Test each enabled provider and an allowed model.
6. If validation is enabled, test one developer question and one clearly unrelated question.
7. Test required MCP tools.
8. Test feedback and notebook behavior when enabled.
9. Restart the pod and confirm the expected persistence behavior.

## Roll back

Keep the 1.10 manifests, values, images, ConfigMaps, and secret references until the 2.1 acceptance tests pass. For a parallel Helm installation, direct traffic back to the 1.10 release if verification fails. For the Operator, restore the backed-up custom resource and associated customer-managed resources according to the Operator's supported downgrade policy.

Do not overwrite 1.10 persistent data with an untested 2.1 path. Take storage-level backups before changing claims or database schemas.

## Sources

- [Unified configuration change](https://github.com/redhat-developer/rhdh-intelligent-assistant-configs/commit/bee6d08114aee054b6bc0ffefc1aefb4b8534245)
- [Pinned 2.1-era configuration](https://github.com/redhat-developer/rhdh-intelligent-assistant-configs/blob/e85128ed014a1ce18431fa27d19b5cbb365d9dcc/lightspeed-core-configs/lightspeed-stack.yaml)
- [Helm 2.1 chart migration notice](https://github.com/redhat-developer/rhdh-chart/blob/b60cc88b47abe9be981b969e884747d7f359992c/charts/rhdh/README.md#upgrading-from-the-backstage-chart-rhdh-1y)
- [RHDH 2.1 Operator Intelligent Assistant documentation](https://github.com/redhat-developer/rhdh-operator/blob/41b057ba97a7c15ba8374b42397c1ba315c6b9d4/docs/intelligent-assistant.md)
- [Official RHDH 1.10 MCP client configuration](https://docs.redhat.com/en/documentation/red_hat_developer_hub/1.10/html/interacting_with_model_context_protocol_tools_for_red_hat_developer_hub/configure-mcp-tokens-and-endpoints-to-authorize-client-access_interacting-with-model-context-protocol-tools-for-rhdh)
