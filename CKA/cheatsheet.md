# cheatshet for CKA

```bash
# exports for quick access
export DR="--dry-run=client"
export DRO="--dry-run=client -o yaml"
export NH="--no-headers"
export C="wc -l"
export D="describe"

# create a pod WITH a service via expose flag
kubectl run nginx --image=nginx --port=80 --expose
# remove a taint on a node with "-" postfix
kubectl taint nodes node1 key1=value1:NoSchedule-
# for deployments or daemonset, generate from 
k create deploy nginx --image=nginx --dry-run=client -o yaml > nginx-deploy.yaml
# creating static pods
 k run --restart=Never --image=busybox static-busybox --dry-run=client -o yaml --command -- sleep 1000 > static-busybox.yaml
# to view events
k get events -o wide

# view cluster configs
k config view
# tagerted view 
k config view --kubeconfig=my-config
# get contexts
k config get-contexts
# get current context
k config current-context
# set contexts 
k config use-context <context>
# get clusters
k config get-clusters
# set namespaces
k config set-context -current -namespace=dev

# view shortnames etc
k api-resources
```
