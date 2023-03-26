# Multi-istance Helm chart deployment with ArgoCD ApplicationSet

This tutorial will guide you through deploying the My-Frontend Helm Chart using Argo CD ApplicationSet on Minikube. We will deploy three instances of the chart with different titles.

Prerequisites:

- Minikube installed and running
- kubectl and helm installed on your local machine
- Argo CD installed on your Kubernetes cluster
- Access to the My-Frontend Helm Chart repository: https://github.com/banshee86vr/argocd-applicationset/tree/main/my-frontend

## Step 1: Start Minikube

Start Minikube with the following command:

```sh
minikube start
```

Step 2: Install Argo CD

Follow the instructions provided in the official Argo CD documentation to install Argo CD on your Kubernetes cluster:
https://argo-cd.readthedocs.io/en/stable/getting_started/#installing-argo-cd

## Step 3: Install the My-Frontend Helm Chart

Use the following command to install the My-Frontend Helm Chart:

```sh
helm install my-frontend banshee86vr/my-frontend --set frontend.title="My Frontend"
```

## Step 4: Create the Argo CD ApplicationSet

Create the Argo CD ApplicationSet manifest file with the following content:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: my-frontend
  namespace: argocd
spec:
  generators:
  - list:
      elements:
      - value: frontend1
      - value: frontend2
      - value: frontend3
    input:
      name: frontend
  template:
    spec:
      source:
        repoURL: https://github.com/banshee86vr/argocd-applicationset.git
        targetRevision: HEAD
        path: multi-instance-app
      project: default
      syncPolicy:
        automated: {}
      destination:
        server: 'https://kubernetes.default.svc'
        namespace: default
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
    name: {{inputs.parameters.frontend}}-{{metadata.name}}
    project: default
    source:
      chart:
        spec:
          chart: my-frontend
          sourceRef:
            name: my-frontend
            kind: HelmRepository
            namespace: argocd
      helm:
        values:
        - name: frontend.title
          value: {{inputs.parameters.frontend}}
```

create an helm chart with already existing frontend title loaded from configmap. 

