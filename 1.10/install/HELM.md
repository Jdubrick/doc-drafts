# RHDH Helm Chart Installation of Lightspeed

## Documentation From RHDH Helm Chart Repository

https://github.com/redhat-developer/rhdh-chart/blob/main/charts/backstage/README.md

## Extra Information

Lightspeed is enabled by default in the RHDH Helm chart. Everything is controlled via the `values.yaml` file. You can see the file here: https://github.com/redhat-developer/rhdh-chart/blob/main/charts/backstage/values.yaml

Lightspeed Specific portion of the README: https://github.com/redhat-developer/rhdh-chart/blob/main/charts/backstage/values.yaml#L50-L209

## Toggling Lightspeed

Lightspeed is enabled by default, to toggle the enablement you can change the boolean in:

```yaml
global.lightspeed.enabled
```

Accepted values are `true` and `false`.

## Managing Secrets for Lightspeed With Helm

The RHDH Helm chart has the following implementation for the Lightspeed Secret file:

```yaml
global:
  secret:
    create: true
    name: ""
    optional: false
    sourceFile: secret.yaml
```

With the above configuration (default), the Helm install will create a Kubernetes Secret with the following keys:

```yaml
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
**Note:** Will be the same keys as the Operator

For information about what each are see [ENABLING_LLMS.md](../ENABLING_LLMS.md)

On subsequent `Helm upgrade` cycles, this Secret will have its contents wiped. To preserve your changes you should:

1. Make a Kubernetes Secret using the default as a base
2. Update the `values.yaml` file to use that Secret instead
   1. This will persist it through `upgrade` commands

For example if you created a Secret named "my-secret":
```yaml
global:
  secret:
    create: false
    name: "my-secret"
```

## Managing Config Maps for Lightspeed With Helm

Similar to the Secret mentioned above, you will sometimes need to make changes to Lightspeed configuration files, and you do not want them to be overwritten on `helm upgrade` cycles.

To preserve your changes for changes to the:
* `lightspeed-stack.yaml`
* `config.yaml`
* `rhdh-profile.py`

You should:
1. Make a Config Map using the default as a reference
2. Update the `values.yaml` file to use that file instead

Example with `lightspeed-stack.yaml`:
```yaml
global:
  configMaps:
    - name: stack
      create: false
      nameOverride: "my-stack"
      mountPath: /app-root/lightspeed-stack.yaml
      subPath: lightspeed-stack.yaml
      sourceFile: lightspeed-stack.yaml
      optional: false
```

Similar overrides are possible for the `config.yaml` and `rhdh-profile.py` files: https://github.com/redhat-developer/rhdh-chart/blob/main/charts/backstage/values.yaml#L119C1-L140C24

## Overriding Images

You can change the image for the RAG container and Lightspeed Core container by updating the `values.yaml` file.

* `global.lightspeed.initContainer.image`
* `global.lightspeed.sidecar.image`

Both of these take full image strings, e.g:
```yaml
global:
  lightspeed:
    sidecar:
      image: quay.io/xyz/abc:123
```