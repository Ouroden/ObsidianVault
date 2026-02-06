[[kubernetes-resource]]

Runs a copy of the specified pod on all or some nodes in the cluster.
Useful for applications such as:
 - log aggregation
 - node monitoring
 - storage setups

By default run specified pod on all nodes excluding control-plane nodes. 

Namespace.yaml
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: 04--daemonset
```

Pod.fluentd-minimal.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: fluentd-minimal
spec:
  containers:
    - name: fluentd
      image: fluentd:v1.16-1
```

DaemonSet.fluentd-minimal.yaml
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-minimal
  namespace: 04--daemonset
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
        - name: fluentd
          image: fluentd:v1.16-1
```

Setup
```bash
kubectl apply -f Namespace.yaml && setNamespace; kubens $NAMESPACE
kubectl apply -f Pod.fluentd-minimal.yaml
kubectl apply -f DaemonSet.fluentd-minimal.yaml
kubectl delete -f Namespace.yaml
```

Commands
```bash
(devbox) [root@WINDOWS-ELLJEC8 DaemonSet]# k get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES
fluentd-minimal-mhhdf   1/1     Running   0          20s   10.244.2.15   kind-worker2   <none>           <none>
fluentd-minimal-zgbxd   1/1     Running   0          20s   10.244.1.16   kind-worker    <none>           <none>

(devbox) [root@WINDOWS-ELLJEC8 DaemonSet]# k get nodes
NAME                 STATUS   ROLES           AGE   VERSION
kind-control-plane   Ready    control-plane   30d   v1.34.0
kind-worker          Ready    <none>          30d   v1.34.0
kind-worker2         Ready    <none>          30d   v1.34.0

```

#kubernetes-resources #definitions 