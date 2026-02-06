[[kubernetes-resource]]

Evolution of Ingress API.
Not to confuse with "API Gateway":
 - GatewayAPI - specific Kubernetes API
 - API Gateway - general concept of a cloud resource that can take API calls and route them
 Adds support for TCP/UDP.
 Handles more advanced routing scenarios.

![[Pasted image 20251124153213.png]]

Namespace.yaml
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: 04--gatewayapi
```

Deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-minimal
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-pod-label
  template:
    metadata:
      labels:
        app: nginx-pod-label
    spec:
      containers:
        - name: nginx
          image: nginx:1.26.0
```

Gateway.gke.yaml
```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: Gateway
metadata:
  name: gke
spec:
  gatewayClassName: gke-l7-global-external-managed
  listeners:
    - name: http
      protocol: HTTP
      port: 80
      allowedRoutes:
        kinds:
          - kind: HTTPRoute
```

HTTPRoute.gke.yaml
```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: HTTPRoute
metadata:
  name: gke
spec:
  parentRefs:
    - name: gke
  hostnames:
    - "gateway-example-gke.com"
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: nginx-nodeport
          kind: Service
          port: 80
```

Setup
```bash
kubectl apply -f Namespace.yaml && setNamespace; kubens $NAMESPACE
kubectl apply -f Deployment.yaml
kubectl apply -f Gateway.gke.yaml
kubectl apply -f HTTPRoute.gke.yaml

kubectl delete -f Namespace.yaml
```

Commands
```bash
k get httproutes.gateway.networking.k8s.io
k get gateway
k describe gateway gke
# update /etc/hosts
[EXTERNAL IP from gke gateway] gateway-example-gke.com

```

GatewayClass.kong.yaml
```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: GatewayClass
metadata:
  name: kong
  annotations:
    konghq.com/gatewayclass-unmanaged: "true"
spec:
  controllerName: konghq.com/kic-gateway-controller
```

Gateway.kong.yaml
```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: Gateway
metadata:
  name: kong
spec:
  gatewayClassName: kong
  listeners:
    - name: proxy
      port: 80
      protocol: HTTP

```

HTTPRoute.kong.yaml
```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: HTTPRoute
metadata:
  name: kong
spec:
  parentRefs:
    - name: kong
  hostnames:
    - "gateway-example-kong.com"
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: nginx-clusterip
          kind: Service
          port: 80
```

Setup
```bash
helm upgrade --install kong ingress \
          --repo https://charts.konghq.com \
          -n kong \
          --create-namespace \
          --version 0.12.0

# Kong requires the latest CRDs
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.1.0/experimental-install.yaml

kubectl apply -f GatewayClass.kong.yaml
kubectl apply -f Gateway.kong.yaml
kubectl apply -f HTTPRoute.kong.yaml

kubectl delete ns kong
kubectl delete ns haproxy-controller
```

Commands
```bash
k get pods -n kong

k get gateway
k get httproutes.gateway.networking.k8s.io

k describe gateway gke
# update /etc/hosts
[EXTERNAL IP from gke gateway] gateway-example-gke.com
[EXTERNAL IP from k get gateway] gateway-example-kong.com
```

#kubernetes-resources #definitions 