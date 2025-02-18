# Kubernetes Deployment, ReplicaSet, and Pod Configuration Documentation

This document provides an overview of key Kubernetes concepts related to Pods, ReplicaSets, and Deployments. It covers Pod configuration (with and without resource limits), the role of ReplicaSets in managing Pods, the difference between ReplicaSets and Deployments, rolling update mechanisms (including `maxUnavailable` and `maxSurge`), and how to record change causes for rollout history.

## Table of Contents
- [1. Pod Configuration](#1-pod-configuration)
  - [1.1. Pod Without Resource Limits](#11-pod-without-resource-limits)
  - [1.2. Pod With Resource Limits](#12-pod-with-resource-limits)
- [2. ReplicaSet Configuration](#2-replicaset-configuration)
  - [2.1. ReplicaSet YAML Example](#21-replicaset-yaml-example)
  - [2.2. When to Use a ReplicaSet](#22-when-to-use-a-replicaset)
- [3. Deployment Configuration](#3-deployment-configuration)
  - [3.1. Deployment YAML Example](#31-deployment-yaml-example)
  - [3.2. Labels and MatchLabels](#32-labels-and-matchlabels)
  - [3.3. Difference Between ReplicaSet and Deployment](#33-difference-between-replicaset-and-deployment)
- [4. Rolling Updates](#4-rolling-updates)
  - [4.1. Update Process and Strategy](#41-update-process-and-strategy)
  - [4.2. Change Cause Annotation](#42-change-cause-annotation)
  - [4.3. Rolling Update Strategy Parameters](#43-rolling-update-strategy-parameters)
  - [4.4. Operational Tips for rolling updates](#44-operational-tips-for-rolling-updates)
- [5. Difference Between Kubernetes Jobs and CronJobs](#5-difference-between-kubernetes-jobs-and-cronjobs)
  - [5.1. Kubernetes Jobs](#51-kubernetes-jobs)
  - [5.1. Kubernetes CronJobs](#52-kubernetes-cronjobs)
---

## 1. Pod Configuration

### 1.1. Pod Without Resource Limits
A basic Pod configuration running a container with the `httpd:latest` image:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
  labels:
    app: httpd_app
spec:
  containers:
  - name: httpd-container
    image: httpd:latest
    ports:
    - containerPort: 80
```

### 1.2. Pod With Resource Limits
This configuration adds resource requests and limits for CPU and memory:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
spec:
  containers:
  - name: httpd-container
    image: httpd:latest
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "15Mi"
        cpu: "100m"
      limits:
        memory: "20Mi"
        cpu: "100m"
```

---

## 2. ReplicaSet Configuration

### 2.1. ReplicaSet YAML Example

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
  labels:
    app: nginx_app
    type: front-end
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx_app
  template:
    metadata:
      labels:
        app: nginx_app
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80
```

### 2.2. When to Use a ReplicaSet
A ReplicaSet ensures that a specified number of identical Pods are running at any given time. This is useful when:
- You need to maintain a consistent number of Pods without manual intervention.
- Your application does not require rolling updates or version management.
- You want more control over scaling but do not need advanced deployment strategies like canary or blue-green deployments.

However, **Deployments are preferred over ReplicaSets** because they provide additional management capabilities like rollbacks and rolling updates.

---

## 3. Deployment Configuration

### 3.1. Deployment YAML Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx_app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx_app
  template:
    metadata:
      labels:
        app: nginx_app
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
        ports:
        - containerPort: 80
```
### 3.2. Labels and MatchLabels
- **Deployment Metadata Labels:** Organize and filter Deployments.
- **Selector (matchLabels):** Defines which Pods the Deployment manages.
- **Pod Template Labels:** Ensures Pods created by the Deployment have correct labels.


### 3.3. Difference Between ReplicaSet and Deployment
| Feature       | ReplicaSet | Deployment |
|--------------|------------|------------|
| Scaling      | Yes (manual) | Yes (automatic and declarative) |
| Rollback Support | No | Yes |
| Rolling Updates | No | Yes |
| Use Case | Ensuring a fixed number of Pods | Managing updates, rollbacks, and scaling |

**Summary:** If you only need to ensure a certain number of Pods are running, use a ReplicaSet. If you need update strategies and rollbacks, use a Deployment.

---

## 4. Rolling Updates

### 4.1. Update Process and Strategy
- Kubernetes creates new Pods while keeping old ones running until new ones pass readiness checks.
- Old Pods are terminated only after new ones are available.
- **Deployments** use rolling updates by default with `kubectl rollout` commands, ensuring zero-downtime upgrades.

### 4.2. Change Cause Annotation
To document why an update occurred:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.19
kubectl annotate deployment/nginx-deployment kubernetes.io/change-cause="Updated image to nginx:1.19"
```

### 4.3. Rolling Update Strategy Parameters
The `strategy` block should be placed inside the `spec` section of a Deployment.

- **maxUnavailable:** Maximum number of Pods that can be unavailable during an update.
- **maxSurge:** Maximum number of extra Pods that can be created above the desired replica count during an update.

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx_app
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: nginx_app
  template:
    metadata:
      labels:
        app: nginx_app
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
        ports:
        - containerPort: 80
```

---

### 4.4. Operational Tips for rolling updates
- **Monitor rollout status:**
  ```bash
  kubectl rollout status deployment/nginx-deployment
  ```
- **Check rollout history:**
  ```bash
  kubectl rollout history deployment/nginx-deployment
  ```
- **Rollback to a previous version:**
  ```bash
  kubectl rollout undo deployment/nginx-deployment
  ```
---


## 5. Difference Between Kubernetes Jobs and CronJobs

### 5.1. Kubernetes Jobs
A Kubernetes Job creates one or more pods to run a task to completion. Once the task is finished, the Job is marked as complete. Jobs are useful for batch processing tasks that need to run once or a fixed number of times.

### Example: Kubernetes Job

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: simple-job
spec:
  template:
    spec:
      containers:
      - name: hello
        image: busybox
        command: ["echo", "Hello from Kubernetes Job!"]
      restartPolicy: Never
```

#### Expected Output
```shell
kubectl apply -f job.yaml
kubectl get jobs
kubectl get pods --show-all
kubectl logs <pod-name>
```
Expected log output:
```shell
Hello from Kubernetes Job!
```

---

### 5.2 Kubernetes CronJobs
A Kubernetes CronJob runs a Job on a scheduled basis, similar to a Unix cron job. This is useful for periodic tasks like backups, report generation, or data cleanup.

#### Example: Kubernetes CronJob

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: simple-cronjob
spec:
  schedule: "*/1 * * * *" # Runs every minute
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            command: ["echo", "Hello from Kubernetes CronJob!"]
          restartPolicy: Never
```

#### Expected Output
```shell
kubectl apply -f cronjob.yaml
kubectl get cronjobs
kubectl get jobs
kubectl get pods --show-all
kubectl logs <pod-name>
```
Expected log output (repeats every minute):
```shell
Hello from Kubernetes CronJob!
```
```
---
## Summary
- Define Pods with or without resource limits.
- Use ReplicaSets to maintain a fixed number of Pods.
- Use Deployments for version management and updates.
- Utilize rolling update strategies (`maxUnavailable`, `maxSurge`) for zero downtime.
- Record deployment changes using annotations for tracking history.
- Difference between Kubernetes CronJobs and Kubernetes Jobs
