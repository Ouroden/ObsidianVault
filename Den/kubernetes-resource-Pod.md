[[kubernetes-resource]]

Smallest deployable unit.
Can contain multiple containers.
If many containers, we can distinct:
- primary container - usually one
- Init container - one or more - is run before primary to setup files etc.
- Sidecar container - one or more - is run alongside primary, example use case: network proxy

Containers within a pod share networking and storage.

Many configurations available:
- listening ports
- health probes
- resource request/limits
- security context
- environment variables
- volumes
- DNS polices

Namespace.yaml:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: 04--pod
```
```bash
kubectl apply -f Namespace.yaml
```


Pod.nginx-minimal.yaml:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-minimal
spec:
  containers:
    - name: nginx
      image: nginx:1.26.0
```

```bash
kubectl apply -n ${NAMESPACE} -f Pod.nginx-minimal.yaml
kubectl port-forward -n ${NAMESPACE} nginx-minimal 8080:80
```


Pod.nginx-minimal.yaml:
```yaml
# Based on https://github.com/BretFisher/podspec
apiVersion: v1
kind: Pod
metadata:
  name: nginx-better
  namespace: 04--pod
spec:
  containers:
    - name: nginx
      image: cgr.dev/chainguard/nginx:latest
      ports:
        - containerPort: 8080
          protocol: TCP
      readinessProbe:
        httpGet:
          path: /
          port: 8080
      resources:
        limits:
          memory: "50Mi"
        requests:
          memory: "25Mi"
          cpu: "250m"
      securityContext:
        allowPrivilegeEscalation: false
        privileged: false
  securityContext:
    seccompProfile:
      type: RuntimeDefault
    runAsUser: 1001
    runAsGroup: 1001
    runAsNonRoot: true
```

```bash
kubectl apply -n ${NAMESPACE} -f Pod.nginx-better.yaml
kubectl port-forward -n ${NAMESPACE} nginx-better 8080:8080
```

Deleting namespace, deletes all resources within it
```bash
kubectl delete -f Namespace.yaml
```

#kubernetes-resources #definitions 
