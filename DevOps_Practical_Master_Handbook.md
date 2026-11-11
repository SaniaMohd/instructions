v# DevOps Practical Master Handbook

**12 independent topics consolidated from Sets 1--6.** Commands are for
Windows PowerShell unless stated otherwise. Run each topic in its own
folder. Replace `USERNAME`, repository URLs and email placeholders.

## Common setup

Install Python 3.11+, Git, Docker Desktop, kubectl, Jenkins, Terraform
and Helm as needed. Check:

``` powershell
python --version
git --version
docker --version
kubectl version --client
terraform --version
helm version
```

Python virtual environment:

``` powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
# If activation is blocked:
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\.venv\Scripts\Activate.ps1
```

------------------------------------------------------------------------

# 1. Python Backend Development (Flask)

**Install**

``` powershell
mkdir Topic01-Flask; cd Topic01-Flask
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install flask
```

**`app.py`**

``` python
from flask import Flask, jsonify, request
app = Flask(__name__)

students = [{"id": 1, "name": "Rahul"}, {"id": 2, "name": "Priya"}]

@app.get("/")
def home():
    return "Flask backend is running!"

@app.get("/students")
def get_students():
    return jsonify(students)

@app.post("/students")
def add_student():
    data = request.get_json(silent=True) or {}
    if not data.get("name"):
        return jsonify({"error": "name is required"}), 400
    student = {"id": max([s["id"] for s in students], default=0) + 1,
               "name": data["name"]}
    students.append(student)
    return jsonify(student), 201

@app.put("/students/<int:student_id>")
def update_student(student_id):
    data = request.get_json(silent=True) or {}
    for student in students:
        if student["id"] == student_id:
            student["name"] = data.get("name", student["name"])
            return jsonify(student)
    return jsonify({"error": "student not found"}), 404

@app.delete("/students/<int:student_id>")
def delete_student(student_id):
    for student in students:
        if student["id"] == student_id:
            students.remove(student)
            return jsonify({"message": "deleted"})
    return jsonify({"error": "student not found"}), 404

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=True)
```

**Run**

``` powershell
python app.py
```

Test in browser: `http://localhost:5000/` and `/students`. Postman: -
GET `http://localhost:5000/students` - POST same URL, Body → raw → JSON:
`{"name":"Sania"}` - PUT `http://localhost:5000/students/1`, JSON:
`{"name":"Asha"}` - DELETE `http://localhost:5000/students/1`

*Data is in memory and resets when the server restarts.*

------------------------------------------------------------------------

# 2. Git and GitHub

Configure once:

``` powershell
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

Practical:

``` powershell
mkdir Topic02-Git; cd Topic02-Git
"# Git practical" | Out-File -Encoding utf8 README.md
git init
git status
git add .
git commit -m "Initial commit"
git branch -M main
git switch -c feature
"Feature work" | Out-File -Encoding utf8 feature.txt
git add .
git commit -m "Add feature"
git switch main
git merge feature
git log --oneline
```

Create an **empty** GitHub repository, then:

``` powershell
git remote add origin https://github.com/USERNAME/REPOSITORY.git
git remote -v
git push -u origin main
```

Other commands:

``` powershell
git clone https://github.com/USERNAME/REPOSITORY.git
git fetch
git pull origin main
git diff
git branch
git switch main
git branch -d feature
```

`.gitignore`:

``` gitignore
.venv/
__pycache__/
*.pyc
.env
.pytest_cache/
```

For merge conflicts, edit the conflicted file, remove conflict markers,
then:

``` powershell
git add .
git commit -m "Resolve merge conflict"
```

------------------------------------------------------------------------

# 3. Docker Fundamentals

Ensure Docker Desktop is running:

``` powershell
docker version
docker info
docker run hello-world
```

Create `index.html`:

``` html
<!doctype html>
<html><head><title>Docker Practical</title></head>
<body><h1>Hello from Docker container!</h1></body></html>
```

Create `Dockerfile`:

``` dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

Build and run:

``` powershell
docker build -t my-nginx:v1 .
docker images
docker run -d --name web-container -p 8080:80 my-nginx:v1
docker ps
```

Open `http://localhost:8080`. Manage:

``` powershell
docker logs web-container
docker inspect web-container
docker exec -it web-container sh
docker stop web-container
docker start web-container
docker restart web-container
docker rm -f web-container
docker rmi my-nginx:v1
```

