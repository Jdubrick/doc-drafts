# Install Intelligent Assistant with the RHDH 2.1 Helm chart

> **Draft status:** Review draft verified against chart tag `redhat-developer-hub-2.1.0`, commit `b60cc88`. Do not use later `main` behavior for this procedure.

RHDH 2.1 uses the standalone `charts/rhdh` chart named `redhat-developer-hub`. It is a clean break from the legacy 1.x `charts/backstage` chart.

An in-place `helm upgrade` from a 1.x `backstage` release to this chart is not supported. Install a fresh 2.1 release with migrated values. Follow [the full migration checklist](../MIGRATION.md) before moving production traffic.

Use the chart's [official RHDH 1.x to 2.x migration guidance](https://github.com/redhat-developer/rhdh-chart/blob/b60cc88b47abe9be981b969e884747d7f359992c/charts/rhdh/README.md#upgrading-from-the-backstage-chart-rhdh-1y) for the full chart migration. This page covers only the Intelligent Assistant settings.

## What the 2.1 chart deploys

When `intelligentAssistant.enabled` is `true`, the chart:

- creates and mounts `lightspeed-stack.yaml` and `rhdh-profile.py` ConfigMaps unless you name existing ConfigMaps
- adds the `lightspeed-core` sidecar
- loads an existing provider Secret into the sidecar when configured

The chart does not create a provider Secret. Port 8080 belongs to the sidecar inside the RHDH pod.

## Map the 1.10 values

Replace `global.lightspeed.*` with the corresponding root `intelligentAssistant.*` keys:

- `global.lightspeed.enabled` becomes `intelligentAssistant.enabled`.
- old stack ConfigMap settings become `intelligentAssistant.config.stack.existingConfigMap.name` and `.key`.
- old profile ConfigMap settings become `intelligentAssistant.config.profile.existingConfigMap.name` and `.key`.
- the old `name: server` ConfigMap entry has no replacement because 2.1 removes `config.yaml`.
- `global.lightspeed.secret.create` and `.name` become a pre-created Secret plus `intelligentAssistant.existingSecret`.
- old sidecar settings move under `intelligentAssistant.core`.
- old runtime volume settings move under `intelligentAssistant.runtimeVolume`.

The old chart mounted three configuration files. The 2.1 chart mounts two:

```text
1.10: lightspeed-stack.yaml, config.yaml, rhdh-profile.py
2.1:  lightspeed-stack.yaml, rhdh-profile.py
```

Do not create or mount `config.yaml` for 2.1. The unified stack file contains the supported runtime configuration.

The chart migration guide covers changes outside Intelligent Assistant. Build and review a fresh 2.1 values file rather than treating the new keys as aliases.

## Create a unified stack ConfigMap

Skip this step if the bundled file meets your needs. To configure a provider, copy the complete stack file shipped by chart 2.1.0 and edit that copy. Keep every section you do not intend to change, including the complete `shields` block.

Create the target namespace first if it does not exist:

```sh
kubectl create namespace rhdh
```

Create or update the ConfigMap from the complete local file:

```sh
kubectl create configmap my-lightspeed-stack \
  --namespace rhdh \
  --from-file=lightspeed-stack.yaml=./lightspeed-stack.yaml \
  --dry-run=client \
  --output yaml | kubectl apply -f -
```

See [configuration details](../CONFIGURING.md) for provider examples and the complete shield definition.

## Provide provider credentials

Create the Secret before installing the chart:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: intelligent-assistant-secrets
type: Opaque
stringData:
  OPENAI_API_KEY: replace-me
```

```sh
kubectl apply -n rhdh -f intelligent-assistant-secrets.yaml
```

Do not commit real credentials. The Secret can contain other provider and validation fields documented in [Configure Intelligent Assistant](../CONFIGURING.md), but include only the fields you use. To enable question validation, preserve the complete shipped `shields` block before adding its three validation variables.

## Create 2.1 values

```yaml
intelligentAssistant:
  enabled: true
  existingSecret: intelligent-assistant-secrets
  config:
    stack:
      existingConfigMap:
        name: my-lightspeed-stack
        key: lightspeed-stack.yaml
    profile:
      existingConfigMap:
        name: ""
        key: ""
  runtimeVolume:
    type: emptyDir
```

Leaving the profile ConfigMap name empty tells the chart to create the bundled `rhdh-profile.py` ConfigMap. For a custom key, set both the existing ConfigMap `name` and `key`. The stack ConfigMap key does not have to be named `lightspeed-stack.yaml`, but setting it explicitly avoids ambiguity.

If feedback, cache, or vector data must survive pod replacement, configure a tested `persistentVolumeClaim` under `intelligentAssistant.runtimeVolume`. Do not claim persistence with the default `emptyDir`.

Use `intelligentAssistant.core` only for supported sidecar overrides. Do not copy image tags from development branches into production values.

## Install a fresh release

```sh
helm repo add redhat-developer https://redhat-developer.github.io/rhdh-chart
helm repo update
helm install my-rhdh redhat-developer/redhat-developer-hub \
  --namespace rhdh \
  --create-namespace \
  --version 2.1.0 \
  --values values-2.1.yaml
```

Use a new release name or namespace if the 1.10 release still exists. Do not target the 1.10 release with this command.

## Apply Secret changes

Updating the Secret object does not change the Helm release and might not restart the pod. Restart the RHDH Deployment or rerun a no-op chart upgrade with the saved 2.1 values so the `lightspeed-core` process receives the new environment.

```sh
helm upgrade my-rhdh redhat-developer/redhat-developer-hub \
  --namespace rhdh \
  --version 2.1.0 \
  --values values-2.1.yaml
```

## Verify the release

```sh
helm status my-rhdh -n rhdh
kubectl get pods -n rhdh
kubectl get deploy -n rhdh -l app.kubernetes.io/instance=my-rhdh
kubectl logs -n rhdh deployment/my-rhdh -c lightspeed-core
```

The exact Deployment name can differ when `nameOverride` or `fullnameOverride` is set. Use the name returned by `kubectl get deploy`.

Check that:

- the pod has the main RHDH container and `lightspeed-core`
- the intended stack and profile ConfigMaps are mounted
- no `config.yaml` volume or mount remains
- provider, model, validation, vector-store, and MCP tests pass

## Sources

- [Chart 2.1.0 README](https://github.com/redhat-developer/rhdh-chart/blob/b60cc88b47abe9be981b969e884747d7f359992c/charts/rhdh/README.md)
- [Chart 2.1.0 values](https://github.com/redhat-developer/rhdh-chart/blob/b60cc88b47abe9be981b969e884747d7f359992c/charts/rhdh/values.yaml)
- [Intelligent Assistant ConfigMap template](https://github.com/redhat-developer/rhdh-chart/blob/b60cc88b47abe9be981b969e884747d7f359992c/charts/rhdh/templates/intelligent-assistant/intelligent-assistant-configmaps.yaml)
- [Deployment template](https://github.com/redhat-developer/rhdh-chart/blob/b60cc88b47abe9be981b969e884747d7f359992c/charts/rhdh/templates/deployment.yaml)
