# Install-Custom-Metrics-HPA

Kubernetes HP with Custom Metrics

## Install Helm3
```
curl -sSLf https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```
## Install Metrics-Server
```
git clone https://github.com/kubernetes-sigs/metrics-server.git
cd metrics-server/manifests/base
```

Edit deployment.yaml

Extend the args list with:
```
- --kubelet-insecure-tls 
```
Finally issue
```
kubectl apply -k ./
```

## Install Kube-Prometheus
```
git clone https://github.com/prometheus-operator/kube-prometheus.git
cd kube-prometheus
cd manifests/setup
kubectl create -f ./
cd ..
kubectl create -f ./
```
if you desire to access your prometheus UI via the public IP address of your cluster, extend the prometheus-service.yaml with the following line
```
type: LoadBalancer
kubectl apply -f prometheus-service.yaml
```
To get the port through which you can access the web UI issue
```
kubectl get service -n monitoring prometheus-k8s
```

## Install Prometheus-Adapter
```
git clone https://github.com/kubernetes-sigs/prometheus-adapter.git
cd deploy/manifests

kubectl create namespace custom-metrics
export PURPOSE=serving
openssl req -x509 -sha256 -new -nodes -days 365 -newkey rsa:2048 -keyout ${PURPOSE}-ca.key -out ${PURPOSE}-ca.crt -subj "/CN=ca"
echo '{"signing":{"default":{"expiry":"43800h","usages":["signing","key encipherment","'${PURPOSE}'"]}}}' > "${PURPOSE}-ca-config.json"
kubectl -n custom-metrics create secret tls cm-adapter-serving-certs --cert=./serving-ca.crt --key=./serving-ca.key
```
Edit custom-metrics-apiserver-deployment.yaml and do the following changes:
```
image: directxman12/k8s-prometheus-adapter-amd64
- --tls-cert-file=/var/run/serving-cert/tls.crt
- --tls-private-key-file=/var/run/serving-cert/tls.key
- --prometheus-url=http://prometheus-operated.monitoring.svc:9090/
```
Edit custom-metrics-apiservice.yaml
change all 
```
apiregistration.k8s.io/v1beta1
```
to 
```
apiregistration.k8s.io/v1
```
Finally issue
```
kubectl apply -f ./
```
## Install Istio (Optional)
```
git clone https://github.com/istio/istio.git
git checkout release-1.10.4-patch
kubectl create namespace istio-system
helm install istio-base manifests/charts/base -n istio-system --set global.jwtPolicy=first-party-jwt 
helm install istiod manifests/charts/istio-control/istio-discovery -n istio-system --set global.jwtPolicy=first-party-jwt 
helm install istio-ingress manifests/charts/gateways/istio-ingress -n istio-system --set global.jwtPolicy=first-party-jwt 
helm install istio-egress manifests/charts/gateways/istio-egress -n istio-system --set global.jwtPolicy=first-party-jwt 
```
