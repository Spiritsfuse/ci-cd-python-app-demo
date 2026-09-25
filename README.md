# ⚡ hey-cicd — DevSecOps Dashboard

## 📁 Project Structure

```
hey-cicd/
├── app/
│   ├── app.py              # Flask application
│   ├── templates/
│   │   └── index.html      # Dashboard UI
│   └── static/
│       ├── css/styles.css
│       └── js/main.js
├── tests/
│   └── test_app.py         # Unit tests
├── k8s/
│   ├── deployment.yaml     # Kubernetes Deployment
│   └── service.yaml        # Kubernetes Service
├── .github/
│   └── workflows/
│       └── devsecops.yml   # CI/CD Pipeline
├── Dockerfile
├── requirements.txt
├── requirements-dev.txt
└── README.md
```

> 💡 **[cmd-explained] Core Jargon & Concepts Quick-Glossary:**
> - **CI/CD**: Continuous Integration (automating build & test on every commit) & Continuous Delivery/Deployment (automating release/deploy to environments).
> - **DevSecOps**: Shifting security "left" into developer workflows by automating vulnerability scans (SAST, SCA, container checks) directly inside the CI/CD pipeline.
> - **Virtual Environment (`venv`)**: An isolated directory sandbox with its own Python binary and packages, preventing conflicting library versions across projects.
> - **Container (Docker)**: A lightweight, standalone package bundling code, runtime, system tools, and libraries to run identically in any environment.
> - **Orchestration (Kubernetes / k8s)**: An automated platform for deploying, scaling, self-healing, and networking containerized applications across host machines.

---

## 🌐 API Endpoints

| Method | Route | Description |
|--------|-------|-------------|
| `GET` | `/` | Dashboard UI |
| `GET` | `/health` | Health check |
| `GET` | `/api/status` | App info, uptime, Python version |
| `GET` | `/api/greet/<name>` | Returns a greeting for the name |
| `POST` | `/api/add` | Adds two numbers |
| `POST` | `/api/calculate` | Calculator (add/subtract/multiply/divide/power/modulo) |
| `POST` | `/api/pipeline/run` | Simulates a CI/CD pipeline run |

---

## 🖥️ Method 1 — Run Manually (Python)

### Step 1 — Clone the repository

```bash
git clone https://github.com/YOUR_USERNAME/hey-cicd.git
cd hey-cicd
```

> 💡 **[cmd-explained]**
> - `git clone <url>`: Downloads the remote Git repository and its full commit history into a new local directory.
> - `cd hey-cicd`: (*Change Directory*) Navigates your terminal shell into the cloned repository folder.

### Step 2 — Create a virtual environment

```bash
python3 -m venv .venv
source .venv/bin/activate        # Mac/Linux
# .venv\Scripts\activate         # Windows
```

> 💡 **[cmd-explained]**
> - `python3 -m venv .venv`:
>   - `python3`: Runs the Python 3 interpreter.
>   - `-m venv`: Flag `-m` executes a standard library module (`venv`) as a script.
>   - `.venv`: The name of the target directory created to store isolated Python binaries and libraries.
> - `source .venv/bin/activate` (Mac/Linux) or `.venv\Scripts\activate` (Windows):
>   - Modifies current shell's `$PATH` so `python` and `pip` point to `.venv` instead of system-wide installations.

### Step 3 — Install dependencies

```bash
pip install -r requirements.txt
```

> 💡 **[cmd-explained]**
> - `pip install -r requirements.txt`:
>   - `pip`: Python's official package management tool.
>   - `install`: Subcommand instructing pip to download and install packages.
>   - `-r <file>`: Reads package specifications from a text file to install production dependencies in batch.
> - `pip list`: *(Helpful check)* Prints all installed packages and their current version numbers in the active environment.

### Step 4 — Run the app

```bash
python3 app/app.py
```

