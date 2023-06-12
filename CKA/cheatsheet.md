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

# create the role based on the above yaml
k create -f developer-role.yaml
# link the role to a user
k create -f devuser-role-binding.yaml 
# view roles
k get roles
# get rolebindings
k get rolebindings
# describe role
k describe role developer
# decribe role bindings
k describe rolebindings devuser-role-binding
# check my user
k auth can-i create deployments
# impersonate other users
k auth can-i create pods --as dev-user --namespace test
# edit role on the fly
k edit role developer -n blue
# implicit role create
k create role developer \
--verb=list,create,delete \
--resource=pods
# implicit create rolebinding
k create rolebinding developer-binding \
--clusterrole=developer \
--user=developer

# see namespaced resources
k api-resources --namespaced=true
k api-resources --namespaced=false

# describe clusterrole
k describe  clusterrole argocd-server
k describe  clusterrolebinding argocd-server
# create clusterrole
k create clusterrole storage-admin --verb='*' --resource=persistentvolumes,storageclasses
# create clusterrolebinding
k create clusterrolebinding storage-admin-binding --clusterrole=storage-admin --user=storage-admin

k create sa my-service-account
k create token my-service-account
# create docker creds
k create secret docker-registry regcred \
--docker-server=<registry> \
--docker-username=<user> \
--docker-password=<password> \
--docker-email=<email>

# replace a resource with yaml
k replace -f pod.yaml --force

# network policies
k get netpol

## networking
# check service ip range
ps aux |grep -i kube-api-server
range="10.X.X.X/12" 




# view shortnames etc
k api-resources
# get help wit yaml values
k explain Pod.spec 
```
## Steps
- node interfaces
```bash
# get internal IP of node
k get nodes -o wide
# grep IP from ip a 
ip a |grep -i "<ip>"
```
- show bridge interface
```bash
ip a show type bridge
```
- check default routes
```bash
ip route
```
- check established sockets to etcd
```bash
netstat -npa |grep -i "etcd" |grep -i "established"
```
- check container flag on kubelet service 
```bash
ps aux |grep kubelet |grep container
```
- all plugins are stored in `/opt/cni/bin`
- all configs are stored in `/opt/cni/net.d`
- ensure a container lands on specific node 
```bash
k run busybox --image=busybox --dry-run=client -o yaml -- sleep 1000 > busybox.yaml
```
```yaml
...
spec:
  nodeName: node01
  containers:
  ...

```



## Reads
- Network policy https://github.com/networkpolicy/tutorial#what-is-stateful-policy-enforcement 