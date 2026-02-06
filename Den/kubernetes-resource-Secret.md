[[kubernetes-resource]]

Similar to ConfigMaps except:
 - Data is base64 encoded
 - Secrets can be managed with specific authorization policies

Namespace.yaml
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: 04--secret
```

Secret.string-data.yaml
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: string-data
type: Opaque
stringData:
  foo: bar
```

Secret.base64-data.yaml
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: base64-data
type: Opaque
data:
  foo: YmFy
```

Secret.dockerconfigjson.yaml
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: dockerconfigjson
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: |
    eyJhdXRocyI6eyJodHRwczovL2luZGV4LmRvY2tlci5pby92MSI6eyJ1c2VybmFtZSI6InVzZXJuYW1lIiwicGFzc3dvcmQiOiJwYXNzd29yZCIsImVtYWlsIjoiZm9vQGJhci5jb20iLCJhdXRoIjoiZFhObGNtNWhiV1U2Y0dGemMzZHZjbVE9In19fQ==
```

Pod.secret-example.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-example
spec:
  containers:
    - name: nginx
      image: nginx:1.26.0
      volumeMounts:
        - name: secret-base64-data
          mountPath: /etc/config
      env:
        - name: ENV_VAR_FROM_SECRET
          valueFrom:
            secretKeyRef:
              name: base64-data
              key: foo
  imagePullSecrets:
    # Not necessary since example uses a public image, but including to show how
    # you would use a registry credential secret to access a private image
    - name: dockerconfigjson
  volumes:
    - name: secret-base64-data
      secret:
        secretName: base64-data

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
(devbox) [root@WINDOWS-ELLJEC8 Secret]# k get secrets -o yaml | yq
apiVersion: v1
items:
  - apiVersion: v1
    data:
      foo: YmFy
    kind: Secret
    metadata:
      annotations:
        kubectl.kubernetes.io/last-applied-configuration: |
          {"apiVersion":"v1","kind":"Secret","metadata":{"annotations":{},"name":"string-data","namespace":"04--secret"},"stringData":{"foo":"bar"},"type":"Opaque"}
      creationTimestamp: "2025-11-24T12:31:03Z"
      name: string-data
      namespace: 04--secret
      resourceVersion: "24077"
      uid: ed02a8ac-db5a-4a6b-b4bd-a9a9eff93d63
    type: Opaque
kind: List
metadata:
  resourceVersion: ""
  
(devbox) [root@WINDOWS-ELLJEC8 Secret]# k get secrets string-data -o yaml | yq '.data.foo' | base64 -d
bar

(devbox) [root@WINDOWS-ELLJEC8 Secret]# kubectl create secret docker-registry dockerconfigjson \
          --docker-email=foo@bar.com \
          --docker-username=username \
          --docker-password=password \
          --docker-server=https://index.docker.io/v1/ --dry-run=client -o yaml | yq
apiVersion: v1
data:
  .dockerconfigjson: eyJhdXRocyI6eyJodHRwczovL2luZGV4LmRvY2tlci5pby92MS8iOnsidXNlcm5hbWUiOiJ1c2VybmFtZSIsInBhc3N3b3JkIjoicGFzc3dvcmQiLCJlbWFpbCI6ImZvb0BiYXIuY29tIiwiYXV0aCI6ImRYTmxjbTVoYldVNmNHRnpjM2R2Y21RPSJ9fX0=
kind: Secret
metadata:
  name: dockerconfigjson
type: kubernetes.io/dockerconfigjson

(devbox) [root@WINDOWS-ELLJEC8 Secret]# kubectl exec secret-example -c nginx -- printenv
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=secret-example
NGINX_VERSION=1.26.0
NJS_VERSION=0.8.4
NJS_RELEASE=2~bookworm
PKG_RELEASE=1~bookworm
ENV_VAR_FROM_SECRET=bar
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
HOME=/root

(devbox) [root@WINDOWS-ELLJEC8 Secret]# kubectl exec secret-example -c nginx -- cat /etc/config/foo
bar

```

#kubernetes-resources #definitions 