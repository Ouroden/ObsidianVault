```bash
(devbox) [root@WINDOWS-ELLJEC8 03-installation-and-setup]# kubectl get nodes
NAME                 STATUS   ROLES           AGE     VERSION
kind-control-plane   Ready    control-plane   5m54s   v1.30.0
kind-worker          Ready    <none>          5m33s   v1.30.0
kind-worker2         Ready    <none>          5m35s   v1.30.0
```

```
```bash
(devbox) [root@WINDOWS-ELLJEC8 03-installation-and-setup]# kubectl get pods -A
NAMESPACE            NAME                                         READY   STATUS    RESTARTS   AGE
kube-system          coredns-7db6d8ff4d-gw6r8                     1/1     Running   0          8m10s
kube-system          coredns-7db6d8ff4d-qslrv                     1/1     Running   0          8m10s
kube-system          etcd-kind-control-plane                      1/1     Running   0          8m26s
kube-system          kindnet-cht52                                1/1     Running   0          8m9s
kube-system          kindnet-fn88h                                1/1     Running   0          8m7s
kube-system          kindnet-lmkhh                                1/1     Running   0          8m11s
kube-system          kube-apiserver-kind-control-plane            1/1     Running   0          8m26s
kube-system          kube-controller-manager-kind-control-plane   1/1     Running   0          8m26s
kube-system          kube-proxy-4ndzm                             1/1     Running   0          8m11s
kube-system          kube-proxy-hh4x6                             1/1     Running   0          8m9s
kube-system          kube-proxy-njxd2                             1/1     Running   0          8m7s
kube-system          kube-scheduler-kind-control-plane            1/1     Running   0          8m26s
local-path-storage   local-path-provisioner-988d74bc-npwzq        1/1     Running   0          8m10s
```

#kubernetes #commands