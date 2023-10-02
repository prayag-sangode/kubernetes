### On all nodes

cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system

sudo apt-get update && sudo apt-get install -y containerd

sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl status containerd

sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt-get update
sudo apt-get install -y kubelet=1.23.0-00 kubeadm=1.23.0-00 kubectl=1.23.0-00
sudo apt-mark hold kubelet kubeadm kubectl

### Only on master -
#sudo kubeadm init --pod-network-cidr 192.168.0.0/16 --kubernetes-version 1.23.0

sudo kubeadm init --pod-network-cidr=10.0.0.0/16 --kubernetes-version 1.23.0
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl get nodes
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
kubectl get nodes --watch
kubeadm token create --print-join-command

### Join command -

sudo kubeadm join 192.168.200.61:6443 --token <token> --discovery-token-ca-cert-hash <hash>

[prayag@linux-bastion exercise]$ kubectl describe nodes k8s-master1.example.com | grep -i taint
Taints:             node-role.kubernetes.io/master:NoSchedule
[prayag@linux-bastion exercise]$ kubectl describe nodes k8s-worker1.example.com | grep -i taint
Taints:             <none>
[prayag@linux-bastion exercise]$ kubectl describe nodes k8s-worker2.example.com | grep -i taint
Taints:             <none>
[prayag@linux-bastion exercise]$

Nodes can have taints that repel pods, and pods can have tolerations that allow them to be scheduled on nodes with certain taints. 
Make sure there are no taints that prevent your pods from being scheduled.

---
Metal LB - https://docs.k0sproject.io/v1.23.6+k0s.2/examples/metallb-loadbalancer/
mkdir metallb
cd metallb
wget https://raw.githubusercontent.com/metallb/metallb/v0.10.2/manifests/namespace.yaml
wget https://raw.githubusercontent.com/metallb/metallb/v0.10.2/manifests/metallb.yaml
kubectl apply -f namespace.yaml
kubectl apply -f metallb.yaml

# Specify LB address pool. In this case same range as that of VM IP.

cat > metallb-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.200.155-192.168.200.166

kubectl apply -f metallb-configmap.yaml

---
Nginx Ingress - https://docs.k0sproject.io/v1.23.6+k0s.2/examples/nginx-ingress/
mkdir nginx-ingress
cd nginx-ingress
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.0/deploy/static/provider/baremetal/deploy.yaml
kubectl apply -f deploy.yaml
kubectl get pods -n ingress-nginx
kubectl get services -n ingress-nginx
kubectl -n ingress-nginx get ingressclasses
kubectl -n ingress-nginx annotate ingressclasses nginx ingressclass.kubernetes.io/is-default-class="true"

Now lets change service to LB instead of NodePort


apiVersion: v1
kind: Service
metadata:
  annotations:
  labels:
    helm.sh/chart: ingress-nginx-4.0.1
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 1.0.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: controller
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  type: LoadBalancer <-- edit this
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: http
      appProtocol: http
    - name: https
      port: 443
      protocol: TCP
      targetPort: https
      appProtocol: https
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/component: controller

kubectl apply -f deploy.yaml

We can see LB is allocated like below -

[prayag@linux-bastion nginx-ingress]$ kubectl -n ingress-nginx get svc
NAME                                 TYPE           CLUSTER-IP     EXTERNAL-IP       PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.105.46.8    192.168.200.155   80:32134/TCP,443:32135/TCP   4m30s
ingress-nginx-controller-admission   ClusterIP      10.102.83.73   <none>            443/TCP                      4m30s
[prayag@linux-bastion nginx-ingress]$

---
## Istio install
curl -L https://istio.io/downloadIstio | sh -
istio-1.19.0/bin/istioctl install
kubectl get pods -n istio-system
---

Upgrade -

### NOTES

# both 

kubectl get nodes

sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm=1.23.3-00 && \
sudo apt-mark hold kubeadm

# master

kubeadm version
sudo kubeadm upgrade plan
sudo kubeadm upgrade apply -y v1.23.3
kubectl get nodes
MASTER=`kubectl get nodes -ojsonpath="{.items[0].metadata.name}"`
echo $MASTER
WORKER=`kubectl get nodes -ojsonpath="{.items[1].metadata.name}"`
echo $WORKER
kubectl drain $MASTER --ignore-daemonsets
kubectl drain $WORKER --ignore-daemonsets

# worker
sudo kubeadm upgrade node

# both
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet=1.23.3-00 kubectl=1.23.3-00 && \
sudo apt-mark hold kubelet kubectl

sudo systemctl daemon-reload
sudo systemctl restart kubelet

# master
kubectl uncordon $MASTER
kubectl uncordon $WORKER
kubectl get nodes

---

examples 

kubectl create deployment nginx --image=nginx --port=80
kubectl expose deployment nginx
kubectl create ingress nginx --class=nginx \
  --rule nginx.example.com/=nginx:80

kubectl create deployment httpd --image=httpd --port=80
kubectl expose deployment httpd
kubectl create ingress httpd --class=nginx \
  --rule httpd.example.com/=httpd:80

--


apiVersion: v1
kind: Namespace
metadata:
  name: web
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-server
spec:
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: httpd
        image: httpd:2.4.48-alpine3.14
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: web-server-service
spec:
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-server-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: web.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-server-service
            port:
              number: 80



