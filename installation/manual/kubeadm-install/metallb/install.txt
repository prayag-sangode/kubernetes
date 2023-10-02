## Metal LB - https://docs.k0sproject.io/v1.23.6+k0s.2/examples/metallb-loadbalancer/
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
