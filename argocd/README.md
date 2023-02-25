# argocd
- part of k8s cluster
- pulls k8ss manifest changes and applies them to the cluster
# CD workflow
1. deploy ArgoCD in a k8s cluster
2. configure ArgoCD to track Git repo
3. ArgoCD monitors for changes and applies them
# Manifests Repo
- separate git for
    - application source code
    - application configuration (k8s manifests)
- even separate git for system configs
# K8s manifests
- types of manifests
    1. K8s YAML files
    2. helm charts
    3. kustomize 
# Auto vs Manual
- can be configured to auto sync
- can be configured to manual sync and send alerts on changes
## K8s Access control
- manage cluster access via changes in Git
- no need for external cluster access to non-human users as argocd is in the cluster
- no cluster credentials outside of k8s
# ArgoCD as k8s extension
- uses existing k8s functionalities
    - etcd for data storage
    - k8s controllers for monitoring and comparing actual and desired state
- real-time updates of k8s cluster
# Configure ArgoCD
1. deploy ArgoCD into k8s cluster
2. Configure ArgoCD with k8s yaml file
- which git repo?
- which k8s cluster?
# Working with multiple clusters
1. 1 ArgoCD instance per cluster per environment
- but can be sourced from same git repo
2. using overlays with kustomize to change config per environment

## ArgoCD setup
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
# instal argocd cli
brew install argocd
# Expose the Argo CD API server - not exposed by default
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
k8s get svc argocd-server -n argocd
# get svc info
describe svc argocd-server
# Port forward the Argo CD API server (temp)
kubectl port-forward svc/argocd-server -n argocd 8080:443
# browse to https://127.0.0.1:8080
k8s get secret argocd-initial-admin-secret -n argocd -o yaml
# decode the base64
echo <hash> | base64 -D
# get default internal dns - kubernetes.default.svc is the default
kubectl get svc kubernetes -n default
```
## ArgoCD defaults
```yaml
  syncPolicy:
    syncOptions:
    # Automatically create namespaces for applications (off by default) via
    - CreateNamespace=true
    # pulling in 3 mins intervals
    automated:
    # changes made to live cluster will not trigger auto sync by default
      selfHeal: true
    # by default, auto sync will not delete resources
      prune: true
```

## test making manual changes
```bash
k8s edit deployment -n myapp myapp-deployment
```

```
## Todo
- [ ] setup ingress for argocd UI 
## port, targetPort and nodePort
- port: the port that the service exposes
- targetPort: the port that the pod exposes
- nodePort: the port that the node exposes