# Kubernetes Node Setup Script

This script automates the setup of a Kubernetes node with `containerd` as the container runtime, Calico as the CNI plugin, and initialization of a master node if applicable.

## Features
- Installs and configures `containerd` with proper settings for Kubernetes.
- Updates kernel modules and sysctl settings for networking compatibility.
- Installs Kubernetes components (`kubeadm`, `kubelet`, `kubectl`).
- Initializes the Kubernetes master node if the hostname matches.
- Installs Calico as the CNI plugin for pod networking.
- Outputs the `kubeadm join` command for worker nodes.

## Usage
1. Run the script on your node:
   ```bash
   bash k8s-ub24.sh
   ```
2. If the node is a master node (e.g., hostname `k8s-node1.example.com`), it initializes the Kubernetes control plane.
3. To add more nodes to the cluster, use the `kubeadm join` command output by the master node.

## Remove Taint for Single Node Usage
If you plan to use a single Kubernetes node (master node) for both control plane and workloads, remove the taint:
```bash
kubectl taint nodes k8s-node1.example.com node-role.kubernetes.io/control-plane-
```
This allows scheduling workloads on the master node.

### Example Deployment
Deploy and expose a sample NGINX application:
```bash
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --type="ClusterIP" --port=80
```

Node setup is complete and ready for use. 🎉
```
