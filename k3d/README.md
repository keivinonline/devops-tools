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
k3d cluster create "kvcluster" -p "8080:80@loadbalancer" --agents 2
```