> 💡 **[cmd-explained]**
> - `python3 app/app.py`: Executes the Flask application entrypoint script, launching a local development web server listening on port `5001`.

### Step 5 — Open in browser

```
http://localhost:5001
```

> 💡 **[cmd-explained]**
> - `localhost` (`127.0.0.1`): Loopback network address resolving to your own machine.
> - `:5001`: Port number assigned in `app.py` for routing incoming HTTP requests to the Flask server.

### Step 6 — Run the tests

```bash
pip install -r requirements-dev.txt
python3 -m pytest --cov=app --cov-report=term-missing
```

> 💡 **[cmd-explained]**
> - `pip install -r requirements-dev.txt`: Installs developer/testing tooling (`pytest` test runner and `pytest-cov` coverage plugin).
> - `python3 -m pytest`: Runs pytest via Python module flag to guarantee execution within the active virtual environment.
> - `--cov=app`: Measures test execution coverage across all Python code inside the `app/` directory.
> - `--cov-report=term-missing`: Prints coverage table directly to the terminal, showing overall percentage and listing line numbers not tested.

**Expected output:**
```
tests/test_app.py::test_home                      PASSED
tests/test_app.py::test_health                    PASSED
tests/test_app.py::test_greet                     PASSED
tests/test_app.py::test_add_numbers               PASSED
tests/test_app.py::test_add_numbers_missing_fields PASSED
tests/test_app.py::test_calculator_multiply       PASSED
tests/test_app.py::test_calculator_divide_by_zero PASSED
tests/test_app.py::test_status                    PASSED
8 passed in 0.Xs
```

### Test the API manually

```bash
# Health check
curl http://localhost:5001/health

# Greet someone
curl http://localhost:5001/api/greet/Nensi

# Add two numbers
curl -X POST http://localhost:5001/api/add \
  -H "Content-Type: application/json" \
  -d '{"number1": 10, "number2": 20}'

# Calculator
curl -X POST http://localhost:5001/api/calculate \
  -H "Content-Type: application/json" \
  -d '{"a": 6, "b": 3, "operation": "multiply"}'
```

> 💡 **[cmd-explained] curl flags & HTTP fields:**
> - `curl`: Command-line utility to transfer data to or from a network server using protocols like HTTP/HTTPS.
> - `-X POST`: Sets the HTTP request method to `POST` (submitting data to the endpoint). Default is `GET`.
> - `-H "Content-Type: application/json"`: Adds an HTTP header indicating the request body payload is JSON.
> - `-d '<data>'`: Passes data payload in the body of the HTTP request.

---

## 🐳 Method 2 — Run with Docker

### Prerequisites
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running

### Step 1 — Build the Docker image

```bash
docker build -t hey-cicd:latest .
```

> 💡 **[cmd-explained] Docker build command & Dockerfile fields:**
> - `docker build`: Assembles an immutable container image from instructions specified in `Dockerfile`.
> - `-t hey-cicd:latest`: Flags image tag (`-t`) with `repository_name:tag_version` format.
> - `.`: Specifies build context (current directory containing files sent to Docker daemon).
>
> **Dockerfile Directives Breakdown:**
> - `FROM python:3.12-slim`: Base OS and Python image layer; `slim` provides a minimal Debian environment to keep image size small.
> - `WORKDIR /app`: Defines container working directory; subsequent instructions run relative to this path.
> - `COPY <src> <dest>`: Copies files from local host into container filesystem.
> - `RUN <cmd>`: Executes commands during image build step (e.g. installing dependencies).
> - `EXPOSE 5001`: Documentation field noting container listens on port 5001 at runtime.
> - `CMD ["python", "app/app.py"]`: Default executable command run when the container starts.

### Step 2 — Run the container

```bash
docker run -p 5001:5001 hey-cicd:latest
```

> 💡 **[cmd-explained]**
> - `docker run`: Creates a writable container layer over the specified image and starts it.
> - `-p 5001:5001`: Port publishing flag (`-p <host_port>:<container_port>`). Forwards requests on host machine port 5001 to container internal port 5001.

