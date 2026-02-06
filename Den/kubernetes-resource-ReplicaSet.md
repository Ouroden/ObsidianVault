[[kubernetes-resource]]

Adds the concept of "replicas". Rarely created directly.
If number of requested pods is less than required it automatically created new pods.
Labels are the link between ReplicaSets and Pods.

Namespace.yaml
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: 04--replicaset
```

ReplicaSet.nginx-minimal.yaml
```yaml
apiVersion: apps/v1
kind: ReplicaSet
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

ReplicaSet.nginx-better.yaml
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-better
  namespace: 04--replicaset
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
kubens 04--replicaset
kubectl apply -f ReplicaSet.nginx-minimal.yaml
kubectl apply -f ReplicaSet.nginx-better.yaml
kubectl delete -f Namespace.yaml
```

Commands
```bash
kubectl get replicasets.apps
kubectl get rs # same as above

(devbox) [root@WINDOWS-ELLJEC8 ReplicaSet]# k get rs
NAME            DESIRED   CURRENT   READY   AGE
nginx-minimal   3         3         3       107s
```

#kubernetes-resources #definitions 