# k3s
- [k3s](https://k3s.io/)
- Lightweight Kubernetes distribution
```bash
curl -sfL https://get.k3s.io | sh - 
# Check for Ready node, takes ~30 seconds 
k3s kubectl get node 
```
# k3d 
- dockerized k3s
```bash
# Install k3d
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
# Create cluster
k3d cluster create "homelab" -p "8080:80@loadbalancer" --agents 2
# Getting info 
k3d cluster list
k3d node list
# Adding nodes
k3d node create myagent --role agent --cluster lb-cluster
# removing nodes
k3d node delete k3d-myagent-0


```

# k8s commands
```bash
# get context
k8s config current-context
# set namespace
k8s config set-context --current --namespace=dev

```

# high level steps
- k3d simplifies deployments of k3s clusters
- once a cluster is setup, it exposes a k8s api endpoint where kubectl can be used to manage the cluster
- from here, helm charts can be deployed (e.g. argocd)
