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


## get ingress
k get ingress
## describe to see paths etc
k describe ingress <name>
# imperative create ingress
k create ingress <name> --rule="host/path=service:port"
# example 
k create ingress ingress-test --rule="mystore.com/checkout*=checkout-service:8080"

# expose a deployment with a service
k expose deploy ingress-controller -n ingress-space \
--name ingress --port=80 --target-port=80 --type=NodePort

# install specific version of kubeadm and etc
sudo apt-get install kubeadm=1.26.0-00 kubelet=1.26.0-00 kubectl=1.26.0-00

# quickly debug a pod via logs 
k get event  --field-selector involvedObject.name=<pod-name>



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
- for nginx-ingress, if the web service does not have a path e.g. `/pay`, we need to annotate the ingress to use rewrite to match the path of e.g. `/path -> /` from ingress to service
```yaml
...
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/sssl-redirect: "false" # if we want to redirect to https
```



## Reads
- Network policy https://github.com/networkpolicy/tutorial#what-is-stateful-policy-enforcement 
## things to revise on 
- expose command


## questions
1. static pod - need to create in manifest dir 
```bash
kubectl run --restart=Never --image=busybox static-busybox --dry-run=client -oyaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml
```
2. exposing a service within cluster
```bash
#Create a service messaging-service to expose the messaging application within the cluster on port 6379.
kubectl expose pod messaging --port=6379 --name messaging-service
```
3. expose a nodeport 
```bash
# Expose the hr-web-app as service hr-web-app-service application on port 30082 on the nodes on the cluster.

# The web application listens on port 8080.

kubectl expose deployment hr-web-app --type=NodePort --port=8080 --name=hr-web-app-service --dry-run=client -o yaml > hr-web-app-service.yaml to generate a service definition file.

#Now, in generated service definition file add the nodePort field with the given port number under the ports section and create a service.
```
4. creating a pv with hostPath
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-analytics
spec:
  capacity:
    storage: 100Mi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  hostPath:
      path: /pv/data-analytics
```
5. etcd backup
```bash
# check etcd manifest file
cat /etc/kubernetes/manifests/etcd.yaml
ETCDTL_API=3 etcdctl \
endpoints=http://127.0.0.1:2379 \
--cacert=/etc/kubernetes/manifests/etcd/ca.crt \
--cert=/etc/kubernetes/manifests/etcd/server.crt \
--key=/etc/kubernetes/manifests/etcd/server.key \
snapshot save xxxx.db
```
5. We have deployed a new pod called np-test-1 and a service called np-test-service. Incoming connections to this service are not working. Troubleshoot and fix it.
Create NetworkPolicy, by the name ingress-to-nptest that allows incoming connections to the service over port 80.

Important: Don't delete any current objects deployed.
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ingress-to-nptest
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: np-test-1 # depends on labels
  policyTypes:
    - Ingress
  ingress:
  - ports:
    - protocol: TCP
      port: 80
```
6. taint a node and set tolerations to a pod
```bash
# taint
k taint nodes node01 env_type=production:NoSchedule
```
```yaml
...
spec:
  tolerations:
  - key: "env_type"
    operator: "Equal"
    value: "production"
    effect: "NoSchedule"
```
1) install etcd
sudo apt install etcd-client

```
cluster2-controlplane ~ ➜ cd /tmp
cluster2-controlplane ~ ➜ export RELEASE=$(curl -s https://api.github.com/repos/etcd-io/etcd/releases/latest | grep tag_name | cut -d '"' -f 4)
cluster2-controlplane ~ ➜ wget https://github.com/etcd-io/etcd/releases/download/${RELEASE}/etcd-${RELEASE}-linux-amd64.tar.gz
cluster2-controlplane ~ ➜ tar xvf etcd-${RELEASE}-linux-amd64.tar.gz ; cd etcd-${RELEASE}-linux-amd64
cluster2-controlplane ~ ➜ mv etcd etcdctl  /usr/local/bin/
```
7. get top pod usage 
```bash
kubectl top pods -A --context cluster1 --no-headers | sort -nr -k3 | head -1
```
8. always add a `context` flag to scripts
```bash
kbectl --context cluster1 get service service-cka25-arch -o jsonpath='{.spec.ports[0].targetPort}'
```
9. mysql needs min 512 MB of ram
10. endpoints
```yaml
apiVersion: v1
kind: Endpoints
metadata:
  # the name here should match the name of the Service
  name: external-webserver-cka03-svcn
subsets:
  - addresses:
      - ip: $IP_ADDR
    ports:
      - port: 9999
```
```bash
k get endpoints
```
11. match Namespace + label
```yaml
spec:
  egress:
  - {}
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: cyan-white-cka28-trb
      namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: default
```



## todos
```bash
# setup alias
API="kubectl api-resources"
OUT=" --dry-run=client -o yaml"
DRY=" --dry-run=client"

# shortcut to get json paths 
k get nodes -o json |jq -c 'paths' |grep -i xx

# test internal services
k run curl --image=alpine/curl --rm -it -- sh
```


## killer.sh
7. show node and pod processes
```bash
# show nodes 
kubectl top node
# show pods and container processes
kubectl top pod --containers=true
```
12. ensure only 1 pod runs on 1 node at a time via antiaffinity
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    id: very-important                  # change
  name: deploy-important
  namespace: project-tiger              # important
spec:
  replicas: 3                           # change
  selector:
    matchLabels:
      id: very-important                # change
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        id: very-important              # change
    spec:
      containers:
      - image: nginx:1.17.6-alpine
        name: container1                # change
        resources: {}
      - image: kubernetes/pause         # add
        name: container2                # add
      affinity:                                             # add
        podAntiAffinity:                                    # add
          requiredDuringSchedulingIgnoredDuringExecution:   # add
          - labelSelector:                                  # add
              matchExpressions:                             # add
              - key: id                                     # add
                operator: In                                # add
                values:                                     # add
                - very-important                            # add
            topologyKey: kubernetes.io/hostname             # add
```
14. get cluster info
```bash
# check service range 
ssh controlplane-node
cat /etc/kubernetes/manifests/kube-apiserver.yaml |grep -i range

# get CNI plugin info
cd /etc/cni/net.d/
cat <file_name>.conflist

# Which suffix will static pods have that run on cluster1-node1?
-cluster1-node1
```
15. get events
```bash
# get events sort by metadata.creationTimestamp
kubectl get events -A --sort-by=.metadata.creationTimestamp
```
16. get namespaced api-resources
```bash
kubectl api-resources --namespaced -o name
# get count of roles
k get roles --no-headers | wc -l
```