### Step 3 — Open in browser

```
http://localhost:5001
```

### Useful Docker commands

```bash
# See running containers
docker ps

# Stop the container
docker stop <container-id>

# Remove the image
docker rmi hey-cicd:latest

# Run in background (detached mode)
docker run -d -p 5001:5001 hey-cicd:latest
```

> 💡 **[cmd-explained]**
> - `docker ps`: Lists running containers with Container ID, image name, status, and port bindings.
> - `docker stop <id>`: Gracefully stops running container by sending SIGTERM signal.
> - `docker rmi <image>`: (*Remove Image*) Deletes the local container image from storage.
> - `-d`: (*Detached mode*) Runs container in background as a daemon process and outputs container ID, freeing up terminal.

### Step 4 — Tag & Push to Docker Hub (Registry)

```bash
# 1. Authenticate with Docker Hub
docker login

# 2. Tag local image with your Docker Hub username and repository
docker tag python-web:latest spiritsfuse/python-web:latest

# 3. Push the image to Docker Hub
docker push spiritsfuse/python-web:latest
```

> 💡 **[cmd-explained] Docker Hub Publishing Flow:**
> - `docker login`: Authenticates your CLI session with Docker Hub using your credentials.
> - `docker tag <source> <target>`: Creates an alias pointing to the existing image layers under the `<username>/<repo>:<tag>` format needed for remote upload.
> - `docker push <target>`: Uploads the container image layers and manifest to the Docker Hub registry so Kubernetes or remote servers can pull it.

---

## ⚙️ Method 3 — CI/CD Pipeline (GitHub Actions)

The pipeline runs automatically every time you push code to `main` or open a pull request.

### Pipeline Stages

```
Push to GitHub
      │
      ▼
┌─────────────────┐
│  STEP 1: Tests  │  pytest — runs all 8 unit tests
└────────┬────────┘
         │
┌────────▼────────┐
│  STEP 2: SAST   │  CodeQL — scans code for security issues
└────────┬────────┘
         │
┌────────▼────────┐
│   STEP 3: SCA   │  pip-audit — checks for vulnerable packages
└────────┬────────┘
         │ (all 3 must pass)
┌────────▼────────┐
│  STEP 4: Build  │  docker build — creates the Docker image
└────────┬────────┘
         │
┌────────▼────────┐
│  STEP 5: Scan   │  Trivy — scans the Docker image for CVEs
└────────┬────────┘
         │
┌────────▼────────┐
│  STEP 6: Push   │  Pushes image to GitHub Container Registry
└────────┬────────┘
         │ (only on push to main)
┌────────▼────────┐
│ STEP 7: Deploy  │  kubectl apply → deploys to Kubernetes
└─────────────────┘
```

> 💡 **[cmd-explained] DevSecOps Stages & Jargon:**
> - **SAST (Static Application Security Testing — CodeQL)**: Scans source code without executing it to detect security flaws (e.g., OWASP top 10, SQLi, insecure deserialization).
> - **SCA (Software Composition Analysis — pip-audit)**: Scans open-source third-party dependencies against CVE databases for known vulnerabilities.
> - **CVE (Common Vulnerabilities and Exposures)**: Standardized public identifier for known cybersecurity vulnerabilities.
> - **Image Scan (Trivy)**: Scans compiled container image layers for OS package and language library CVEs.
> - **GHCR / Docker Hub**: Container registries serving as secure repositories to store, version, and distribute container images.

### How to trigger the pipeline

```bash
# Make a change, commit, and push
git add .
git commit -m "your message"
git push origin main
```

> 💡 **[cmd-explained] Git commands:**
> - `git add .`: Stages all changes (modified, added, deleted files) in current directory for the upcoming commit.
> - `git commit -m "message"`: Permanently saves staged changes to local commit history with an informative commit message.
> - `git push origin main`: Transmits committed changes from local `main` branch to remote repository (`origin`).

