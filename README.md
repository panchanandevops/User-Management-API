# User Management API 

The **User Management API** allows for seamless CRUD operations to manage user data with powerful observability, a streamlined CI/CD pipeline using GitHub Actions, and robust deployments through Argo CD. Below are instructions for interacting with the API, deploying it on Kubernetes, and ensuring continuous integration and monitoring.

---

## API Endpoints and `curl` Commands

### 1. **POST**: Create New Users

- **Endpoint**: `POST /users`
- **Description**: Adds a new user with name, email, and password.

#### Examples:
```sh
# Create user "John Doe"
curl -X POST http://ts-api.com/users \
-H "Content-Type: application/json" \
-d '{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "password123"
}'

# Create user "Jane Doe"
curl -X POST http://ts-api.com/users \
-H "Content-Type: application/json" \
-d '{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "password": "password456"
}'

# Create user "Alice Smith"
curl -X POST http://ts-api.com/users \
-H "Content-Type: application/json" \
-d '{
  "name": "Alice Smith",
  "email": "alice@example.com",
  "password": "password789"
}'
```

### 2. **GET**: Retrieve Users

- **Endpoint**: `GET /users`
- **Description**: Retrieves all users.

```sh
curl -X GET http://ts-api.com/users
```

#### Get a User by ID
- Replace `USER_ID` with the actual user ID.
```sh
curl -X GET http://ts-api.com/users/66c63abdda7164136f89c956
```

### 3. **PUT**: Update User Information

- **Endpoint**: `PUT /users/:id`
- **Description**: Updates a specific user's details based on user ID.

```sh
curl -X PUT http://ts-api.com/users/66c63abdda7164136f89c956 \
-H "Content-Type: application/json" \
-d '{
  "name": "Updated Name",
  "email": "updated@example.com",
  "password": "newpassword123"
}'
```

### 4. **DELETE**: Remove a User

- **Endpoint**: `DELETE /users/:id`
- **Description**: Deletes a specific user by ID.

```sh
curl -X DELETE http://ts-api.com/users/66c63abdda7164136f89c956
```

---

## CI/CD Pipeline with GitHub Actions

The pipeline automates building, testing, and deploying the Dockerized API to Docker Hub and Kubernetes.

### Workflow Overview: `build-docker-image.yaml`

- **Triggers**: Runs on tag pushes (e.g., `v1.0.0`) and changes to relevant directories.
- **Workflow Steps**:
  1. **Checkout & Setup**: Pulls the code and configures Docker Buildx.
  2. **Build & Push Docker Image**: Builds the Go API image, tags it, and pushes it to Docker Hub.
  3. **Update Deployment YAML**: Edits the Kubernetes deployment YAML to use the new image tag.
  4. **Create Pull Request**: Opens a PR to review and merge the updated image tag.

### Required GitHub Secrets

- `DOCKER_USERNAME`: Docker Hub username.
- `DOCKER_PASSWORD`: Docker Hub password.
- `PAT_TOKEN`: GitHub Personal Access Token to create pull requests.

---

## Deployment Using Argo CD

Argo CD offers three different approaches for managing application deployment to Kubernetes:

### Argo CD Applications

1. **Helm-based Deployment**:
   - **Config File**: `helm.yaml`
   - **Source**: `Deploy/user-helm-chart/`
   - **Destination**: `dev` namespace
   - **Sync Policy**: Automated self-healing and pruning

2. **Kubernetes Manifest Deployment**:
   - **Config File**: `k8s-manifests.yaml`
   - **Source**: `Deploy/k8s/manifests`
   - **Sync Policy**: Self-healing and pruning enabled

3. **Kustomize-based Deployment**:
   - **Config File**: `kustomize.yaml`
   - **Source**: `Deploy/kustomize/overlays/prod`
   - **Sync Policy**: Automatic sync with self-healing and pruning

### Deploying with Argo CD
1. Clone the repository and apply the Argo CD configurations:
   ```bash
   git clone https://github.com/panchanandevops/User-Management-API.git
   kubectl apply -f path/to/argo-application.yaml
   ```



---

## Observability and Monitoring Stack

This API application is equipped with observability tools using **Grafana**, **Prometheus**, and **Loki** to monitor, collect, and analyze data.

- **Grafana**: Configured to visualize time-series metrics and system health, with dashboard alerts for anomaly detection.
- **Prometheus**: Aggregates and stores time-series data from the application, enabling historical analysis and critical alerts.
- **Loki**: Manages log aggregation, allowing for efficient querying and debugging.

This setup ensures application stability, enabling quick diagnosis of potential issues and a proactive approach to monitoring.

