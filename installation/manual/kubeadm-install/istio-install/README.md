curl -L https://istio.io/downloadIstio | sh -
istio-1.19.0/bin/istioctl install
kubectl get pods -n istio-system

