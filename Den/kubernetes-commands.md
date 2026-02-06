```bash

kubectl <VERB> <NOUN> -n <NAMESPACE> -o <FORMAT>

alias k='kubectl'
kubectx
kubens # set default namespace

k logs <POD_NAME>
k logs deployment/<DEPLOYMENT_NAME>

k exec -it <POD_NAME> -c <CONTAINER_NAME> -- bash
k debug -it <POD_NAME> --image <DEBUG_IMAGE> -- bash

k port-forward <POD_NAME> <LOCAL_PORT>:<POD_PORT>
k port-forward svc/<DEPLOYMENT_NAME> <LOCAL_PORT>:<POD_PORT>

k get pods -n 04-pod
k get pods -A # (--all-namespaces)
k get pods -l key=value
k get pods -o wide # more info

k explain <NOUN>.path.to.field
k explain pod.spec.containers.image
k explain job.spec.backoffLimit
```


#kubernetes #cheat-sheet #commands 
