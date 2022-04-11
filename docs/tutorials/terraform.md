# Manage Terraform Resources

> ⏲️ Estimated time used: *10 minutes*.

This is a tutorial to show how could we use **Flux Subsystem for Argo** to bring a Terraform manage feature from the Flux world to your **Argo CD** UI. In order to do so, we need [Weave GitOps Terraform Controller](https://github.com/weaveworks/tf-controller) to help us reconcile our Terraform resources.

## What is Weave GitOps Terraform Controller?

Weave GitOps Terraform Controller aka Weave TF-controller is a Kubernetes controller that allows you to GitOpsify Terraform resources without rewriting them to YAML files. We can use Weave TF-controller in any GitOps environment managed by Flux or Argo CD. In case of this tutorial, We use FSA to help bring Weave TF-controller to Argo CD.

## Tools

We use the following tools in this tutorial.

  * A Kubernetes cluster. EKS on AWS, or `kind` on your desktop.
  * Argo CD **v2.2.8**
  * Flux v2 **0.28.5**
  * Flux Subsystem for Argo **FL.1**
  * Weave GitOps Terraform Controller **v0.9.3 or later**
  * `kubectl`
  * `kustomize`
  * `yq` (optional)


## Create a new KIND cluster

If you already have a `kind` cluster you can skip this step.
Also, if you'd like to use Kubernetes on a cloud like EKS on AWS, you can also skip this step.

To create a new `kind` cluster, please install `kind` CLI and run the following command.

```shell
kind create cluster
```

## Install Argo CD

We use the official Argo CD instructions to install Argo CD. You could skip this step, if you already have Argo CD v2.2.8 installed. It's the non-HA installation. Please refer to the Argo CD documentation for the HA installation which is not covered by this tutorial.

```shell
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.2.8/manifests/install.yaml
```

## Install Flux Subsystem for Argo

At your terminal, please set `FSA_VERSION` variable to a specific version of Flux Subsystem for Argo you'd like to install. In this tutorial, we use `v2.2.8-fl.1-main-305be5e6`.

```shell
export FSA_VERSION=v2.2.8-fl.1-main-305be5e6
```

There are many options to install **Flux Subsystem for Argo**. Please choose **one of the followings** to upgrade your existing Argo CD to FSA, replace the current Argo CD instalation with FSA, or install FSA from scratch.

### Option 1 - Upgrade the existing Argo CD

This option is to upgrade your existing Argo CD.  If you already have Argo CD installed, you would want to go with replacing the image of the existing instllation with the image from FSA.

```shell
kustomize build https://github.com/flux-subsystem-argo/flamingo//release?ref=${FSA_VERSION} \
  | yq e '. | select(.kind=="Deployment" or .kind=="StatefulSet")' - \
  | kubectl -n argocd apply -f - 
```

### Option 2 - Replace the current Argo CD

This option will replace your current installation, including every setting in your configmaps.

```shell
kubectl -n argocd apply -k https://github.com/flux-subsystem-argo/flamingo//release?ref=${FSA_VERSION}
```

### Option 3 - Install FSA from Scratch

With this option, you don't need any existing Argo CD installation.

```shell
kubectl create ns argocd
kubectl -n argocd apply -k https://github.com/flux-subsystem-argo/flamingo//release?ref=${FSA_VERSION}

```

### Login to Argo CD UI

For a fresh Argo CD or FSA installation, you would find your initial admin password by running this command.

```shell
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

Then please forward Argo CD UI's port to localhost:8080, open it with your Web browser and login with `admin` and your password found by the previous command.

```shell
kubectl -n argocd port-forward svc/argocd-server 8080:443
```

## Install Weave GitOps Terraform Controller

**Weave GitOps Terraform Controller** can be installed using its Helm chart. Now we have Flux Subsystem for Argo installed, we'll install a Helm chart via FSA.

Nothing special, just check additional 2 check boxes and you are good to go. With FSA, your Argo CD UI will have 2 additional check boxes, ☑️ **Use Flux Subsystem**, and ☑️ **Auto-Create Flux Resources**.

