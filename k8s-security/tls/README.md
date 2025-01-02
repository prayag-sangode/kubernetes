  
## Assuming K8s cluster is created. In this case vanilla k8s cluster is provisioned.

## Install Metallb if you are using vanilla k8s (created using kubeadm)
```bash
kubectl apply -f metallb.yaml
kubectl apply -f metallb-configmap.yaml
```
## Install nginx-ingress
```bash
kubectl apply -f ingress-deploy.yaml
kubectl -n ingress-nginx annotate ingressclasses nginx ingressclass.kubernetes.io/is-default-class="true"
kubectl apply -f nginx-ingress-svc.yam
```

## Create privatekey and certificate
```bash
openssl req -x509 -nodes -newkey rsa:2048 -keyout nginx.example.com.key -out nginx.example.com.crt -days 365 -subj "/CN=nginx.example.com/O=nginx.example.com"
```
 
##  Create tls secret
```bash
kubectl create secret tls nginx-cert --cert=nginx.example.com.crt --key=nginx.example.com.key -n dev
```
## Create deployment, service and ingress
```bash
kubectl apply -f deployment.yaml
```

## Create test deployment
```bash
kubectl create deployment nginx-deployment --image=nginx  
```

## Test. Make sure nginx.example.com points to ingress lb ip in /etc/hosts

```bash
curl -k https://nginx.example.com
```

## We can test from test pod as well
```bash
kubectl exec -it pod/nginx-deployment-c45d79c8-khmfm -- /bin/bash
```
