# Multi-istance Helm chart deployment with ArgoCD ApplicationSet

This tutorial will guide you through deploying the `multi-instance-app` Helm Chart using Argo CD ApplicationSet on Minikube. We will deploy three instances of the chart with different titles.

Prerequisites:

- Minikube installed and running
- kubectl and helm installed on your local machine
- Argo CD installed on your Kubernetes cluster
- Access to the `multi-instance-app` Helm Chart repository: https://github.com/banshee86vr/argocd-applicationset/tree/main/multi-instance-app

## Step 1: Start Minikube

Start Minikube with the following command:

```sh
minikube start
```

Step 2: Install Argo CD

Follow the instructions provided in the official Argo CD documentation to install Argo CD on your Kubernetes cluster:
https://argo-cd.readthedocs.io/en/stable/getting_started/#installing-argo-cd

## Step 3: Create the Argo CD ApplicationSet

Create the Argo CD ApplicationSet manifest file with the following content:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: multi-instance-app-as
  namespace: argocd
spec:
  generators:
    - git:
        repoURL: https://github.com/banshee86vr/argocd-applicationset.git
        revision: HEAD
        files:
          - path: "customers/dev/**.json"
  template:
    spec:
      source:
        repoURL: https://github.com/banshee86vr/argocd-applicationset.git
        targetRevision: HEAD
        path: multi-instance-app
        helm:
          valueFiles:
            - values.yaml
          values: |
            title: "{{ title }}"
      destination:
        server: "https://kubernetes.default.svc"
        namespace: multi-instance-app
      syncPolicy:
        automated:
          selfHeal: true
        syncOptions:
          - CreateNamespace=true
          - Prune=true
          - Validate=true
          - Timeout=10m
          - retries=3
          - RetryBackoff=5000
          - SkipHooks=".*"
          - SkipApplyOnPrune=true
          - Replace=true
      project: multi-instance-app
    metadata:
      name: "{{title}}"
```