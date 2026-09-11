# Install Intelligent Assistant with the RHDH 2.1 Operator

> **Draft status:** Review draft for RHDH 2.1.

The RHDH 2.1 Operator includes an `intelligent-assistant` flavour. It is enabled by default. The flavour adds the `lightspeed-core` sidecar and bundled configuration.

A minimal `Backstage` custom resource includes the flavour:

```yaml
apiVersion: rhdh.redhat.com/v1alpha5
kind: Backstage
metadata:
  name: my-rhdh
spec: {}
```

The sidecar remains unconfigured until an inference provider is defined in `lightspeed-stack.yaml` and its required environment values are supplied.

To disable the default flavour:

```yaml
spec:
  flavours:
    - name: intelligent-assistant
      enabled: false
```

An existing custom resource that explicitly lists `name: lightspeed` must use `name: intelligent-assistant` in 2.1.

## Remove 1.10 assumptions

The 2.1 flavour uses one Lightspeed Core runtime file. Remove 1.10 references to:

- the `llama-stack-config` ConfigMap
- `/app-root/config.yaml`
- `llama_stack.library_client_config_path`
- provider `ENABLE_OPENAI`, `ENABLE_VLLM`, `ENABLE_VERTEX_AI`, or `ENABLE_OLLAMA` switches

The Operator's 2.1 resources already omit these items. This list applies only to customer-managed ConfigMaps and custom-resource overrides carried over from 1.10.

The bundled `rhdh-profile.py` remains separate. Do not copy or override it unless you intentionally maintain custom prompts and have tested the replacement against RHDH 2.1.

## Create the provider Secret

Create a user-owned Secret in the same namespace as the `Backstage` custom resource. This example is paired with the OpenAI stack ConfigMap in the next section:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: intelligent-assistant-secrets
type: Opaque
stringData:
  OPENAI_API_KEY: replace-me
```

Do not commit a real credential. Setting `OPENAI_API_KEY` does not enable OpenAI by itself. The stack file must contain the matching provider entry. To enable question validation, preserve the complete shipped `shields` block before adding the validation variables described in [Configure Intelligent Assistant](../CONFIGURING.md#configure-question-validation).

Apply the Secret:

```sh
kubectl apply -n rhdh -f intelligent-assistant-secrets.yaml
```

For Vertex AI, mount the Google credential file into `lightspeed-core` and set `GOOGLE_APPLICATION_CREDENTIALS` to its path inside that container. A Secret value containing a host path does not mount the file.

## Create a unified stack ConfigMap

Skip this section when the bundled stack file needs no changes. The Operator owns its bundled ConfigMap and can overwrite direct edits during reconciliation. For custom providers, copy the complete RHDH 2.1 `lightspeed-stack.yaml` into a new ConfigMap that you own.

Keep the `sentence_transformers` provider, the shipped vector-store settings, and the complete `shields` block. Give each LLM provider a unique `id`. Create or update the ConfigMap from the complete local file:

```sh
kubectl create configmap my-lightspeed-stack \
  --namespace rhdh \
  --from-file=lightspeed-stack.yaml=./lightspeed-stack.yaml \
  --dry-run=client \
  --output yaml | kubectl apply -f -
```

See [Configure Intelligent Assistant](../CONFIGURING.md) for provider examples and the complete shield definition.

## Reference the Secret and ConfigMap

Inject the Secret through `extraEnvs.secrets`. Mount the unified stack file through `extraFiles.configMaps`:

```yaml
apiVersion: rhdh.redhat.com/v1alpha5
kind: Backstage
metadata:
  name: my-rhdh
spec:
  flavours:
    - name: intelligent-assistant
      enabled: true
  application:
    extraEnvs:
      secrets:
        - name: intelligent-assistant-secrets
          containers:
            - lightspeed-core
    extraFiles:
      configMaps:
        - name: my-lightspeed-stack
          key: lightspeed-stack.yaml
          mountPath: /app-root
          containers:
            - lightspeed-core
```

Use these values for RHDH 2.1:

- `key` is `lightspeed-stack.yaml`.
- `mountPath` is `/app-root`.
- `containers` contains `lightspeed-core`.

Do not add the stack ConfigMap to the main RHDH container. Do not add a second custom ConfigMap for `config.yaml`.

Apply the custom resource:

```sh
kubectl apply -n rhdh -f backstage.yaml
```

## Change Secret values

The Secret is user-owned. Updating it does not guarantee that the running sidecar reloads environment variables. Restart the managed RHDH Deployment after a Secret change, or make a safe custom-resource update that causes the Operator to roll out the pod.

Resolve the generated Deployment name before restarting it:

```sh
kubectl get deploy -n rhdh
kubectl rollout restart deployment/my-rhdh -n rhdh
kubectl rollout status deployment/my-rhdh -n rhdh
```

Replace `my-rhdh` with the actual Deployment name.

## Verify the installation

```sh
kubectl get backstage -n rhdh my-rhdh -o yaml
kubectl get pods -n rhdh
kubectl get deploy -n rhdh
kubectl logs -n rhdh deployment/my-rhdh -c lightspeed-core
```

Use the actual Deployment name in the logs command. Check that:

- the Operator reports a reconciled `Backstage` resource
- the RHDH pod is ready and contains `lightspeed-core`
- the sidecar mounts the user-owned `lightspeed-stack.yaml` at `/app-root`
- the flavour-managed `rhdh-profile.py` remains present
- no `config.yaml` mount remains
- provider, validation, model, vector-store, and MCP tests pass

Avoid commands that print Secret data during verification.

## Sources

- [RHDH 2.1 Operator Intelligent Assistant documentation](https://github.com/redhat-developer/rhdh-operator/blob/41b057ba97a7c15ba8374b42397c1ba315c6b9d4/docs/intelligent-assistant.md)
- [RHDH 2.1 Operator example](https://github.com/redhat-developer/rhdh-operator/blob/41b057ba97a7c15ba8374b42397c1ba315c6b9d4/examples/intelligent-assistant.yaml)
- [RHDH 2.1 flavour ConfigMaps](https://github.com/redhat-developer/rhdh-operator/blob/41b057ba97a7c15ba8374b42397c1ba315c6b9d4/config/profile/rhdh/default-config/flavours/intelligent-assistant/configmap-files.yaml)
- [Pinned common stack source](https://github.com/redhat-developer/rhdh-intelligent-assistant-configs/blob/e85128ed014a1ce18431fa27d19b5cbb365d9dcc/lightspeed-core-configs/lightspeed-stack.yaml)