Then go to your GitHub repo → **Actions** tab to watch it run.

### Required GitHub Secrets

Go to **GitHub repo → Settings → Secrets and variables → Actions** and add:

| Secret Name | Value |
|-------------|-------|
| `KUBECONFIG` | Contents of your `~/.kube/config` file (needed for Step 7 deploy) |

> ℹ️ `GITHUB_TOKEN` is automatically provided by GitHub — you don't need to add it manually.

> 💡 **[cmd-explained] GitHub Actions Workflow Fields (.github/workflows/devsecops.yml):**
> - `name`: Name of workflow displayed in GitHub Actions tab.
> - `on`: Event triggers that execute workflow (e.g., `push` or `pull_request` on `main`).
> - `jobs`: Defines distinct operational units running in parallel or sequentially.
> - `runs-on: ubuntu-latest`: The virtual machine operating system running the job runner.
> - `needs`: Specifies dependency on prior jobs; prevents execution until required jobs succeed.
> - `steps`: Ordered list of tasks executed in sequence on the runner.
> - `uses`: References reusable action from GitHub Marketplace (e.g., `actions/checkout@v4`).
> - `with`: Input parameter dictionary passed into an action.
> - `run`: Shell commands executed directly in runner terminal.
> - `permissions`: Fine-grained token access controls (e.g., `security-events: write` for CodeQL alerts).
> - `if`: Conditional guard controlling whether a job or step runs (e.g., only deploy on `push` to `main`).

### View your Docker image after push

After Step 6 runs, your image is available at:
```
ghcr.io/YOUR_USERNAME/hey-cicd:latest
```

Go to **GitHub repo → Packages** to see it.

---

## ☸️ Method 4 — Deploy to Kubernetes manually

> Do this if you want to deploy without the pipeline, directly from your terminal.

### Prerequisites
- A running Kubernetes cluster (minikube, k3s, or cloud)
- `kubectl` installed and connected to your cluster

> 💡 **[cmd-explained] Docker Desktop vs Docker Hub vs Minikube — Where does the image live?**
> - **Docker Desktop (Local Host Cache)**: When you run `docker build -t my-app .`, the image exists **only on your local PC**. External clusters and isolated VMs cannot reach it.
> - **Docker Hub (Remote Cloud Registry)**: A public/private online image registry (like GitHub for Docker images). Any Kubernetes cluster with internet access can pull images from `docker.io/<username>/<repo>:<tag>`.
> - **Minikube (Isolated Local Cluster)**: Minikube runs inside its own virtual machine or container with its **own separate Docker/containerd engine**. It **cannot see** Docker Desktop's local images by default!
>
> **Two ways to make your image available to Kubernetes:**
>
> 🔹 **Approach A — Push to Docker Hub (Standard for remote/cloud clusters):**
> ```bash
> # 1. Tag local image with your Docker Hub username
> docker tag <local-image>:<tag> <dockerhub-username>/<repo-name>:<tag>
> # Example for user spiritsfuse:
> docker tag python-web:latest spiritsfuse/python-web:latest
>
> # 2. Upload image to Docker Hub
> docker push spiritsfuse/python-web:latest
> ```
> *In `k8s/deployment.yaml`: set `image: spiritsfuse/python-web:latest` and keep `imagePullPolicy: Always`.*
>
> 🔹 **Approach B — Load directly into Minikube (Offline / No Docker Hub needed):**
> ```bash
> # 1. Build locally
> docker build -t hey-cicd:latest .
>
> # 2. Inject image directly from host Docker into Minikube's internal VM
> minikube image load hey-cicd:latest
> ```
> *In `k8s/deployment.yaml`: set `image: hey-cicd:latest` and change `imagePullPolicy: IfNotPresent` or `Never` (otherwise K8s will attempt to query Docker Hub and fail).*

