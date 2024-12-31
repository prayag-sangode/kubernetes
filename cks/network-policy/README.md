# Network Policy

NetworkPolicy in Kubernetes controls traffic to and from pods in a cluster. It allows you to define rules that explicitly specify allowed ingress (incoming) and egress (outgoing) traffic. By default, without any NetworkPolicy, all traffic is allowed. However, once a NetworkPolicy is applied, any traffic not explicitly allowed is implicitly denied.

# Kubernetes Network Policy Lab

This lab demonstrates how to configure Kubernetes Network Policies to restrict communication between Pods across namespaces and within a namespace. The setup ensures that only the `frontend` pod can communicate with the `backend` pod, while all other traffic is blocked.

---

## Files

### 1. **`deployment.yaml`**
This file defines the deployments for:
- **`frontend`** in the `dev` namespace.
- **`backend`** in the `dev` namespace.
- **`external-app`** in the `default` namespace.

### 2. **`network-policy.yaml`**
This file defines a network policy that:
- Allows only traffic from the `frontend` pod to the `backend` pod in the `dev` namespace.
- Denies all other traffic to the `backend` pod.

---

## Setup Instructions

### 1. Create Namespace
```bash
kubectl create namespace dev
```

### 2. Deploy Pods
Apply the deployments to create the necessary Pods:
```bash
kubectl apply -f deployment.yaml
```

### 3. Verify Default Connectivity
Before applying the NetworkPolicy, all Pods can communicate freely. Test this using the following commands:

#### **From `frontend` to `backend`:**
```bash
kubectl -n dev exec deployment.apps/frontend -- bash -c "apt update && apt -y install curl"
kubectl -n dev exec deployment.apps/frontend -- bash -c "curl -s http://backend.dev.svc.cluster.local"
```

#### **From `external-app` to `backend`:**
```bash
kubectl exec deployment.apps/external-app -- bash -c "apt update && apt -y install curl"
kubectl exec deployment.apps/external-app -- bash -c "curl -s http://backend.dev.svc.cluster.local"
```

### 4. Apply the Network Policy
Restrict communication by applying the network policy:
```bash
kubectl apply -f network-policy.yaml
```

### 5. Test Connectivity
After applying the NetworkPolicy, verify the following:

#### **From `frontend` to `backend`:**
```bash
kubectl -n dev exec deployment.apps/frontend -- bash -c "apt update && apt -y install curl"
kubectl -n dev exec deployment.apps/frontend -- bash -c "curl -s http://backend.dev.svc.cluster.local"
```

#### **From `external-app` to `backend`:**
```bash
kubectl exec deployment.apps/external-app -- bash -c "apt update && apt -y install curl"
kubectl exec deployment.apps/external-app -- bash -c "curl -s http://backend.dev.svc.cluster.local"
```

---

## Clean-Up

To delete the created resources:
```bash
kubectl delete namespace dev
kubectl delete deployment external-app -n default
```

---

## Key Concepts

1. **NetworkPolicy**: A Kubernetes resource used to control traffic flow to and from pods.
2. **Allow Rules**: NetworkPolicies explicitly define what traffic is allowed. 
3. **Default Deny**: Any traffic not explicitly allowed by the policy is automatically denied.
4. **Namespace Isolation**: NetworkPolicies apply only to pods within the same namespace unless explicitly stated otherwise.

--- 
