# First Istio Book Tutorial
## Summary
This repo follows Istio demo(~/istio-1.18.1) sample tutorial 
<br>
kubernetes/bookinfo.yaml pull istio/bookinfo docker for deployment, service
<br>
kubernets/namespace.yml creates bookinfo-namespace 
<br>
istio/bookinfo-gateway.yaml deploys istio-gateway

Istio gateways
<br>
(1) Ingress gateway: handles incoming Http, Https traffic to the mesh
<br>
(2) Egress gateway: handles outgoing traffic from the mesh


## Links
https://phoenixnap.com/kb/istio-tutorial 
<br>
https://www.solo.io/topics/istio/istio-tutorial/

## Steps
```
# On MacOs - start socket_vmnet
HOMEBREW=$(which brew) && sudo ${HOMEBREW} services start socket_vmnet
HOMEBREW=$(which brew) && sudo ${HOMEBREW} services stop socket_vmnet

minikube start --driver qemu --network socket_vmnet

# Istio pre-installation check 
istioctl x precheck

# Istio profile refers to predefined configuration bundle, settings
# demo profile includes Istio core, Ingress gateway...
istioctl install --set profile=demo -y

# Create namespace
kubectl apply -f kubernetes/namespace.yml

kubectl get namespace

# Add label 'istio-injection=enabled'  to pods in 'bookinfo-namespace' namespace
# label are used to identify kub resources 
kubectl label namespace bookinfo-namespace istio-injection=enabled

# Verify labels
kubectl get namespace bookinfo-namespace --show-labels

kubectl apply -f kubernetes/bookinfo.yaml

kubectl get services
kubectl get pods

# Verify deployment, should return '<title>Simple Bookstore App</title>'
# kubectl exec POD_NAME [-c CONTAINER_NAME] COMMAND [FLAGS]
# kubectl exec: executes a command in a running container within a pod -> command is 'curl -sS ...'
# "$(...)": substitution syntax -> output of the clause will be used as the argument
# kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}'
# 1. -l app=ratings: get pod using label app=ratings
# 2. -o jsonpath='{.items[0].metadata.name}': output format of command to be JSON, use jsonpath to extract json
# grep -o: --only-matching, find regex <title>.*</title>

kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"

# Allow outside traffic to the application
kubectl apply -f istio/bookinfo-gateway.yaml

istioctl analyze --namespace bookinfo-namespace

# Check istio-ingressgateway service info
kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec}'

# Export local env variables
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')

# Verify env variables
echo "$INGRESS_PORT" && echo "$SECURE_INGRESS_PORT"

# Check miikube ip which is the ingress host
minikube ip

export INGRESS_HOST=$(minikube ip)

echo "$INGRESS_HOST"

# Create network tunnel to allow external traffic to reach services running in local Minikube cluster
minikube tunnel

export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT

echo "$GATEWAY_URL"

echo http://$GATEWAY_URL/productpage # Visit http://192.168.49.2:32628/productpage on browser

minikube stop
minikube delete
```