`-p HOST_PORT:CONTAINER_PORT`; `-d` runs in background.

------------------------------------------------------------------------

# 4. Docker with Python Flask

`app.py`:

``` python
from flask import Flask, jsonify
app = Flask(__name__)

@app.get("/")
def home():
    return "Flask running inside Docker!"

@app.get("/health")
def health():
    return jsonify(status="healthy")

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

`requirements.txt`:

``` text
flask
```

`Dockerfile`:

``` dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
EXPOSE 5000
CMD ["python", "app.py"]
```

Run:

``` powershell
docker build -t flask-demo:v1 .
docker run -d --name flask-demo-container -p 5000:5000 flask-demo:v1
docker ps
docker logs flask-demo-container
```

Test `http://localhost:5000/` and `/health`. Cleanup:
`docker rm -f flask-demo-container`. Flask must bind to `0.0.0.0` inside
the container.

------------------------------------------------------------------------

# 5. Kubernetes Deployments and Services

Enable Kubernetes in Docker Desktop and wait for it to be ready. Check:

``` powershell
kubectl version --client
kubectl cluster-info
kubectl get nodes
```

`deployment.yaml`:

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80
```

`service.yaml`:

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: ClusterIP
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
```

Apply and inspect:

``` powershell
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl get deployments
kubectl get pods -o wide
kubectl get services
kubectl describe deployment web-deployment
kubectl get all
```

Access (keep terminal open):

``` powershell
kubectl port-forward service/web-service 8080:80
```

Open `http://localhost:8080`. Cleanup:

``` powershell
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
```

------------------------------------------------------------------------

# 6. Kubernetes ConfigMaps and Secrets

## ConfigMap

`configmap.yaml`:

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_MESSAGE: "Hello from ConfigMap"
  APP_MODE: "practical"
```

`configmap-deployment.yaml`:

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: config-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: config-demo
  template:
    metadata:
      labels:
        app: config-demo
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          envFrom:
            - configMapRef:
                name: app-config
```

Run:

``` powershell
kubectl apply -f configmap.yaml
kubectl apply -f configmap-deployment.yaml
kubectl get configmaps
kubectl describe configmap app-config
kubectl get pods
```

Verify (replace `POD_NAME` with actual name):

``` powershell
kubectl exec POD_NAME -- printenv APP_MESSAGE
```

## Secret

Create demo-only secret:

``` powershell
kubectl create secret generic app-secret --from-literal=DB_USER=demo --from-literal=DB_PASSWORD=demo123
```

`secret-deployment.yaml`:

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secret-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: secret-demo
  template:
    metadata:
      labels:
        app: secret-demo
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          env:
            - name: DB_USER
              valueFrom:
                secretKeyRef:
                  name: app-secret
                  key: DB_USER
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: app-secret
                  key: DB_PASSWORD
```

Run:

``` powershell
kubectl apply -f secret-deployment.yaml
kubectl get secrets
kubectl describe secret app-secret
kubectl get pods
kubectl exec deployment/secret-demo -- printenv DB_USER
```

Cleanup:

``` powershell
kubectl delete -f secret-deployment.yaml
kubectl delete -f configmap-deployment.yaml
kubectl delete -f configmap.yaml
kubectl delete secret app-secret
```

**Security:** Kubernetes Secret data is base64-encoded, not
automatically encrypted. Use demo credentials only; enable encryption at
rest and RBAC for real systems.

------------------------------------------------------------------------

# 7. Kubernetes Scaling, Rolling Updates and Troubleshooting

## Scaling and rolling update

`scale-deployment.yaml`:

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: scale-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: scale-demo
  template:
    metadata:
      labels:
        app: scale-demo
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80
```

Run:

``` powershell
kubectl apply -f scale-deployment.yaml
kubectl get deployments
kubectl get pods
kubectl scale deployment scale-demo --replicas=5
kubectl get pods
kubectl set image deployment/scale-demo nginx=nginx:1.28
kubectl rollout status deployment/scale-demo
kubectl rollout history deployment/scale-demo
kubectl rollout undo deployment/scale-demo
kubectl rollout status deployment/scale-demo
kubectl scale deployment scale-demo --replicas=2
```

## Deliberately faulty deployment

`wrong-deployment.yaml`:

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: broken-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: broken-demo
  template:
    metadata:
      labels:
        app: broken-demo
    spec:
      containers:
        - name: nginx
          image: nginx:invalid-tag
          ports:
            - containerPort: 80
