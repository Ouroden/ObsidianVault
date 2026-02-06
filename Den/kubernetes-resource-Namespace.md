[[kubernetes-resource]]

Groups resources within a cluster.

There are 4 initial namespaces:
 - default
 - kube-system
 - kube-node-lease
 - kube-public

By default namespaces do not act as a network/security boundary.

Create namespace imperatively:
```bash
kubectl create namespace 04--namespace-cli
```

or better with keeping config in git:
```bash
kubectl apply -f Namespace.yaml
```
Namespace.yaml:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: 04--namespace-file
```

Get namespace info:
```bash
kubectl get namespace 04--namespace-cli -o yaml
```

Delete namespaces:
```bash
kubectl delete namespace 04--namespace-cli
kubectl delete -f Namespace.yaml
```
      



#kubernetes-resources #definitions 
