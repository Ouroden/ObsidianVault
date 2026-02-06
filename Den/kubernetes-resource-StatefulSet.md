[[kubernetes-resource]]

Workload type, similar to Deployment except:
 - design for stateful workflows
 - pods get sticky identity: pod-0, pod-1 etc. instead of random hash
 - within Deployment with persistent volume each Replica share the single volume, however in StatefulSet each Pod have separate volume
 - rollout behavior is ordered from 0 instead of random
 Enables configuring workload that require state management e.g. primary vs read-replica for DB.

Namespace.yaml
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: 04--statefulset
```

Service.nginx.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx # singular since it points to a single cluster IP
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

Service.nginxs.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginxs # Plural because it sets up DNS for each replica in the StatefulSet (e.g. nginx-0.nginxs.default.svc.cluster.local)
spec:
  type: ClusterIP
  clusterIP: None # This makes it a "headless" service
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

StatefulSet.nginx-minimal.yaml
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-minimal
spec:
  serviceName: nginxs
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.26.0
```

StatefulSet.nginx-with-init-container.yaml
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-with-init-conainer
spec:
  serviceName: nginxs
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      initContainers:
        - name: populate-default-html
          image: nginx:1.26.0
          # Nginx is a silly example to use for a stateful application (you should use a deployment for nginx)
          # but this demonstrates how you can use an init container to pre-populate a pod specific config file
          # For example, you might configure a database StatefulSet with some pods having read/write access, and
          # others only providing read access.
          #
          # See: https://kubernetes.io/docs/tasks/run-application/run-replicated-stateful-application/
          command:
            - bash
            - "-c"
            - |
              set -ex
              [[ $HOSTNAME =~ -([0-9]+)$ ]] || exit 1
              ordinal=${BASH_REMATCH[1]}
              echo "<h1>Hello from pod $ordinal</h1>" >  /usr/share/nginx/html/index.html
          volumeMounts:
            - name: data
              mountPath: /usr/share/nginx/html
      containers:
        - name: nginx
          image: nginx:1.26.0
          volumeMounts:
            - name: data
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: "standard"
        resources:
          requests:
            storage: 100Mi
```


Setup
```bash
kubectl apply -f Namespace.yaml && setNamespace; kubens $NAMESPACE
kubectl apply -f Service.nginx.yaml
kubectl apply -f Service.nginxs.yaml
kubectl apply -f StatefulSet.nginx-minimal.yaml
kubectl delete -f StatefulSet.nginx-minimal.yaml
kubectl apply -f Service.nginx.yaml
kubectl apply -f Service.nginxs.yaml
kubectl apply -f StatefulSet.nginx-with-init-container.yaml
kubectl delete -f Namespace.yaml
```

Commands
```bash
(devbox) [root@WINDOWS-ELLJEC8 StatefulSet]# k get pods
NAME                         READY   STATUS    RESTARTS   AGE
nginx-with-init-conainer-0   1/1     Running   0          29s
nginx-with-init-conainer-1   1/1     Running   0          23s
nginx-with-init-conainer-2   1/1     Running   0          17s

(devbox) [root@WINDOWS-ELLJEC8 StatefulSet]# k get svc
NAME     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
# will load balance across all replicas
nginx    ClusterIP   10.96.152.182   <none>        80/TCP    72s
# because of "clusterIP: None" setup DNS to allow connecting to pods independentely
nginxs   ClusterIP   None            <none>        80/TCP    72s


```

#kubernetes-resources #definitions 