# Kubernetes Deployment and Pod Configuration Documentation

This document provides an overview of key Kubernetes concepts related to Pods and Deployments. It covers Pod configuration (with and without resource limits), the role of labels and matchLabels in Deployments, rolling update mechanisms (including maxUnavailable and maxSurge), and how to record change causes for rollout history.

## Table of Contents
- [1. Pod Configuration](#1-pod-configuration)
  - [1.1. Pod Without Resource Limits](#11-pod-without-resource-limits)
  - [1.2. Pod With Resource Limits](#12-pod-with-resource-limits)
- [2. Deployment Configuration](#2-deployment-configuration)
  - [2.1. Deployment YAML Example](#21-deployment-yaml-example)
  - [2.2. Labels and MatchLabels](#22-labels-and-matchlabels)
- [3. Rolling Updates](#3-rolling-updates)
  - [3.1. Update Process and Strategy](#31-update-process-and-strategy)
  - [3.2. Change Cause Annotation](#32-change-cause-annotation)
  - [3.3. Rolling Update Strategy Parameters](#33-rolling-update-strategy-parameters)
- [4. Operational Tips](#4-operational-tips)

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

## 2. Deployment Configuration

### 2.1. Deployment YAML Example

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

### 2.2. Labels and MatchLabels
- **Deployment Metadata Labels:** Organize and filter Deployments.
- **Selector (matchLabels):** Defines which Pods the Deployment manages.
- **Pod Template Labels:** Ensures Pods created by the Deployment have correct labels.

---

## 3. Rolling Updates

### 3.1. Update Process and Strategy
- Kubernetes creates new Pods while keeping old ones running until new ones pass readiness checks.
- Old Pods are terminated only after new ones are available.
- **Deployments** use rolling updates by default with `kubectl rollout` commands, ensuring zero-downtime upgrades.
- **StatefulSets** support rolling updates but perform them in a controlled order (respecting pod identity and ordering).
- **DaemonSets** can perform rolling updates using the `RollingUpdate` update strategy.
- 🚫 **Standalone Pods do not support rolling updates** because they are not managed by a controller. If you delete a Pod, Kubernetes does not automatically recreate it.

### 3.2. Change Cause Annotation
To document why an update occurred:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.19
kubectl annotate deployment/nginx-deployment kubernetes.io/change-cause="Updated image to nginx:1.19"
```

### 3.3. Rolling Update Strategy Parameters
The `strategy` block should be placed inside the `spec` section of a Deployment.

- **maxUnavailable:** Maximum number of Pods that can be unavailable during an update.
- **maxSurge:** Maximum number of extra Pods that can be created above the desired replica count during an update.

Example configuration:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

---

## 4. Operational Tips
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

## Summary
- Define Pods with or without resource limits.
- Use Deployments for managing multiple replicas.
- Ensure `matchLabels` in Deployments match Pod labels.
- Utilize rolling update strategies (`maxUnavailable`, `maxSurge`) for zero downtime.
- Record deployment changes using annotations for tracking history.
