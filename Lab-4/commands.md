# Lab-4: Read the Scheduler and Find Why Pods Stay Pending

## Step 0: Check the cluster and Git

```powershell
kubectl config use-context kind-klu
kubectl get nodes
kubectl describe node klu-control-plane | Select-String "Taints"
git --version
```

## Step 1: Download only the scheduler source code

```powershell
cd $HOME
git clone --depth 1 --branch v1.37.0 --filter=blob:none --sparse https://github.com/kubernetes/kubernetes.git k8s-src
cd k8s-src
git sparse-checkout set pkg/scheduler staging/src/k8s.io/kube-scheduler
dir pkg\scheduler
```

## Step 2: Find the default plugins and weights

```powershell
notepad pkg\scheduler\apis\config\v1\default_plugins.go
Select-String -Path pkg\scheduler\apis\config\v1\default_plugins.go -Pattern "Name: names"
```

## Step 3: Find Filter, Score and PreFilter functions

```powershell
Get-ChildItem pkg\scheduler\framework\plugins -Recurse -Filter *.go -Exclude *_test.go | Select-String -Pattern "^func \(.*\) (Filter|Score)\("
Get-ChildItem pkg\scheduler\framework\plugins -Recurse -Filter *.go -Exclude *_test.go | Select-String -Pattern "^func \(.*\) PreFilter\("
Get-ChildItem pkg\scheduler\framework\plugins -Recurse -Filter *.go -Exclude *_test.go | Select-String -Pattern "^func \(.*\) Reserve\("
```

## Step 4: Follow one scheduling cycle

```powershell
Select-String -Path pkg\scheduler\schedule_one.go -Pattern "^func .*(ScheduleOne|schedulingCycle|bindingCycle|schedulePod|findNodesThatFitPod|prioritizeNodes)\("
```

## Step 5: Experiment A — too much CPU

```powershell
mkdir $HOME\lab4
cd $HOME\lab4
notepad big.yaml
kubectl apply -f big.yaml
kubectl get pod big
kubectl describe pod big
Select-String -Path $HOME\k8s-src\pkg\scheduler\framework\plugins\noderesources\fit.go -Pattern "Insufficient"
```

## Step 6: Experiment B — missing label / NodeAffinity

```powershell
notepad ssd.yaml
kubectl apply -f ssd.yaml
kubectl describe pod needs-ssd
kubectl label node klu-worker disktype=ssd
kubectl get pod needs-ssd -o wide
Select-String -Path $HOME\k8s-src\pkg\scheduler\framework\plugins\nodeaffinity\node_affinity.go -Pattern "didn't match"
```

## Step 7: Experiment C — taints and tolerations

```powershell
kubectl taint node klu-worker exam=running:NoSchedule
kubectl create deployment spread --image=nginx:1.27 --replicas=4
kubectl get pods -l app=spread -o wide
notepad spread-tol.yaml
kubectl apply -f spread-tol.yaml
kubectl get pods -l app=spread -o wide
kubectl taint node klu-worker exam=running:NoSchedule-
```

## Step 8: Experiment D — same host port

```powershell
notepad hostport.yaml
kubectl apply -f hostport.yaml
kubectl get pods -l app=hostport -o wide
kubectl describe pod <YOUR-ACTUAL-PENDING-HOSTPORT-POD-NAME>
Select-String -Path $HOME\k8s-src\pkg\scheduler\framework\plugins\nodeports\node_ports.go -Pattern "free ports"
```

> Use the actual Pending Pod name shown by `kubectl get pods -l app=hostport -o wide`. Do not copy the example Pod name from the lab record.

## Step 9: Experiment E — cordon the nodes

```powershell
kubectl cordon klu-worker
kubectl cordon klu-worker2
kubectl get nodes
kubectl run closed --image=nginx:1.27
kubectl describe pod closed
kubectl uncordon klu-worker
kubectl uncordon klu-worker2
kubectl get pod closed -o wide
```

## Step 10: Clean up

```powershell
kubectl delete pod big needs-ssd closed
kubectl delete deployment spread hostport
kubectl label node klu-worker disktype-
kubectl get pods
```

Keep the `k8s-src` folder for later labs.

## Lab-4 evidence

Screenshots were pasted into the completed Experiment 4 lab record document. The record includes:
- Cluster and control-plane taint
- Scheduler source inspection
- Default plugin list
- Filter/Score function search
- Scheduling-cycle functions
- Experiment A Pending event
- Experiment B Pending → Running after labeling
- Experiment C taint/toleration behavior
- Experiment D Pending hostPort event
- Experiment E cordon/uncordon behavior
- Cleanup and analysis/viva sections