```

Diagnose and fix:

``` powershell
kubectl apply -f wrong-deployment.yaml
kubectl get pods
kubectl describe pods
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl logs POD_NAME
kubectl set image deployment/broken-demo nginx=nginx:1.27
kubectl rollout status deployment/broken-demo
kubectl get pods
```

Cleanup:

``` powershell
kubectl delete deployment scale-demo
kubectl delete deployment broken-demo
```

Useful troubleshooting:

``` powershell
kubectl get all
kubectl get pods -o wide
kubectl describe pod POD_NAME
kubectl logs POD_NAME
kubectl logs POD_NAME --previous
kubectl get events
```

------------------------------------------------------------------------

# 8. Jenkins Freestyle Job and Pipeline

Install Jenkins LTS and a supported Java LTS version. Open
`http://localhost:8080`. Ensure Git and Python are installed and
available to the Jenkins service account. Install Git, Pipeline and
JUnit plugins if needed.

## Freestyle job

1.  Dashboard → New Item → `GitHub-Freestyle` → Freestyle project.
2.  Source Code Management → Git → enter repository URL and branch
    `*/main`.
3.  Build Steps → Execute Windows batch command:

``` bat
git --version
python --version
dir
```

4.  Save → Build Now → Console Output.

## Pipeline project files

`app.py`:

``` python
from flask import Flask
app = Flask(__name__)

@app.get("/")
def home():
    return "Jenkins pipeline app"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

`requirements.txt`:

``` text
flask
pytest
```

`test_app.py`:

``` python
from app import app

def test_home():
    response = app.test_client().get("/")
    assert response.status_code == 200
    assert response.data == b"Jenkins pipeline app"
```

`Dockerfile`:

``` dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
EXPOSE 5000
CMD ["python", "app.py"]
```

`Jenkinsfile` (Windows agent):

``` groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Install') {
            steps { bat 'python -m pip install -r requirements.txt' }
        }
        stage('Test') {
            steps { bat 'python -m pytest -v --junitxml=report.xml' }
        }
        stage('Build Docker Image') {
            steps { bat 'docker build -t jenkins-flask:v1 .' }
        }
    }
    post {
        always {
            junit allowEmptyResults: true, testResults: 'report.xml'
        }
        success { echo 'Pipeline successful' }
        failure { echo 'Pipeline failed' }
    }
}
```

Push:

``` powershell
git init
git add .
git commit -m "Add Jenkins pipeline project"
git branch -M main
git remote add origin https://github.com/USERNAME/JenkinsDemo.git
git push -u origin main
```

Jenkins → New Item → Pipeline → Pipeline script from SCM → Git →
repository URL → branch `*/main` → Script Path `Jenkinsfile` → Save →
Build Now.

Docker stage requires Jenkins to access Docker. For Linux agents, use
`sh` rather than `bat`.

------------------------------------------------------------------------

# 9. GitHub Actions

Use the Flask app and test from Topic 8. `requirements.txt`:

``` text
flask
pytest
```

Create `.github/workflows/python-ci.yml`:

``` yaml
name: Python CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
      - name: Run tests
        run: python -m pytest -v
```

Push:

``` powershell
git add .
git commit -m "Add GitHub Actions workflow"
git push origin main
```

GitHub repository → Actions → Python CI → inspect job logs and green
check.

------------------------------------------------------------------------

# 10. Python Automated Testing with Pytest

Install:

``` powershell
mkdir Topic10-Pytest
cd Topic10-Pytest
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install pytest
```

`calculator.py`:

``` python
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def multiply(a, b):
    return a * b

def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b
```

`test_calculator.py`:

``` python
import pytest
from calculator import add, subtract, multiply, divide

def test_add():
    assert add(10, 5) == 15

def test_subtract():
    assert subtract(10, 5) == 5

def test_multiply():
    assert multiply(10, 5) == 50

def test_divide():
    assert divide(10, 5) == 2

def test_divide_by_zero():
    with pytest.raises(ValueError):
        divide(10, 0)
```

Run:

``` powershell
pytest -v
pytest -q
pytest test_calculator.py::test_add -v
pytest --junitxml=report.xml
```

Expected: 5 tests passed.

------------------------------------------------------------------------

# 11. Terraform / Infrastructure as Code

Install Terraform CLI from
`https://developer.hashicorp.com/terraform/install`, reopen PowerShell
and check `terraform --version`.

