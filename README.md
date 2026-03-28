# Smart Manufacturing Machine Efficiency Prediction

An end-to-end **MLOps** project that predicts manufacturing machine efficiency (**High / Low / Medium**) using a Logistic Regression model, served through a Flask web application, and deployed to a **Kubernetes cluster on GCP** via a fully automated **Jenkins CI/CD pipeline** powered by a **Jenkins Shared Library**.

Built by **Hitesh Nikam**.

![MLOps Architecture on GCP](assets/c__Users_Kirtish_Wankhedkar_AppData_Roaming_Cursor_User_workspaceStorage_72f8378b16e3723212031aa10427a0c7_images_Architecture-968fd841-6552-4e2f-8cf5-8979bdc1d10d.png)

---

## Table of Contents

- [Project Overview](#project-overview)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [ML Pipeline](#ml-pipeline)
- [Flask Web Application](#flask-web-application)
- [CI/CD Pipeline](#cicd-pipeline)
- [Jenkins Shared Library](#jenkins-shared-library)
- [Kubernetes Deployment](#kubernetes-deployment)
- [Getting Started](#getting-started)

---

## Project Overview

Manufacturing plants generate real-time sensor data from machines -- temperature, vibration, power consumption, network latency, error rates, and more. I built this project to solve a real-world problem: **predicting the efficiency status of a manufacturing machine** based on its operational parameters, and automating the entire deployment lifecycle using modern MLOps practices.

The goal was to go beyond just training a model -- I wanted to build a complete production-grade workflow where a single `git push` triggers an automated pipeline that builds, containerizes, and deploys the application to a Kubernetes cluster on **Google Cloud Platform**.

---

## Architecture

The system follows a modern MLOps architecture deployed on **Google Cloud Platform**:

1. **Code Repository (GitHub)** -- Application code, Dockerfile, and Kubernetes manifests are version-controlled here.
2. **Jenkins Shared Library Repository (GitHub)** -- A separate repo containing reusable Groovy scripts (`vars/`) that I built to keep CI/CD logic DRY across projects.
3. **GCP Virtual Machine** -- Hosts Docker, Jenkins, and Kubernetes (kubectl) -- acts as the central automation hub.
4. **Jenkins CI/CD Pipeline** -- Checks out code, builds a Docker image, pushes it to Docker Hub, and deploys to Kubernetes.
5. **Docker Hub** -- Stores versioned Docker images of the application.
6. **Kubernetes Cluster** -- Runs the Flask application container, exposed via a NodePort service.

![Jenkins Shared Library Concept](assets/c__Users_Kirtish_Wankhedkar_AppData_Roaming_Cursor_User_workspaceStorage_72f8378b16e3723212031aa10427a0c7_images_Jenkins-Shared_Library-5f0960f2-9c74-4661-b2e5-f11d3505c6e7.png)

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Language** | Python 3.11 |
| **Machine Learning** | scikit-learn (Logistic Regression, StandardScaler, LabelEncoder) |
| **Data Handling** | pandas, numpy, joblib |
| **Visualization** | matplotlib, seaborn |
| **Web Framework** | Flask, Jinja2 |
| **Containerization** | Docker |
| **Container Orchestration** | Kubernetes (Deployment + NodePort Service) |
| **CI/CD** | Jenkins Declarative Pipeline |
| **Pipeline Reusability** | Jenkins Shared Library (Groovy) |
| **Cloud Platform** | Google Cloud Platform (GCP VM) |
| **Image Registry** | Docker Hub |
| **Version Control** | Git / GitHub |

---

## Project Structure

```
smart-manufacturing-mlops/
├── application.py              # Flask web app (serves predictions)
├── Dockerfile                  # Docker image definition (Python 3.11)
├── Jenkinsfile                 # CI/CD pipeline using shared library
├── setup.py                    # Python package setup
├── requirements.txt            # Python dependencies
│
├── src/                        # Core ML source code
│   ├── data_processing.py      # Data loading, preprocessing, scaling
│   ├── model_training.py       # Model training and evaluation
│   ├── logger.py               # Custom logging utility
│   └── custom_exception.py     # Custom exception handling with traceback
│
├── pipeline/
│   └── training_pipeline.py    # End-to-end training orchestration
│
├── artifacts/
│   ├── raw/
│   │   └── data.csv            # Raw manufacturing sensor data
│   ├── processed/              # Scaled train/test splits + scaler
│   └── models/
│       └── model.pkl           # Trained Logistic Regression model
│
├── k8s/
│   ├── deployment.yaml         # Kubernetes Deployment manifest
│   └── service.yaml            # Kubernetes NodePort Service (port 30080)
│
├── templates/
│   └── index.html              # HTML form for predictions
├── static/
│   └── style.css               # Styling
└── notebook/
    └── notebook.ipynb          # Exploratory data analysis
```

---

## ML Pipeline

The training pipeline (`pipeline/training_pipeline.py`) runs two stages sequentially:

### 1. Data Processing (`src/data_processing.py`)

- Loads raw CSV data from `artifacts/raw/data.csv`
- Parses `Timestamp` and extracts `Year`, `Month`, `Day`, `Hour` as new features
- Drops non-predictive columns (`Timestamp`, `Machine_ID`)
- Label-encodes categorical columns (`Operation_Mode`, `Efficiency_Status`)
- Applies `StandardScaler` to normalize all features
- Performs stratified train/test split (80/20)
- Persists processed artifacts as `.pkl` files using joblib

### 2. Model Training (`src/model_training.py`)

- Loads processed train/test splits
- Trains a `LogisticRegression` classifier (max_iter=1000)
- Evaluates performance on the test set using Accuracy, Precision, Recall, and F1-Score
- Saves the trained model to `artifacts/models/model.pkl`

### Input Features

| Feature | Description |
|---|---|
| `Operation_Mode` | Machine operation mode (label-encoded) |
| `Temperature_C` | Operating temperature in Celsius |
| `Vibration_Hz` | Vibration frequency in Hertz |
| `Power_Consumption_kW` | Power usage in kilowatts |
| `Network_Latency_ms` | Network latency in milliseconds |
| `Packet_Loss_%` | Data packet loss percentage |
| `Quality_Control_Defect_Rate_%` | Product defect rate percentage |
| `Production_Speed_units_per_hr` | Production throughput (units/hr) |
| `Predictive_Maintenance_Score` | Predictive maintenance score |
| `Error_Rate_%` | Machine error rate percentage |
| `Year`, `Month`, `Day`, `Hour` | Extracted timestamp components |

### Prediction Classes

| Label | Efficiency Status |
|---|---|
| 0 | High |
| 1 | Low |
| 2 | Medium |

---

## Flask Web Application

The Flask app (`application.py`) serves a web UI where users input machine parameters and receive a real-time efficiency prediction.

- **`GET /`** -- Displays the input form with fields for each feature
- **`POST /`** -- Accepts form data, scales inputs using the saved scaler, runs model inference, and returns the predicted efficiency label
- **Port:** 5000

---

## CI/CD Pipeline

The `Jenkinsfile` defines a 4-stage declarative pipeline that automates the entire delivery workflow:

```groovy
@Library('jenkins-shared') _

pipeline {
    agent any
    environment {
        DOCKER_REPO = "hiteshnikam/smart-manufacturing-mlops"
    }
    stages {
        stage('Checkout')            { ... }
        stage('Build & Push Image')  { ... }
        stage('Install Kubectl')     { ... }
        stage('Deploy to Kubernetes'){ ... }
    }
}
```

| Stage | What It Does | Shared Library Function |
|---|---|---|
| **Checkout** | Clones the GitHub repository | `gitCheckout()` |
| **Build & Push Image** | Builds the Docker image and pushes to Docker Hub | `dockerBuildAndPush()` |
| **Install Kubectl** | Ensures kubectl is available on the Jenkins agent | `installKubectl()` |
| **Deploy to Kubernetes** | Applies K8s deployment and service manifests | `k8sDeploy()` |

### Jenkins Credentials Required

| Credential ID | Purpose |
|---|---|
| `github-token` | GitHub repository access |
| `dockerhub-token` | Docker Hub image push |
| `kubeconfig` | Kubernetes cluster authentication |

---

## Jenkins Shared Library

A key design decision in this project was to use a **Jenkins Shared Library** (`jenkins-shared`) -- a separate Git repository I set up containing reusable Groovy scripts that any Jenkins pipeline can import.

Instead of duplicating CI/CD logic in every project's Jenkinsfile, the shared library centralizes common operations:

- `gitCheckout(repoUrl, branch, credentialsId)` -- Clones a Git repository
- `dockerBuildAndPush(repoName, credentialsId)` -- Builds and pushes a Docker image
- `installKubectl()` -- Installs the kubectl CLI tool
- `k8sDeploy(kubeconfigCredentialsId)` -- Applies Kubernetes manifests to a cluster

**Why this matters:**
- **Reusability** -- Write CI/CD steps once, use them in any pipeline
- **Consistency** -- Every project follows the same tested deployment logic
- **Maintainability** -- Fix a bug or add a feature in one place, all pipelines benefit

---

## Kubernetes Deployment

The application is deployed to Kubernetes using two manifests:

- **Deployment** (`k8s/deployment.yaml`) -- Runs 1 replica of the Flask container using `hiteshnikam/smart-manufacturing-mlops:latest`
- **Service** (`k8s/service.yaml`) -- NodePort service exposing the application on port **30080**

Access the running application at: `http://<node-ip>:30080`

---

## Getting Started

### Prerequisites

- Python 3.11+
- Docker
- Kubernetes cluster (or Minikube for local testing)
- Jenkins with the Shared Library configured

### Run Locally

```bash
# Install dependencies
pip install -e .

# Run the training pipeline
python pipeline/training_pipeline.py

# Start the Flask app
python application.py
```

The app will be available at `http://localhost:5000`.

### Run with Docker

```bash
# Build the image
docker build -t smart-manufacturing-mlops .

# Run the container
docker run -p 5000:5000 smart-manufacturing-mlops
```

### Deploy to Kubernetes

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

Access at `http://<node-ip>:30080`.

---

## Author

**Hitesh Nikam**

If you have questions or suggestions, feel free to open an issue or reach out.
