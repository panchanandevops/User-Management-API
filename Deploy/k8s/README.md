# Kubernetes Deployment for User Management API

This repository contains the Kubernetes manifests needed to deploy a **User Management API** with a MongoDB backend. The setup includes the required Persistent Volumes, Namespace, ConfigMap, Secrets, StatefulSets, Deployments, Services, and Ingress resources.

## Folder Structure

```
k8s
├── manifests
│   ├── 0-pv.yaml
│   ├── 1-namespace.yaml
│   ├── 2-configMap.yaml
│   ├── 3-mongo-secret.yaml
│   ├── 4-pvc.yaml
│   ├── 5-statefulSet.yaml
│   ├── 6-api-deployment.yaml
│   ├── 7-api-ingress.yaml
└── README.md
```

## Resources Breakdown

### 1. Persistent Volume (`0-pv.yaml`)

This file defines a **Persistent Volume** for MongoDB to store its data persistently using the host's storage.

- **Storage**: 1Gi
- **Access Mode**: `ReadWriteOnce`
- **Path**: `/storage/data` on the host

### 2. Namespace (`1-namespace.yaml`)

This manifest defines the `dev` namespace where all Kubernetes resources for this deployment will reside.

### 3. ConfigMap (`2-configMap.yaml`)

Defines a ConfigMap named `api-config` which stores configuration data for the API, such as the port number (`3000`).

### 4. MongoDB Secret (`3-mongo-secret.yaml`)

The Secret resource contains sensitive information for the MongoDB connection, including the root username, password, and the initial database.

### 5. Persistent Volume Claim (`4-pvc.yaml`)

Defines the **Persistent Volume Claim** for MongoDB to bind with the Persistent Volume and manage storage requests.

- **Storage**: 1Gi
- **Access Mode**: `ReadWriteOnce`

### 6. MongoDB StatefulSet (`5-statefulSet.yaml`)

The StatefulSet manages the MongoDB pod and ensures stable and ordered deployment and scaling.

- **Image**: `mongo:latest`
- **Replicas**: 1
- **Volume**: Mounted on `/data/db` from the Persistent Volume Claim

### 7. API Deployment (`6-api-deployment.yaml`)

Defines the deployment of the **User Management API**:

- **Image**: `panchanandevops/express-ts-api:v4.0.0`
- **Port**: 3000
- **Replicas**: 1
- **Resources**:
  - Requests: 64Mi Memory, 50m CPU
  - Limits: 128Mi Memory, 100m CPU

### 8. Ingress Resource (`7-api-ingress.yaml`)

This file sets up an Ingress resource to expose the API externally.

- **Ingress Class**: `nginx`
- **Host**: `ts-api.com`
- **Service**: Points to `api-service` on port `3000`

## Deployment Instructions

### Prerequisites

- **Kubernetes Cluster**: Ensure you have access to a Kubernetes cluster.
- **kubectl**: Make sure `kubectl` is configured to interact with your cluster.
- **Nginx Ingress Controller**: An Ingress Controller (such as Nginx) should be installed.

### Steps to Deploy

1. **Clone the repository**:

   ```bash
   git clone https://github.com/panchanandevops/user-management-api-k8s.git
   cd user-management-api-k8s/k8s/manifests
   ```

2. **Create the Namespace**:

   ```bash
   kubectl apply -f 1-namespace.yaml
   ```

3. **Create the Persistent Volume**:

   ```bash
   kubectl apply -f 0-pv.yaml
   ```

4. **Create the ConfigMap and Secret**:

   ```bash
   kubectl apply -f 2-configMap.yaml
   kubectl apply -f 3-mongo-secret.yaml
   ```

5. **Create the Persistent Volume Claim**:

   ```bash
   kubectl apply -f 4-pvc.yaml
   ```

6. **Deploy MongoDB StatefulSet and Service**:

   ```bash
   kubectl apply -f 5-statefulSet.yaml
   ```
7. **Deploy User deployment and Service**:
   ```bash
   kubectl apply -f 6-api-deployment.yaml
   ```

8. **Deploy the Ingress Resource**:
   ```bash
   kubectl apply -f 7-api-ingress.yaml
   ```

### Accessing the API

Once the deployment is complete and the Ingress resource is created, you can access the API via the domain `ts-api.com` (ensure it is configured to point to your Ingress IP).


### Verification

You can monitor the status of the resources using the following commands:

- Check the pods:

  ```bash
  kubectl get pods -n dev
  ```

- Check services:

  ```bash
  kubectl get svc -n dev
  ```

- Check the Ingress resource:
  ```bash
  kubectl get ingress -n dev
  ```



## Cleanup

To delete all resources, run:

```bash
kubectl delete -f .
```


