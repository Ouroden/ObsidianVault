[[kubernetes-resource]]

Enables environment specific configuration to be decoupled from container images.
Two styles:
 - property like: MY_ENV_VAR = "MY_VALUE"
 - file like: conf.yaml = \<multi-line-string>

Namespace.yaml
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: 04--configmap
```

ConfigMap.file-like-keys.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: file-like-keys
data:
  conf.yml: |
    name: YourAppName
    version: 1.0.0
    author: YourName
```

ConfigMap.property-like-keys.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: property-like-keys
data:
  NAME: YourAppName
  VERSION: 1.0.0
  AUTHOR: YourName
```

Pod.configmap-example.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-example
spec:
  containers:
    - name: nginx
      image: nginx:1.26.0
      volumeMounts:
        - name: configmap-file-like-keys
          mountPath: /etc/config
      envFrom:
        - configMapRef:
            name: property-like-keys
  volumes:
    - name: configmap-file-like-keys
      configMap:
        name: file-like-keys
```

Setup
```bash
kubectl apply -f Namespace.yaml && setNamespace; kubens $NAMESPACE
kubectl apply -f ConfigMap.file-like-keys.yaml
kubectl apply -f ConfigMap.property-like-keys.yaml
kubectl apply -f Pod.configmap-example.yaml
kubectl delete -f Namespace.yaml
```

Commands
```bash
(devbox) [root@WINDOWS-ELLJEC8 ConfigMap]# k get pods
NAME                READY   STATUS    RESTARTS   AGE
configmap-example   1/1     Running   0          32s
# kubectl get configmaps
(devbox) [root@WINDOWS-ELLJEC8 ConfigMap]# k get cm
NAME                 DATA   AGE
file-like-keys       1      41s
kube-root-ca.crt     1      2m48s
property-like-keys   3      40s
(devbox) [root@WINDOWS-ELLJEC8 ConfigMap]# kubectl exec configmap-example -c nginx -- cat /etc/config/conf.yml
name: YourAppName
version: 1.0.0
author: YourName
(devbox) [root@WINDOWS-ELLJEC8 ConfigMap]# kubectl exec configmap-example -c nginx -- printenv
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=configmap-example
NGINX_VERSION=1.26.0
NJS_VERSION=0.8.4
NJS_RELEASE=2~bookworm
PKG_RELEASE=1~bookworm
AUTHOR=YourName
NAME=YourAppName
VERSION=1.0.0
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
HOME=/root
```

#kubernetes-resources #definitions 