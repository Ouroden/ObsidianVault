[[kubernetes-resource]]

Serves as a internal load balancer across replicas of applications.
Can also be accessible outside the cluster.
Uses pod labels to determine which pods to serve.
There are 3 types:
 - ClusterIP - default - internal to cluster, provides stable IP
 - NodePort - listens on each node in cluster, can route from outside of cluster to inside app
 - LoadBalancer - uses CloudControllerManager component of Kubernetes to talk to the cloud API and provision external load balancer within cloud's system to route from outside to app, additional cost

One can use LoadBalancer or NodePort with Ingress and GatewayAPI to route instead.

Namespace.yaml
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: 04--service
```

Deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-minimal
  labels:
    foo: deployment-label
  annotations:
    bar: deployment-annotation
spec:
  replicas: 3
  selector:
    matchLabels:
      baz: pod-label
  template:
    metadata:
      labels:
        baz: pod-label
      annotations:
        bing: pod-annotation
    spec:
      containers:
        - name: nginx
          image: nginx:1.26.0
```

Service.nginx-clusterip.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
  labels:
    foo: service-label
  annotations:
    bar: service-annotation
spec:
  type: ClusterIP # This is the default value
  selector:
    baz: pod-label
  ports:
    - protocol: TCP
      port: 80 # Port the service is listening on
      targetPort: 80 # Port the container is listening on (if unset, defaults to equal port value)
```

Service.nginx-nodeport.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    baz: pod-label
  ports:
    - protocol: TCP
      port: 80 # Port the service is listening on
      targetPort: 80 # Port the container is listening on (if unset, defaults to equal port value)
      # nodePort: 30XXX (if unset, kubernetes will assign a port within 30000-32767)
```

Service.nginx-loadbalancer.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-loadbalancer
spec:
  type: LoadBalancer # Will only work if cluster is configured to provision one from an external source (e.g. cloud provider)
  selector:
    baz: pod-label
  ports:
    - protocol: TCP
      port: 80 # Port the service is listening on
      targetPort: 80 # Port the container is listening on (if unset, defaults to equal port value)
```

Commands
```bash
# kubectl get service or kubectl get svc
(devbox) [root@WINDOWS-ELLJEC8 Service]# k get service 
NAME                 TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
nginx-clusterip      ClusterIP      10.96.244.43   <none>        80/TCP         77s
nginx-loadbalancer   LoadBalancer   10.96.3.226    <pending>     80:31122/TCP   66s
nginx-nodeport       NodePort       10.96.78.239   <none>        80:30553/TCP   71s

# show coredns pods
(devbox) [root@WINDOWS-ELLJEC8 Service]# k get pods -n kube-system
NAME                                         READY   STATUS    RESTARTS      AGE
coredns-7db6d8ff4d-kkhp7                     1/1     Running   3 (42m ago)   7d9h
coredns-7db6d8ff4d-knx2g                     1/1     Running   3 (42m ago)   7d9h
etcd-kind-control-plane                      1/1     Running   0             42m
kindnet-jgt7j                                1/1     Running   3 (42m ago)   7d9h
kindnet-krrct                                1/1     Running   3 (42m ago)   7d9h
kindnet-v6zmq                                1/1     Running   3 (42m ago)   7d9h
kube-apiserver-kind-control-plane            1/1     Running   0             42m
kube-controller-manager-kind-control-plane   1/1     Running   3 (42m ago)   7d9h
kube-proxy-7mfkx                             1/1     Running   3 (42m ago)   7d9h
kube-proxy-nk55x                             1/1     Running   3 (42m ago)   7d9h
kube-proxy-x56wx                             1/1     Running   3 (42m ago)   7d9h
kube-scheduler-kind-control-plane            1/1     Running   3 (42m ago)   7d9h

```

Test ClusterIP from pod
```bash
1. Same namespace
# create temporary pod in the same namespace
(devbox) [root@WINDOWS-ELLJEC8 Service]# kubectl run curl-pod -it --rm --image=curlimages/curl --command -- sh

# run curl with DNS
~ $ curl nginx-clusterip:80
<html>

2. Any other namespace
# create temporary pod in any other namespace
(devbox) [root@WINDOWS-ELLJEC8 Service]# kubectl run curl-pod -it --rm --image=curlimages/curl --command -- sh

# run curl with DNS
~ $ curl nginx-clusterip
curl: (6) Could not resolve host: nginx-clusterip

# curl <ServiceName>.<Namespace>.svc.cluster.local
~ $ curl nginx-clusterip.04--service.svc.cluster.local
<html>


```

Additional kind setup for LoadBalancer to resolve pending status for getting external IP.
```bash
#Run sigs.k8s.io/cloud-provider-kind@latest to enable load balancer services with KinD
export GOBIN=$(git rev-parse --show-toplevel)/bin
export PATH=$GOBIN:$PATH
go install sigs.k8s.io/cloud-provider-kind@v0.2.0
sudo cloud-provider-kind
```

#kubernetes-resources #definitions 