# Kustomize Setup for MongoDB & API Deployment

This project uses Kustomize to deploy a MongoDB StatefulSet and an API service for a User Management application, with different overlays for `dev` and `prod` environments. The deployment includes configurations for PersistentVolumes, Secrets, ConfigMaps, and Ingress for the API service.

## Project Structure

The project follows a standard Kustomize directory layout, with `base` and `overlays` directories to manage resources and configurations.

```
kustomize
├── base
│   ├── 0-pv.yaml
│   ├── 1-namespace.yaml
│   ├── 2-configMap.yaml
│   ├── 3-mongo-secret.yaml
│   ├── 4-pvc.yaml
│   ├── 5-statefulSet.yaml
│   ├── 6-api-deployment.yaml
│   └── kustomization.yaml        # Base kustomization configuration
├── overlays
│   └── prod
│       ├── 7-api-ingress.yaml
│       ├── kustomization.yaml    # Production kustomization with patches
│       ├── patch-replicas.yaml
│       ├── patch-resources.yaml
│       └── patch-secret.yaml
└── README.md
```

## Base Resources

The base directory contains the foundational YAML files and kustomization configuration for the development environment.

- **PersistentVolume (`0-pv.yaml`)**: Defines storage for MongoDB using hostPath.
- **Namespace (`1-namespace.yaml`)**: Creates a `dev` namespace for the project.
- **ConfigMap (`2-configMap.yaml`)**: Holds API configuration (e.g., port).
- **Secret (`3-mongo-secret.yaml`)**: Stores MongoDB credentials.
- **PersistentVolumeClaim (`4-pvc.yaml`)**: Claims storage for MongoDB.
- **StatefulSet (`5-statefulSet.yaml`)**: Manages MongoDB instances.
- **API Deployment (`6-api-deployment.yaml`)**: Deploys the API service with MongoDB URI from the secret.


## Overlays

The overlays directory contains environment-specific customizations.

### Production Overlay

The `prod` overlay modifies the base configuration with additional resources and patches for production-specific settings.

- **Ingress (`7-api-ingress.yaml`)**: Configures NGINX Ingress to expose the API.
- **patch-replicas.yaml**: Scales the API replicas to 5 in production.
- **patch-resources.yaml**: Adjusts CPU and memory resources for the API service.
- **patch-secret.yaml**: Updates the MongoDB URI for the production environment.



### Production Kustomization

The `kustomization.yaml` for production uses patches to modify:

- **API Deployment Resources**: Adjust memory and CPU limits for better production performance.
- **API Deployment Replicas**: Scale the API service to handle production traffic.
- **MongoDB URI**: Uses the production MongoDB service.

### Kustomize Patches

- **patch-replicas.yaml**: Scales the API replicas for production.

  ```yaml
  - op: replace
    path: /spec/replicas
    value: 5
  ```

- **patch-resources.yaml**: Adjusts the resource limits for the API container.

  ```yaml
  - op: replace
    path: /spec/template/spec/containers/0/resources
    value:
      requests:
        memory: "128Mi"
        cpu: "250m"
      limits:
        memory: "256Mi"
        cpu: "500m"
  ```

- **patch-secret.yaml**: Updates the MongoDB URI for production.

  ```yaml
  - op: replace
    path: /stringData/MONGODB_URI
    value: mongodb://root:example@prod-mongo-0.prod-mongo-headless.prod.svc.cluster.local:27017/user_management_db?authSource=admin
  ```

## Requirements for this Project:

- Kubernetes cluster (Kind or any other provider)
- kubectl and Kustomize installed
- NGINX Ingress controller installed (for production)
- metallb for local cluster

## Deployment Instructions

1. **Set up the cluster**: Ensure your Kubernetes cluster is running and has access to persistent storage.

2. **Apply production overlay** (optional for production deployment):

   ```bash
   kubectl apply -k ./overlays/prod
   ```

3. **Verify**: Ensure the resources are running as expected by checking the Kubernetes dashboard or running:

   ```bash
   kubectl get all -n prod
   ```

## Ingress

The production environment uses an NGINX Ingress to expose the API service. The ingress rule is set for the domain `ts-api.com`. Ensure that the domain points to your ingress controller.

```bash
kubectl get ing -n prod
```

## Cleanup

To clean up resources from your Kubernetes cluster:

```bash
kubectl delete -k ./overlays/prod
```



This project provides a structured approach to deploying a MongoDB-backed API service in both development and production environments using Kustomize. By leveraging overlays, you can manage environment-specific configurations efficiently while maintaining a clean and reusable base configuration.
