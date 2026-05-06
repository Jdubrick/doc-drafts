# RHDH Operator Installation of Lightspeed

## Documentation From RHDH Operator Repository

* https://github.com/redhat-developer/rhdh-operator/blob/main/docs/lightspeed.md

## Extra Information

Lightspeed is enabled as a default flavour in the Operator, that means when you deploy a Backsage CR then Lightspeed is going to be enabled and the appropriate sidecar resources will be deployed alongside RHDH.

When you apply a Backstage CR, the Operator deploys all of the Lightspeed components automatically and it will be in an "unconfigured" state. This is by design and is reflected in the UI.

There are 2 important pieces you will need to go from Operator Deployed --> RHDH instance up and running with Lightspeed and those are:

* Backstage CR
* Kubernetes Secret file for LLM definitions

There is a good example in the RHDH Operator repository that shows this: https://github.com/redhat-developer/rhdh-operator/blob/main/examples/lightspeed.yaml

### Deploying Kubernetes Secret (Pre-req to Backstage CR)

The following has all of the various Secret keys that could be used for Lightspeed. For information about what each are see [ENABLING_LLMS.md](../ENABLING_LLMS.md)

You need to apply this Secret to the namespace you plan on deploying the Backstage CR:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: llama-stack-secrets
type: Opaque
stringData:
  ENABLE_VLLM: ""
  ENABLE_VERTEX_AI: ""
  ENABLE_OPENAI: ""
  ENABLE_OLLAMA: ""
  ENABLE_VALIDATION: ""
  VLLM_URL: ""
  VLLM_API_KEY: ""
  VLLM_MAX_TOKENS: ""
  VLLM_TLS_VERIFY: ""
  OPENAI_API_KEY: ""
  VERTEX_AI_PROJECT: ""
  VERTEX_AI_LOCATION: ""
  GOOGLE_APPLICATION_CREDENTIALS: ""
  OLLAMA_URL: ""
  VALIDATION_PROVIDER: ""
  VALIDATION_MODEL_NAME: ""
  LLAMA_STACK_LOGGING: ""
```
**Note:** Since this is not Operator managed, when you update the Secret you may need to manually re-rollout the Deployment for changes to take effect.

### Deploying the Backstage CR

Since you have the Secret present in the namespace you wish to deploy the Backstage CR to, you can reference it in the `extraEnvs` section of the CR. 

To finalize RHDH, you can apply the following CR to your cluster:
```yaml
apiVersion: rhdh.redhat.com/v1alpha5
kind: Backstage
metadata:
  name: lightspeed-test
spec:
  application:
    extraEnvs:
      secrets:
        - name: llama-stack-secrets
          containers:
            - lightspeed-core
```

## Persisting Changes to Configuration Files

Sometimes you will need to make changes to the various configuration files used for Lightspeed. The primary file you may need to alter is the `lightspeed-stack.yaml` file. This is done for various reasons, such as adding MCP servers, toggling feedback enablement, using Postgres instead of Sqlite, etc. 

Since the configuration files are deployed by the Operator, they are also manged by the Operator. That means if you make changes to those ConfigMaps they will be overwritten on Operator reconcile.

You are able to instead deploy your own copy of the ConfigMap to the namespace and reference it in the Backstage CR. This will overwrite the "default" by the flavour, so your changes persist.

You can do so by ensuring the ConfigMap is in the namespace and then add it to the Backstage CR:

```yaml
apiVersion: rhdh.redhat.com/v1alpha5
kind: Backstage
metadata:
  name: lightspeed-test
spec:
  application:
    extraEnvs:
      secrets:
        - name: llama-stack-secrets
          containers:
            - lightspeed-core
    extraFiles:
      configMaps:
        - name: "my-configmap"
          mountPath: /app-root
          key: lightspeed-stack.yaml
          containers:
            - lightspeed-core
```

## Overriding Fields (E.g. Images)

You can leverage the Deployment Patching from the Operator to override some fields added by default for Lightspeed, such as images.

https://github.com/redhat-developer/rhdh-operator/blob/main/docs/configuration.md#deployment-patching

