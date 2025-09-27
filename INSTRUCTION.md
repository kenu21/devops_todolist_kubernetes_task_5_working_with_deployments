# Deployment Instructions for Django ToDo App on Kubernetes

This document explains how to deploy the Django ToDo app to a Kubernetes cluster, including configuration choices for resources, autoscaling, and deployment strategy.

---

## 1. Deploying the Application

To deploy the app, execute the following commands:

```bash
kubectl apply -f .infrastructure/namespace.yml
kubectl apply -f .infrastructure/deployment.yml
kubectl apply -f .infrastructure/hpa.yml
kubectl apply -f .infrastructure/nodeport.yml
```

These commands will:

1. Create the `mateapp` namespace.
2. Deploy the ToDo app using a Kubernetes Deployment.
3. Configure Horizontal Pod Autoscaling (HPA) for dynamic scaling.
4. Expose the app through a NodePort service.

After deployment, the app will be accessible at:

```
http://localhost:30080/
```

---

## 2. Resource Requests and Limits

The Todo app container has the following resource configuration:

| Resource | Request | Limit |
| -------- | ------- | ----- |
| CPU      | 250m    | 500m  |
| Memory   | 64Mi    | 128Mi |

**Explanation:**

* **Requests** are the minimum resources guaranteed to the pod. The ToDo app is lightweight, so we set modest values (250m CPU, 64Mi memory) for idle conditions.
* **Limits** define the maximum resources the pod can consume (500m CPU, 128Mi memory), allowing it to handle traffic spikes without being throttled immediately.
* Properly setting requests ensures efficient pod scheduling, while limits prevent resource exhaustion on nodes.

---

## 3. Horizontal Pod Autoscaler (HPA)

The HPA is configured as follows:

* **Minimum replicas:** 2
* **Maximum replicas:** 5
* **Metrics:**

  * CPU utilization: 70% target
  * Memory utilization: 70% target

**Explanation:**

* Minimum of 2 pods ensures high availability during low traffic.
* Maximum of 5 pods allows scaling under higher load.
* Target utilization of 70% balances performance and resource efficiency.

**Scaling formula:**

```
Desired Replicas = Current Replicas × (Current Metric Value / Target Metric Value)
```

**Example:**
If CPU utilization reaches 140% on 2 pods:

```
Desired Replicas = 2 × (140 / 70) = 4 pods
```

This ensures the app automatically scales based on demand.

---

## 4. Deployment Strategy

The deployment uses a **RollingUpdate** strategy:

* **maxUnavailable:** 1
* **maxSurge:** 1

**Explanation:**

* RollingUpdate updates pods gradually, avoiding downtime.
* `maxUnavailable: 1` ensures at least one pod is always available during updates.
* `maxSurge: 1` allows creating an extra pod temporarily to speed up deployment.
* With 2 replicas, these values provide smooth updates and minimal service disruption.

---

This instruction set ensures your Django ToDo app is deployed efficiently, highly available, and able to handle increased load with autoscaling.
