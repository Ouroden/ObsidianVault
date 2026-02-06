[[kubernetes-resource]]



Namespace.yaml
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: 04--ingress
```

Secret.string-data.yaml
```yaml

```

Secret.base64-data.yaml
```yaml

```

Secret.dockerconfigjson.yaml
```yaml

```

Pod.secret-example.yaml
```yaml

```

Setup
```bash
kubectl apply -f Namespace.yaml && setNamespace; kubens $NAMESPACE
kubectl apply -f Secret.string-data.yaml
kubectl apply -f Secret.base64-data.yaml
kubectl apply -f Secret.dockerconfigjson.yaml
kubectl apply -f Pod.secret-example.yaml
kubectl delete -f Namespace.yaml
```

Commands
```bash

```

#kubernetes-resources #definitions 