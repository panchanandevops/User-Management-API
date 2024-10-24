

# User Helm Chart

## Overview

This Helm chart is designed to deploy the **User Management API** on a Kubernetes cluster. It includes various Kubernetes resources such as Deployments, StatefulSets, Services, Ingress, ConfigMaps, Secrets, Persistent Volumes (PV), and Persistent Volume Claims (PVC).

## Chart Structure

| File/Directory           | Description                                                                           |
|--------------------------|---------------------------------------------------------------------------------------|
| `Chart.yaml`              | Contains metadata about the Helm chart (name, version, application version, etc.).    |
| `README.md`               | Documentation about the Helm chart.                                                   |
| `templates/`              | Directory containing Kubernetes manifest templates.                                   |
| `templates/0-pv.yaml`     | Defines the Persistent Volume (PV) for MongoDB data storage.                          |
| `templates/1-namespace.yaml` | Namespace definition where the application will be deployed.                        |
| `templates/2-configMap.yaml` | ConfigMap for environment variables such as the API port.                          |
| `templates/3-mongo-secret.yaml` | Secret for MongoDB credentials (username, password, database).                  |
| `templates/4-pvc.yaml`    | Persistent Volume Claim (PVC) for MongoDB data storage.                               |
| `templates/5-statefulSet.yaml` | StatefulSet resource for MongoDB deployment.                                     |
| `templates/6-api-deployment.yaml` | Deployment resource for the User Management API.                              |
| `templates/7-api-ingress.yaml` | Ingress resource to expose the API externally via NGINX.                         |
| `templates/_helpers.tpl`  | Helper template for reusable functions.                                               |
| `values.yaml`             | Defines the default configuration values for the Helm chart.                          |

## Key Components

### 1. `Chart.yaml`

This file contains metadata about the Helm chart:

| Field          | Description                                                                 |
|----------------|-----------------------------------------------------------------------------|
| `name`         | Name of the Helm chart (`user-helm-chart`).                                 |
| `version`      | Helm chart version (`0.1.0`).                                                |
| `appVersion`   | Application version (`1.16.0`).                                              |
| `type`         | Specifies the chart type (`application`).                                    |

### 2. `values.yaml`

The `values.yaml` file provides default configurations for the chart. These values can be customized during deployment.

| Parameter                | Description                                                                                 | Default Value                    |
|--------------------------|---------------------------------------------------------------------------------------------|----------------------------------|
| `commonLabel`             | A common label added to all resources.                                                      | `user-management-api`            |
| `namespace.name`          | Namespace where the resources are deployed.                                                 | `dev`                            |
| `extraLabels.environment` | Extra label to define the environment.                                                      | `dev`                            |
| `extraLabels.team`        | Extra label to define the responsible team.                                                 | `backend`                        |
| `deployment.name`         | Name of the API Deployment.                                                                 | `api-deployment`                 |
| `deployment.replicas`     | Number of replicas for the API deployment.                                                  | `2`                              |
| `deployment.container.image` | Docker image for the User Management API.                                                | `panchanandevops/express-ts-api:v3.0.2` |
| `deployment.container.port`  | Port on which the API container listens.                                                 | `3000`                           |
| `apiService.name`         | Name of the API service.                                                                    | `api-service`                    |
| `apiService.type`         | Service type (ClusterIP, NodePort, etc.).                                                   | `ClusterIP`                      |
| `apiService.port`         | Service port for the API.                                                                   | `3000`                           |
| `ingress.enabled`         | Enable/disable the ingress resource.                                                        | `true`                           |
| `ingress.host`            | Host for the Ingress resource.                                                              | `ts-api.com`                     |
| `configmap.name`          | Name of the ConfigMap containing environment variables.                                      | `api-config`                     |
| `secret.name`             | Name of the Secret containing MongoDB credentials.                                           | `mongo-secret`                   |
| `statefulset.name`        | Name of the StatefulSet for MongoDB.                                                         | `mongo`                          |
| `statefulset.container.image` | Docker image for MongoDB.                                                               | `mongo:latest`                   |
| `pv.name`                 | Name of the Persistent Volume (PV) for MongoDB storage.                                      | `mongo-pv`                       |
| `pv.storage`              | Storage size for the Persistent Volume.                                                     | `1Gi`                            |
| `pvc.name`                | Name of the Persistent Volume Claim (PVC).                                                  | `mongo-pvc`                      |

### 3. `Templates Directory`

The `templates` directory contains the Kubernetes manifests for deploying the application:

| Template File             | Description                                                                                 |
|---------------------------|---------------------------------------------------------------------------------------------|
| `0-pv.yaml`                | Defines the Persistent Volume (PV) for MongoDB data storage on the host path `/storage/data`.|
| `1-namespace.yaml`         | Creates the `dev` namespace for the deployment.                                             |
| `2-configMap.yaml`         | Defines environment variables such as the API port using a Kubernetes ConfigMap.            |
| `3-mongo-secret.yaml`      | Defines a Kubernetes Secret to store MongoDB credentials (`username`, `password`, `db`).    |
| `4-pvc.yaml`               | Defines the Persistent Volume Claim (PVC) for MongoDB data storage.                         |
| `5-statefulSet.yaml`       | Deploys MongoDB as a StatefulSet for stable network identities and persistent storage.      |
| `6-api-deployment.yaml`    | Deploys the User Management API as a Kubernetes Deployment, with two replicas.              |
| `7-api-ingress.yaml`       | Defines the Ingress resource to expose the API at `ts-api.com`.                             |
| `_helpers.tpl`             | Contains helper functions to manage labels and other reusable components.                   |

## How to Install the Chart

1. **Install Helm:**

   Ensure you have Helm installed by following the [Helm installation guide](https://helm.sh/docs/intro/install/).

2. **Deploy the Chart:**

   To deploy the chart with the default configuration, run:

   ```bash
   kubectl create namespace dev
   helm install user-management-api ./user-helm-chart --namespace dev
   ```

3. **Customize Values:**

   You can customize the deployment by modifying the `values.yaml` file or by passing custom values during installation:

   ```bash
   helm install user-management-api ./user-helm-chart --namespace dev --set deployment.replicas=3
   ```

4. **Upgrade the Chart:**

   If you make any changes and need to upgrade the existing release, run:

   ```bash
   helm upgrade user-management-api ./user-helm-chart --namespace dev
   ```

5. **Uninstall the Chart:**

   To uninstall the chart and remove all resources:

   ```bash
   helm uninstall user-management-api --namespace dev
   ```

