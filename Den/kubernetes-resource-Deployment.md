[[kubernetes-resource]]

Allows changing Pod specs e.g. updating image, modifying number of resources.
Add concept of "rollouts" and "rollbacks".
Used for long-running stateless applications.


Namespace.yaml
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: 04--deployment
```

Deployment.nginx-minimal.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-minimal
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-minimal
  template:
    metadata:
      labels:
        app: nginx-minimal
    spec:
      containers:
        - name: nginx
          image: nginx:1.26.0
```

Deployment.nginx-better.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-better
  namespace: 04--deployment
  labels:
    app: nginx-better
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-better
  template:
    metadata:
      labels:
        app: nginx-better
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
              memory: "50Mi"
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

Setup
```bash
kubectl apply -f Namespace.yaml
export NAMESPACE=$(grep name Namespace.yaml | cut -d" " -f4)
kubens $NAMESPACE
kubectl apply -f Deployment.nginx-minimal.yaml
kubectl apply -f Deployment.nginx-better.yaml
kubectl delete -f Namespace.yaml
```

Commands
```bash
(devbox) [root@WINDOWS-ELLJEC8 Deployment]# k get deployments.apps
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
nginx-better    3/3     3            3           3m30s
nginx-minimal   3/3     3            3           3m38s

(devbox) [root@WINDOWS-ELLJEC8 Deployment]# k get rs
NAME                       DESIRED   CURRENT   READY   AGE
nginx-better-69848d549d    3         3         3       4m39s
nginx-minimal-6666ccb9cc   3         3         3       4m47s

(devbox) [root@WINDOWS-ELLJEC8 Deployment]# k get pods
NAME                             READY   STATUS    RESTARTS   AGE
nginx-better-69848d549d-8sktv    1/1     Running   0          5m10s
nginx-better-69848d549d-clgcd    1/1     Running   0          5m10s
nginx-better-69848d549d-rmpml    1/1     Running   0          5m10s
nginx-minimal-6666ccb9cc-cnntc   1/1     Running   0          5m18s
nginx-minimal-6666ccb9cc-fhw5c   1/1     Running   0          5m18s
nginx-minimal-6666ccb9cc-l26qs   1/1     Running   0          5m18s
```


Rollout

```bash
# one way to run apply again to update pods specs
# it creates new ReplicaSets and new Pods
kubectl apply -f Deployment.nginx-better.yaml

# second is to issue rollout restart
# both cycle through pods one at a time
# creating new one then deleting old one
# it creates new ReplicaSet and leaving old one with 0 pods desired for rollback

kubectl rollout restart deployment nginx-better

# undo rollout leaving new ReplicaSet with 0 pods
kubectl rollout undo deployment nginx-better

# watch pods changes
watch "kubectl get pods"
```

#kubernetes-resources #definitions 