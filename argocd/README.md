

# Argo CD Applications for User Management API

This repository contains configurations for deploying the **User Management API** using **Argo CD**. The deployment is managed using three different methods: **Helm**, **Kubernetes Manifests**, and **Kustomize**. Each method has its own Argo CD Application YAML file described below.



## Argo CD Application using Helm

The `helm.yaml` file defines an Argo CD Application that uses **Helm** for managing the deployment. Helm simplifies the management of Kubernetes applications by packaging all the Kubernetes resources together.

### Breakdown:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-helm-app
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
```

- **name**: `my-helm-app` is the name of this Argo CD application.
- **namespace**: The Argo CD namespace (`argocd`) where this application will be created.
- **finalizers**: Used to clean up resources when deleting the application.

```yaml
spec:
  project: default
  source:
    repoURL: https://github.com/panchanandevops/User-Management-API.git
    targetRevision: main
    path: Deploy/user-helm-chart/
    helm:
      releaseName: user-api
```

- **source**: Specifies the Git repository and directory (`Deploy/user-helm-chart/`) containing the Helm chart.
- **releaseName**: The name of the Helm release (i.e., the deployed application).

```yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: dev
```

- **destination**: The Kubernetes cluster (`kubernetes.default.svc`) and namespace (`dev`) where the Helm chart will be deployed.

```yaml
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
      - Validate=true
      - CreateNamespace=false
      - PrunePropagationPolicy=foreground
      - PruneLast=true
```

- **syncPolicy**: Enables automatic synchronization of resources. `prune: true` ensures that resources not defined in the Git repository are deleted, while `selfHeal: true` automatically corrects any drift between the desired state and the actual state.

---

## Argo CD Application using Kubernetes Manifests

The `k8s-manifesto.yaml` file defines an Argo CD Application that deploys resources using plain **Kubernetes manifests**.

### Breakdown:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
```

- **name**: `my-app` is the name of this Argo CD application using Kubernetes manifests.
- **namespace**: Specifies the Argo CD namespace (`argocd`).

```yaml
spec:
  project: default
  source:
    repoURL: https://github.com/panchanandevops/User-Management-API.git
    targetRevision: HEAD
    path: Deploy/k8s/
```

- **source**: Specifies the Git repository and directory (`Deploy/k8s/`) containing Kubernetes manifests for deploying the application.

```yaml
  destination:
    server: https://kubernetes.default.svc
```

- **destination**: Specifies the Kubernetes cluster where the manifests will be applied.

```yaml
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
      - Validate=true
      - CreateNamespace=false
      - PrunePropagationPolicy=foreground
      - PruneLast=true
```

- **syncPolicy**: Enables automatic synchronization and self-healing of Kubernetes resources. `prune: true` ensures that any resources not defined in the repository are removed.

---

## Argo CD Application using Kustomize

The `kustomize.yaml` file defines an Argo CD Application that uses **Kustomize**. Kustomize allows for overlaying and customizing Kubernetes resources without modifying the original files.

### Breakdown:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-kustomize
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
```

- **name**: `my-app-kustomize` is the name of this Argo CD application using Kustomize.
- **namespace**: Specifies the Argo CD namespace (`argocd`).

```yaml
spec:
  project: default
  source:
    repoURL: https://github.com/panchanandevops/User-Management-API.git
    targetRevision: HEAD
    path: Deploy/kustomize/overlays/prod
```

- **source**: Specifies the Git repository and the path (`Deploy/kustomize/overlays/prod`) that contains the Kustomize overlays for the production environment.

```yaml
  destination:
    server: https://kubernetes.default.svc
```

- **destination**: Specifies the Kubernetes cluster where the resources will be applied.

```yaml
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
      - Validate=true
      - CreateNamespace=false
      - PrunePropagationPolicy=foreground
      - PruneLast=true
```

- **syncPolicy**: Just like the previous applications, this ensures that the application remains in sync with the desired state, automatically healing any differences, and pruning unused resources.

---

## How to Use These Argo CD Applications

To deploy the User Management API using Argo CD, follow these steps:

1. Install [Argo CD](https://argo-cd.readthedocs.io/en/stable/getting_started/).
2. Apply any of the Argo CD Application manifests (Helm, Kubernetes Manifests, or Kustomize) using the `kubectl` command:

   ```bash
   kubectl apply -f helm.yaml   # For Helm-based deployment
   kubectl apply -f k8s-manifesto.yaml   # For Kubernetes Manifest-based deployment
   kubectl apply -f kustomize.yaml   # For Kustomize-based deployment
   ```

3. Watch the application sync with your cluster by accessing the Argo CD dashboard.

4. Modify the GitHub repository to manage updates to your application. Argo CD will automatically sync changes.



