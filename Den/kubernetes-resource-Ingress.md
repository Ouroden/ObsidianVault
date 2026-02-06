[[kubernetes-resource]]

Enables routing traffic to many services via single external LoadBalancer.
Many options to chose from:
 - Ingress-nginx
 - HAProxy
 - Kong
 - Istio
 - Traefik
Officially supports only layer 7 routing e.g http/https, but some implementation allow for layer 4 TCP/UDP with additional configuration. Annotations can be used.

![[Pasted image 20251124151734.png]]


Namespace.yaml
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: 04--ingress
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

Service.nginx-nodeport.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort # For GKE load balancer this has to be a NodePort
  selector:
    app: nginx-pod-label
  ports:
    - protocol: TCP
      port: 80 # Port the service is listening on
      targetPort: 80 # Port the container is listening on (if unset, defaults to equal port value)
```

Ingress.minimal-gke.yaml
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-gke
  annotations:
    kubernetes.io/ingress.class: "gce"
    # kubernetes.io/ingress.class: "gce-internal" (for traffic external to cluster but internal to VPC)
spec:
  # NOTE: You can't use spec.ingressClassName for GKE ingress
  rules:
    - host: "ingress-example-gke.com"
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-nodeport
                port:
                  number: 80
```

Setup
```bash
kubectl apply -f Namespace.yaml && setNamespace; kubens $NAMESPACE
kubectl apply -f Deployment.yaml
kubectl apply -f Service.nginx-nodeport.yaml # Nodeport needed for GKE
kubectl apply -f Ingress.minimal-gke.yaml # for GKE
kubectl delete -f Namespace.yaml
```

Service.nginx-clusterip.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
spec:
  type: ClusterIP # This is the default value
  selector:
    app: nginx-pod-label
  ports:
    - protocol: TCP
      port: 80 # Port the service is listening on
      targetPort: 80 # Port the container is listening on (if unset, defaults to equal port value)
```

Ingress.minimal-nginx.yaml
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-nginx
  # Can use
  # annotations:
  #   kubernetes.io/ingress.class: "nginx"
spec:
  ingressClassName: nginx
  rules:
    - host: "ingress-example-nginx.com"
      http:
        paths:
          - path: /
            pathType: ImplementationSpecific
            backend:
              service:
                name: nginx-clusterip
                port:
                  number: 80
```

Setup
```bash
# install your own Ingress Controller into the cluster
helm upgrade --install ingress-nginx ingress-nginx \
          --repo https://kubernetes.github.io/ingress-nginx \
          --namespace ingress-nginx \
          --create-namespace \
          --version 4.10.1
          
kubectl apply -f Service.nginx-clusterip.yaml
kubectl apply -f Ingress.minimal-nginx.yaml

k delete ingress --all
k delete ns ingress-nginx
```


Commands
```bash
# get Load Balancer External IP
get svc -n ingress-nginx

# update /etc/hosts
[External IP] ingress-example-gke.com # first example
[External IP] ingress-example-nginx.com # second example
```

#kubernetes-resources #definitions 