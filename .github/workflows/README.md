
# Build and Push User API Docker Image Workflow

This GitHub Actions workflow automates the process of building, scanning, and pushing the **User API Docker image** to Docker Hub. It also updates Kubernetes and Helm deployment manifests with the newly built Docker image tag. Additionally, a vulnerability scan is performed using Trivy, and the results are uploaded to the GitHub Security tab.

## Workflow Overview

This workflow is triggered on a push of a tagged commit (following the format `v*.*.*` like `v1.0.0`) on changes made to the backend service or GitHub workflow YAML file (`.github/workflows/build-docker-image.yaml`). It performs the following steps:

1. **Checkout the code** from the repository.
2. **Set up Docker Buildx** to enable advanced Docker build capabilities like cache management and multi-platform builds.
3. **Log in to Docker Hub** using credentials stored in GitHub Secrets.
4. **Extract Docker metadata** (tags and labels) from the repository and assign it to the image.
5. **Build and push the Docker image** using Docker Buildx, caching layers to improve build performance.
6. **Run a security scan** on the Docker image using Trivy to detect any HIGH or CRITICAL vulnerabilities.
7. **Upload Trivy results** to the GitHub Security tab in SARIF format for further analysis.
8. **Update Kubernetes manifests and Helm charts** with the new Docker image tag.
9. **Create a pull request** to merge the updated manifests into the main branch.

## Triggering the Workflow

The workflow will trigger automatically on:

- **Tag Pushes**: When you push a tag that follows the format `v*.*.*` (e.g., `v1.0.0`, `v2.1.3`, etc.).
- **File Changes**: When modifications are made to the backend service directory (`backend/**`) or the workflow file itself (`.github/workflows/build-docker-image.yaml`).



## Build and Push Backend Docker Image

This job builds the User API Docker image and pushes it to Docker Hub.

```yaml
jobs:
  build-backend:
    runs-on: ubuntu-latest
    steps:
```

### Checkout Code
Clones the repository so that the workflow can operate on the latest code.

```yaml
- name: Checkout code
  uses: actions/checkout@v4
```

### Set up Docker Buildx
Docker Buildx enables advanced features such as multi-platform builds and caching. It’s essential for building and pushing images efficiently.

```yaml
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3
```

### Log in to Docker Hub
Authenticates with Docker Hub using credentials stored in GitHub Secrets.

```yaml
- name: Log in to Docker Hub
  uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKER_USERNAME }}
    password: ${{ secrets.DOCKER_PASSWORD }}
```

### Extract Docker Metadata
Generates dynamic Docker image tags and labels based on the tag pushed to the repository. This helps in versioning and organizing images.

```yaml
- name: Extract Docker metadata (tags, labels)
  id: meta
  uses: docker/metadata-action@v5
  with:
    images: panchanandevops/express-ts-api
    flavor: |
      latest=false
    tags: |
      type=raw,value={{tag}}
```

### Build and Push Docker Image
Builds the Docker image using `backend/Dockerfile.prod` and pushes it to the Docker Hub repository. It also utilizes caching to speed up the build process.

```yaml
- name: Build and push User API Docker image
  uses: docker/build-push-action@v6
  with:
    context: ./backend
    file: ./backend/Dockerfile.prod
    push: true
    tags: ${{ steps.meta.outputs.tags }}
    labels: ${{ steps.meta.outputs.labels }}
    cache-from: type=registry,ref=panchanandevops/express-ts-api:cache
    cache-to: type=registry,ref=panchanandevops/express-ts-api:cache,mode=max
```

---

## Security Scanning with Trivy

This job scans the built Docker image for vulnerabilities using **Trivy**. It checks for HIGH and CRITICAL vulnerabilities and outputs the results.

### Run Trivy
The following step uses Trivy to scan the Docker image for vulnerabilities related to both operating system packages and application libraries.

```yaml
- name: Run Trivy for HIGH, CRITICAL CVEs and report
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: "${{ steps.meta.outputs.tags }}"
    exit-code: 0 # Do not fail the workflow on vulnerabilities
    ignore-unfixed: true
    vuln-type: "os,library"
    severity: "HIGH,CRITICAL"
    format: "sarif"
    output: "trivy-results.sarif"
```

### Upload Trivy Scan Results
After scanning, the results are uploaded to the GitHub Security tab in SARIF format for easy review.

```yaml
- name: Upload Trivy scan results to GitHub Security tab
  uses: github/codeql-action/upload-sarif@v3
  if: always()
  with:
    sarif_file: "trivy-results.sarif"
```

---

## Update Deployment Manifests

This job updates the image tag in your Kubernetes manifests and Helm chart files with the newly built Docker image tag.

### Create a New Branch
A new branch is created to hold the changes related to the updated image tag.

```yaml
- name: Create a new branch
  run: |
    git checkout -b main   
    git pull origin main
    git checkout -b update-deployment-image-${{ github.sha }}
    git push --set-upstream origin update-deployment-image-${{ github.sha }}
```

### Update Image Tags in YAML Files
This step updates the Docker image tag in the Kubernetes and Helm deployment files.

```yaml
- name: Update deployment image in YAML
  run: |
    sed -i "s|image: panchanandevops/express-ts-api:.*|image: ${{ steps.meta.outputs.tags }}|g" Deploy/k8s/manifests/6-api-deployment.yaml
    sed -i "s|image: panchanandevops/express-ts-api:.*|image: ${{ steps.meta.outputs.tags }}|g" Deploy/kustomize/base/6-api-deployment.yaml
    sed -i "s|image: panchanandevops/express-ts-api:.*|image: ${{ steps.meta.outputs.tags }}|g" Deploy/user-helm-chart/values.yaml
```

---

## Create a Pull Request

Once the image tag is updated, the workflow automatically commits the changes and opens a pull request for review.

### Commit the Changes
This step commits the updated files to the newly created branch.

```yaml
- name: Commit changes
  run: |
    git add Deploy/k8s/manifests/6-api-deployment.yaml Deploy/kustomize/base/6-api-deployment.yaml Deploy/user-helm-chart/values.yaml trivy-results.sarif 
    git commit -m "Update tag in Deployment image tag and add Trivy scan results"
```

### Open a Pull Request
Creates a pull request from the updated branch to the `main` branch.

```yaml
- name: Create Pull Request
  uses: peter-evans/create-pull-request@v6
  with:
    token: ${{ secrets.PAT_TOKEN }}
    branch: update-deployment-image-${{ github.sha }}
    base: main
    title: "Update User API image tag to ${{ steps.meta.outputs.tags }}"
    body: "This PR updates the User API image tag to ${{ steps.meta.outputs.tags }}."
```

---

## Secrets and Environment Variables

To ensure this workflow functions smoothly, the following secrets must be configured in your GitHub repository:

- **DOCKER_USERNAME**: Your Docker Hub username.
- **DOCKER_PASSWORD**: Your Docker Hub password or personal access token.
- **PAT_TOKEN**: GitHub personal access token with `repo` and `pull-request` permissions.



## How to Customize

You can adjust the following parameters according to your project needs:
- **Docker Image Tags**: Modify the `tags` parameter in the `docker/metadata-action` step to change how image tags are generated.
- **Kubernetes and Helm Chart Paths**: Ensure that your deployment manifest paths match the paths in the `Update deployment image in YAML` step.

