[[kubernetes-resource]]

Used for long running stateless applications. 
Used for application running to completion.
Adds concept of tracking "completions".



Namespace.yaml
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: 04--job
```

Pod.echo-date-minimal.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: echo-date-minimal
spec:
  containers:
    - name: echo
      image: busybox:1.36.1
      command: ["date"]
  restartPolicy: Never
```

Job.echo-date-minimal.yaml
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: echo-date-minimal
spec:
  template:
    spec:
      containers:
        - name: echo
          image: busybox:1.36.1
          command: ["date"]
      restartPolicy: Never
  backoffLimit: 1
```

Job.echo-date-better.yaml
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: echo-date-better
  namespace: 04--job
spec:
  parallelism: 2
  completions: 2
  activeDeadlineSeconds: 100
  backoffLimit: 1
  template:
    metadata:
      labels:
        app: echo-date
    spec:
      containers:
        - name: echo
          image: cgr.dev/chainguard/busybox:latest
          command: ["date"]
          resources:
            limits:
              memory: "50Mi"
            requests:
              memory: "50Mi"
              cpu: "250m"
          securityContext:
            allowPrivilegeEscalation: false
            privileged: false
            runAsUser: 1001
            runAsGroup: 1001
            runAsNonRoot: true
      restartPolicy: Never
      securityContext:
        seccompProfile:
          type: RuntimeDefault
```

Setup
```bash
kubectl apply -f Namespace.yaml && setNamespace; kubens $NAMESPACE
kubectl apply -f Pod.echo-date-minimal.yaml
kubectl apply -f Job.echo-date-minimal.yaml
kubectl apply -f Job.echo-date-better.yaml
kubectl delete -f Namespace.yaml
```

Commands
```bash
(devbox) [root@WINDOWS-ELLJEC8 Job]# k get pods
NAME                READY   STATUS      RESTARTS   AGE
echo-date-minimal   0/1     Completed   0          10s

(devbox) [root@WINDOWS-ELLJEC8 Job]# k logs echo-date-minimal
Sat Nov 22 11:31:54 UTC 2025


(devbox) [root@WINDOWS-ELLJEC8 Job]# k get jobs
NAME                STATUS     COMPLETIONS   DURATION   AGE
echo-date-minimal   Complete   1/1           7s         14s

(devbox) [root@WINDOWS-ELLJEC8 Job]# k get pods
NAME                      READY   STATUS      RESTARTS   AGE
echo-date-minimal         0/1     Completed   0          3m48s # from 1 example
echo-date-minimal-zzj78   0/1     Completed   0          9s


(devbox) [root@WINDOWS-ELLJEC8 Job]# k get jobs
NAME                STATUS     COMPLETIONS   DURATION   AGE
echo-date-better    Complete   2/2           6s         4m8s
echo-date-minimal   Complete   1/1           7s         16m

(devbox) [root@WINDOWS-ELLJEC8 Job]# k get pods
NAME                      READY   STATUS      RESTARTS   AGE
echo-date-better-b7vd7    0/1     Completed   0          4m14s # two due to parallelism
echo-date-better-xwwwx    0/1     Completed   0          4m14s #
echo-date-minimal         0/1     Completed   0          20m
echo-date-minimal-zzj78   0/1     Completed   0          16m

```



#kubernetes-resources #definitions 