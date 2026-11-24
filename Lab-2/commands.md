# Experiment 2 - Kubernetes Deployment, Scaling and Service

## Deployment

```powershell
kubectl create deployment web --image=nginx --replicas=3
kubectl get pods -o wide
kubectl get pods -w
kubectl delete pod web-5fc9f4bf66-2btn7
```

## Scaling

```powershell
kubectl scale deployment web --replicas=5
kubectl get pods
kubectl scale deployment web --replicas=2
kubectl get pods
kubectl get deployment web
kubectl get deployments
```

## Service

```powershell
kubectl get services
kubectl expose deployment web --type=NodePort --port=80
kubectl get service web
kubectl describe service web
kubectl get endpoints web
kubectl port-forward service/web 8080:80
kubectl get pods -o wide
```
