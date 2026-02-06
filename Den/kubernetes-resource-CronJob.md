[[kubernetes-resource]]

Add concept of a schedule.

Namespace.yaml
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: 04--cronjob
```

CronJob.echo-date-minimal.yaml
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: echo-date-minimal
spec:
  schedule: "* * * * *"
  jobTemplate:
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

CronJob.echo-date-better.yaml
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: echo-date-better
  namespace: 04--cronjob
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      parallelism: 1
      completions: 1
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
kubectl apply -f CronJob.echo-date-minimal.yaml
kubectl apply -f CronJob.echo-date-better.yaml
kubectl delete -f Namespace.yaml
# Manually create a job using a cronjob as the template
kubectl create job --from=cronjob/echo-date-better manually-triggered
```

Commands
```bash
(devbox) [root@WINDOWS-ELLJEC8 CronJob]# k get cronjobs
NAME               SCHEDULE    TIMEZONE   SUSPEND   ACTIVE   LAST SCHEDULE   AGE
echo-date-better   * * * * *   <none>     False     0        <none>          4s

(devbox) [root@WINDOWS-ELLJEC8 CronJob]# k get jobs
NAME                        STATUS     COMPLETIONS   DURATION   AGE
# created every minute
echo-date-better-29396884   Complete   1/1           3s         64s
echo-date-better-29396885   Complete   1/1           3s         4s 


(devbox) [root@WINDOWS-ELLJEC8 CronJob]# kubectl create job --from=cronjob/echo-date-better manually-triggered
job.batch/manually-triggered created
(devbox) [root@WINDOWS-ELLJEC8 CronJob]# k get jobs
NAME                        STATUS     COMPLETIONS   DURATION   AGE
echo-date-better-29396884   Complete   1/1           3s         108s
echo-date-better-29396885   Complete   1/1           3s         48s
manually-triggered          Running    0/1           1s         1s
```


#kubernetes-resources #definitions 