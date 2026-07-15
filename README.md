# Harness CI Environment Setup

## 1. Install and Configure Docker Desktop

1. Install Docker Desktop.

2. Enable Kubernetes in Docker Desktop.

3. Wait for the Docker Desktop Kubernetes cluster to become available.

4. Verify access to the cluster:

   ```bash
   kubectl cluster-info
   ```

## 2. Create the Harness Environment

1. Sign up for a Harness account.

2. Create the Harness project that will contain the connectors, secrets, delegate, and pipelines.

3. Install and configure the Harness CLI.

4. Follow the repository setup instructions:

   ```text
   https://github.com/allyunion/harness-python-yt-dlp-example
   ```

## 3. Configure External Secrets

1. Install External Secrets Operator in the Docker Desktop Kubernetes cluster using Helm.

2. Create a local fake secret store for development and CI testing.

3. Verify that the secret store is available before creating any Kubernetes resources that depend on it.

## 4. Install Artifact Keeper

1. Install Artifact Keeper in the `artifact-keeper` namespace:

   ```bash
   helm install ak charts/artifact-keeper/ \
     --namespace artifact-keeper \
     --create-namespace \
     --set-string secrets.jwtSecret='ci-test-0123456789-ci-test-0123456789' \
     --set-string postgres.auth.password='ci-test' \
     --set-string backend.env.PEER_API_KEY="$(openssl rand -base64 32 | tr -dc 'a-zA-Z0-9' | head -c 32)" \
     --set-string backend.env.ADMIN_PASSWORD='YOUR-ADMIN-PASSWORD'
   ```

2. Replace `YOUR-ADMIN-PASSWORD` with the Artifact Keeper administrator password to use for the local environment.

3. Verify that the Artifact Keeper workloads are running:

   ```bash
   kubectl -n artifact-keeper get pods
   ```

## 5. Access Artifact Keeper Locally

1. Start port forwarding for the Artifact Keeper backend and web interface:

   ```bash
   bash -c '
   kubectl -n artifact-keeper port-forward svc/ak-artifact-keeper-backend 8080:8080 &
   backend_pid=$!

   kubectl -n artifact-keeper port-forward svc/ak-artifact-keeper-web 3000:3000 &
   web_pid=$!

   trap "kill $backend_pid $web_pid 2>/dev/null" EXIT INT TERM
   wait
   '
   ```

2. Keep this command running while using Artifact Keeper.

3. Open the Artifact Keeper web interface:

   ```text
   http://localhost:3000
   ```

4. Log in using the administrator password supplied through `backend.env.ADMIN_PASSWORD`.

## 6. Configure Artifact Keeper Repositories

1. Create a Docker repository named `docker-images`.

2. Create a PyPI repository named `pypi`.

3. For another programming language, create the corresponding package repository instead of, or in addition to, the `pypi` repository.

4. Create an Artifact Keeper access token for authenticating Docker and package repository operations.

5. Retain the repository URLs and access token for the Harness secret and connector configuration.

## 7. Install the Harness Kubernetes Delegate

1. Install the Kubernetes delegate for Harness in the Docker Desktop Kubernetes cluster.

2. Verify that the delegate connects successfully to Harness before configuring pipeline infrastructure.

## 8. Create the Application Namespace

1. Create the `gunicorn-harness` namespace using the repository manifest:

   ```bash
   kubectl apply -f kubernetes/gunicorn-harness-namespace.yaml
   ```

2. Verify that the namespace exists:

   ```bash
   kubectl get namespace gunicorn-harness
   ```

## 9. Configure Harness Secrets

1. Create the Harness secret used to authenticate Docker operations against Artifact Keeper.

2. Create the Harness secret used to authenticate PyPI operations against Artifact Keeper.

3. Store the Artifact Keeper access token as the secret value.

4. Use the secret identifiers expected by the repository and Harness pipeline configuration.

## 10. Configure Harness Connectors

Create and validate the following Harness connectors:

1. `k8s_docker_desktop_connector`

   Configure this connector to access the Docker Desktop Kubernetes cluster through the installed Harness delegate.

2. `github_harness_repo_connector`

   Configure this connector for the primary Harness example repository.

3. `github_harness_gunicorn_repo_connector`

   Configure this connector for the Gunicorn repository.

4. `github_harness_primary_repo_connector`

   Configure this connector for the primary application repository.

5. Test each connector in Harness and confirm that its connection succeeds before running the pipeline.