`main.tf`:

``` hcl
terraform {
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5"
    }
  }
}
provider "local" {}

resource "local_file" "demo" {
  filename = "${path.module}/demo.txt"
  content  = var.message
}
```

`variables.tf`:

``` hcl
variable "message" {
  description = "Text written to the generated file"
  type        = string
  default     = "Infrastructure created using Terraform"
}
```

`outputs.tf`:

``` hcl
output "generated_file" {
  value = local_file.demo.filename
}
```

Run in the project directory:

``` powershell
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
```

Type `yes`. Verify:

``` powershell
Get-Content demo.txt
terraform output
terraform state list
terraform show
```

Destroy:

``` powershell
terraform destroy
```

Type `yes`.

Workspace commands:

``` powershell
terraform workspace list
terraform workspace new development
terraform workspace show
terraform workspace select default
terraform workspace delete development
```

Dependency example (add to a separate Terraform project):

``` hcl
resource "local_file" "first" {
  filename = "${path.module}/first.txt"
  content  = "First resource"
}
resource "local_file" "second" {
  filename = "${path.module}/second.txt"
  content  = "Second resource"
  depends_on = [local_file.first]
}
```

Then run `terraform init`, `terraform validate`, `terraform plan`,
`terraform apply`, `terraform destroy`.

*This uses the local provider and creates local files; no AWS/Azure
account is required.*

------------------------------------------------------------------------

# 12. DevSecOps and Monitoring

## A. DevSecOps / security scanning

Create `app.py`:

``` python
from flask import Flask, jsonify
app = Flask(__name__)

@app.get("/")
def home():
    return jsonify(message="Secure DevOps application")

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

`requirements.txt`:

``` text
flask
bandit
pip-audit
```

Install and scan:

``` powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
bandit -r . -x .venv
pip-audit
```

Build the Docker image using the Flask Dockerfile from Topic 4:

``` powershell
docker build -t secure-flask:v1 .
```

Install Trivy using its official Windows instructions, then:

``` powershell
trivy fs .
trivy image secure-flask:v1
```

## B. Prometheus and Grafana using Helm

Prerequisites: running Kubernetes cluster, `kubectl` and Helm.

``` powershell
kubectl get nodes
helm version
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create namespace monitoring
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring
kubectl get pods -n monitoring
kubectl get services -n monitoring
```

Grafana (keep terminal open):

``` powershell
kubectl port-forward service/monitoring-grafana 3000:80 -n monitoring
```

Open `http://localhost:3000`. Username: `admin`. Get generated password:

``` powershell
kubectl get secret monitoring-grafana -n monitoring -o jsonpath="{.data.admin-password}" | ForEach-Object { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }
```

Prometheus in a second terminal:

``` powershell
kubectl port-forward service/monitoring-kube-prometheus-prometheus 9090:9090 -n monitoring
```

Open `http://localhost:9090`. Example PromQL:

``` promql
up
kube_pod_info
container_memory_working_set_bytes
rate(container_cpu_usage_seconds_total[5m])
```

Cleanup:

``` powershell
helm uninstall monitoring -n monitoring
kubectl delete namespace monitoring
```

------------------------------------------------------------------------

# QUICK TROUBLESHOOTING

Python:

``` powershell
python --version
python -m pip install -r requirements.txt
python -m pip show flask
```

Docker:

``` powershell
docker info
docker ps -a
docker logs CONTAINER_NAME
docker rm -f CONTAINER_NAME
```

Kubernetes:

``` powershell
kubectl config current-context
kubectl get nodes
kubectl get pods
kubectl describe pod POD_NAME
kubectl logs POD_NAME
kubectl get events --sort-by=.metadata.creationTimestamp
```

Git:

``` powershell
git status
git remote -v
git branch
git log --oneline
```

Jenkins: check service status, Console Output, repository URL, branch,
Jenkinsfile path, and whether the service account can access
Python/Git/Docker.

Terraform:

``` powershell
terraform fmt
terraform validate
terraform plan
```

# FINAL PRACTICAL CHECKLIST

For every question: 1. Create the project directory. 2. Create the
correct source/configuration files. 3. Install dependencies. 4. Execute
commands in order. 5. Verify output in terminal, browser or dashboard.
6. Capture successful execution screenshots. 7. Clean up only resources
created for that practical.

Never commit `.env` files, passwords, tokens or real secrets to GitHub.
Do not run cleanup commands blindly.
