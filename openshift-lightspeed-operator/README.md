# OpenShift Lightspeed

Install the OpenShift Lightspeed operator, then one provider instance.

Do not use `operator/base` directly. The channel is set in an overlay.

The current overlays available are for the following channels:

* [stable](operator/overlays/stable)
* [alpha](operator/overlays/alpha)

## Usage

If you have cloned the `gitops-catalog` repository, you can install OpenShift Lightspeed based on the overlay of your choice by running from the root (`gitops-catalog`) directory.

```
oc apply -k openshift-lightspeed-operator/operator/overlays/<channel>
```

Or, without cloning:

```
oc apply -k https://github.com/redhat-cop/gitops-catalog/openshift-lightspeed-operator/operator/overlays/<channel>
```

As part of a different overlay in your own GitOps repo:

```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - https://github.com/redhat-cop/gitops-catalog/openshift-lightspeed-operator/operator/overlays/<channel>?ref=main
```

`<channel>` is `stable` or `alpha`.

## Instance

`instance/base` is the shared OLSConfig named `cluster`. Do not apply the base on its own. Apply one provider overlay. A second overlay replaces that same OLSConfig.

```
openai
azure-openai
gemini-api
gemini-vertex
vertex-anthropic
rhoai-vllm
rhelai-vllm
watsonx
```

`azure-entra` is the `azure-openai` overlay. The secret name stays `azure-api-keys`. Entra ID uses `client_id`, `client_secret`, and `tenant_id` in that secret instead of `apitoken`.

Each overlay names the secret and leaves the token, key, project, URL, and model as placeholders. Create that secret before applying the instance. Sealed Secrets or External Secrets, already separate components in this catalog, can supply it. This component does not install either of those.

```
oc apply -k openshift-lightspeed-operator/instance/overlays/<provider>
```

## ArgoCD

Sync the operator overlay and one instance overlay. The Application does not contain the token. Create the secret named by that overlay before the instance syncs.

Operator, channel `stable`:

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: openshift-lightspeed-operator
  namespace: openshift-gitops
spec:
  project: default
  source:
    repoURL: https://github.com/redhat-cop/gitops-catalog
    targetRevision: main
    path: openshift-lightspeed-operator/operator/overlays/stable
  destination:
    server: https://kubernetes.default.svc
  syncPolicy:
    syncOptions:
      - CreateNamespace=false
```

One instance. This example is `rhoai-vllm`. Change `path` for another provider in the list above. The secret for this example is `rhoai-api-keys` in `openshift-lightspeed`, key `apitoken`.

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: openshift-lightspeed-instance
  namespace: openshift-gitops
spec:
  project: default
  source:
    repoURL: https://github.com/redhat-cop/gitops-catalog
    targetRevision: main
    path: openshift-lightspeed-operator/instance/overlays/rhoai-vllm
  destination:
    server: https://kubernetes.default.svc
  syncPolicy:
    syncOptions:
      - CreateNamespace=false
```
