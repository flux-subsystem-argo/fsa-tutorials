# Introduction

Flamingo is the **Flux Subsystem for Argo** (FSA). Flamingo's container image can be used as a drop-in extension for the equivalent ArgoCD version to visualize, and manage Flux workloads, alongside ArgoCD. You can also ensure that upstream CVEs in Argo CD are quickly backported to Flamingo, maintaining a secure and stable environment.

## Why use Flamingo?

Flamingo is a tool that combines Flux and Argo CD to provide the best of both worlds for implementing GitOps on Kubernetes clusters. With Flamingo, you can:

1. Automate the deployment of your applications to Kubernetes clusters and benefit from the improved collaboration and deployment speed and reliability that GitOps offers.

2. Enjoy a seamless and integrated experience for managing deployments, with the automation capabilities of Flux embedded inside the user-friendly interface of Argo CD.

3. Take advantage of additional features and capabilities that are not available in either Flux or Argo CD individually, such as the robust Helm support from Flux, Flux OCI Repository, Weave GitOps Terraform Controller for Infrastructure as Code, Weave Policy Engine, or Argo CD ApplicationSet for Flux-managed resources.

## How does it work?

![FSA (2)](https://user-images.githubusercontent.com/10666/159503288-5faeda59-8b54-40f0-95ca-b46c22742e30.png)

## Getting Started with Flamingo CLI

Flamingo CLI is the recommended way to install Flamingo for production use.

This guide will provide a step-by-step process for setting up a GitOps environment using Flux and ArgoCD, via Flamingo. By the end of this guide, you will have Flamingo running on your cluster. You will create a podinfo application with a Flux Kustomization, and generate a Flamingo app from this Flux object.

### Install CLIs

You can install required CLI via Homebrew.

```shell
# install Flux CLI
brew install fluxcd/tap/flux

# install Flamingo CLI
brew install flux-subsystem-argo/tap/flamingo
```

### Install Flux

```shell
flux install
```

### Install Flamingo
```shell
flamingo install

# or with a specific Flamingo version
flamingo install --version=v2.8.6
```

### Create a Podinfo Flux Kustomization

```yaml
cat << EOF | kubectl apply -f -
---
apiVersion: v1
kind: Namespace
metadata:
  name: podinfo-kustomize
---
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: OCIRepository
metadata:
  name: podinfo
  namespace: podinfo-kustomize
spec:
  interval: 10m
  url: oci://ghcr.io/stefanprodan/manifests/podinfo
  ref:
    tag: latest
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: podinfo
  namespace: podinfo-kustomize
spec:
  interval: 10m
  targetNamespace: podinfo-kustomize
  prune: true
  sourceRef:
    kind: OCIRepository
    name: podinfo
  path: ./
EOF
```

### Generate App to view the Podinfo KS

```shell
flamingo generate-app \
  --app-name=podinfo-ks \
  -n podinfo-kustomize ks/podinfo
```

### Login to the Flamingo UI

Like a normal Argo CD instance, please firstly obtain the initial password by running the following command to login. The default username is `admin`.

```shell
flamingo show-init-password
```

After that you can port forward and open your browser to http://localhost:8080

```shell
kubectl -n argocd port-forward svc/argocd-server 8080:443
```

## Getting Started with a Fresh KIND cluster

In this getting started guide, you'll be walked through steps to prepare your ultimate GitOps environment using Argo CD and Flux.
We'll bootstrap everything, including installation of Argo CD, from this public repo. So no manual step of Argo CD installation is required.
In case you're forking this repo and change its visibility to private, you will be required to setup a Secret to authenticate your Git repo.

At the end of this guide, you'll have Flux running alongside Argo CD locally on your KIND cluster. You'll run FSA in the anonymous mode, and see 2 pre-defined Argo CD Applications, each of which points to its equivalent Flux Kustomization.

Install CLIs
- [KIND cli](https://kind.sigs.k8s.io/docs/user/quick-start/#installation) 
- [Flux cli](https://fluxcd.io/docs/cmd/)
- [Argo CD cli](https://argo-cd.readthedocs.io/en/stable/cli_installation/)

Example install in macOS or Linux via [homebrew](https://brew.sh/)

```shell
# install KIND cli
brew install kind

# install Flux CLI
brew install fluxcd/tap/flux

# install Argo CD CLI
brew install argocd

```

Create a fresh KIND cluster

```shell
kind create cluster
```

Install **Flux**

```shell
flux install

```

You can check the Flux namespace (`flux-system`) for running pods `kubectl get pods -n flux-system`.

After Pods are ready, then you can copy, and paste this snippet to bootstrap the demo.

```shell
cat <<EOF | kubectl apply -f -
---
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: OCIRepository
metadata:
  name: fsa-demo
  namespace: flux-system
spec:
  interval: 30s
  url: oci://ghcr.io/flux-subsystem-argo/flamingo/manifests
  ref:
    tag: latest
---
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: fsa-demo
  namespace: flux-system
spec:
  prune: true
  interval: 2m
  path: "./demo"
  sourceRef:
    kind: OCIRepository
    name: fsa-demo
  timeout: 3m
EOF
```

You can check Argo CD pods are running and Ready `kubectl get -n argocd pods`.
Finally, port forward and open your browser to http://localhost:8080

```
kubectl -n argocd port-forward svc/argocd-server 8080:443
```

You'll find 2 FSA Applications, each of which consists of 1 Flux's Kustomization and 1 Flux's GitRepository.

![image](https://user-images.githubusercontent.com/10666/161395963-bbabbd72-03f5-4cef-b16d-346afd0eb1fc.png)

![image](https://user-images.githubusercontent.com/10666/161396000-0282f538-88a9-4449-8501-6d7b3a64f2a6.png)