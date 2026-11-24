# Experiment 1 - Kubernetes Basics

## Cluster Setup

```powershell
notepad kind-3node.yaml
kind version
kind create cluster --config kind-3node.yaml
docker run hello-world
kubectl get nodes
docker ps
kubectl config use-context kind-klu
kubectl get nodes
```