### Step 1 — Apply the manifests

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

> 💡 **[cmd-explained] kubectl apply & Kubernetes Manifest Fields:**
> - `kubectl`: Command-line tool controlling Kubernetes cluster through its REST API.
> - `apply`: Declarative configuration command applying desired state configurations.
> - `-f <file>`: Specifies path to YAML manifest file describing target resource.
>
> **Manifest Fields Breakdown (`deployment.yaml` & `service.yaml`):**
> - `apiVersion`: Version of Kubernetes API resource schema (`apps/v1` for Deployment, `v1` for Service).
> - `kind`: Resource type (`Deployment` manages Pod replicas and rolling updates; `Service` manages network routing).
> - `metadata.name`: Unique identifier string for the resource.
> - `spec.replicas`: Number of identical Pod instances maintained by controller.
> - `spec.selector.matchLabels`: Label selector identifying which Pods belong to this Deployment.
> - `spec.template`: Blueprint template used by Deployment to instantiate new Pods.
> - `spec.template.spec.containers`: Container specifications inside Pod (`name`, `image`, `imagePullPolicy`, `ports.containerPort`).
> - `imagePullPolicy: Always`: Ensures kubelet always checks registry and pulls latest image before starting container.
> - `spec.type: NodePort` (Service): Exposes the service outside the cluster on each node's static port (`nodePort`).
> - `ports.port`: Internal port exposed by Service within cluster (port 80).
> - `ports.targetPort`: Port on container Pod where traffic is forwarded (port 5001).
> - `ports.nodePort`: External static port assigned on every cluster Node (port 30001).

### Step 2 — Check the pods are running

```bash
kubectl get pods
kubectl get service session17-python
```

> 💡 **[cmd-explained]**
> - `kubectl get pods`: Queries and displays status of Pods (`Running`, `Pending`, `CrashLoopBackOff`) and restart count.
> - `kubectl get service <name>`: Retrieves service networking details, ClusterIP, NodePort mapping, and active ports.

### Step 3 — Access the app

```bash
# If using minikube
minikube service session17-python

# Or access via NodePort
http://<your-node-ip>:30001
```

> 💡 **[cmd-explained]**
> - `minikube service <name>`: Automatically discovers Minikube VM IP, exposes port, and opens browser to service.
> - `<your-node-ip>:30001`: Direct external access via Node IP on configured `nodePort` (30001).

### Useful kubectl commands

```bash
# See all running pods
kubectl get pods

# See logs from a pod
kubectl logs <pod-name>

# Delete the deployment
kubectl delete -f k8s/deployment.yaml
kubectl delete -f k8s/service.yaml
```

> 💡 **[cmd-explained]**
> - `kubectl logs <pod-name>`: Fetches and displays stdout/stderr output from a container in a Pod (key for debugging runtime errors).
> - `kubectl delete -f <file>`: Gracefully destroys and removes resources defined in the specified manifest file.

---

## 🧪 DevSecOps Concepts Covered

| Concept | Tool Used | Where |
|---------|-----------|-------|
| **Unit Testing** | pytest + pytest-cov | `tests/test_app.py` |
| **SAST** (Static Application Security Testing) | GitHub CodeQL | Pipeline Step 2 |
| **SCA** (Software Composition Analysis) | pip-audit | Pipeline Step 3 |
| **Containerisation** | Docker | `Dockerfile` |
| **Container Image Scanning** | Trivy | Pipeline Step 5 |
| **Container Registry** | GitHub Container Registry (GHCR) | Pipeline Step 6 |
| **Orchestration** | Kubernetes | `k8s/` folder |
| **CI/CD Automation** | GitHub Actions | `.github/workflows/devsecops.yml` |

---

## 👩‍💻 Built With

- **Python 3.12** + **Flask 3.x**
- **Docker**
- **Kubernetes**
- **GitHub Actions**