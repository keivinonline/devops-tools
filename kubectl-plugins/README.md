# kubectl plugins
- list of useful kubectl plugins
- requires `krew` to be installed via https://krew.sigs.k8s.io/docs/user-guide/setup/install/
## helpful aliases
```bash
alias kc='kubectl-ctx'
alias ks='kubectl-ns'
alias k='kubectl'
alias kg='kubectl get'
```

# kubectx
- switch between kubernetes contexts
```bash
kubectl krew install ctx
```
# kubens
- switch between kubernetes namespaces
```bash
kubectl krew install ns
```