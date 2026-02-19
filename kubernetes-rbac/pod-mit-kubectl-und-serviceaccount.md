# Pod mit kubectl und ServiceAccount 

## Prerequisites 

  * Service-Account mit Rechten muss angelegt sein 

## Walkthrough 

```
cd 
mkdir -p manifests/rbac
cd manifests/rbac
nano pod.yaml
```


```
apiVersion: v1
kind: Pod
metadata:
  name: tools-pod
spec:
  serviceAccountName: training<nr>
  containers:
  - name: tools
    image: alpine/k8s:1.35.0
    command: ["sleep", "infinity"]
```

```
kubectl apply -f .
```

```
kubectl exec -it tools-pod -- sh
```

```
# Achtung der pod muss das kommando kubectl drin haben
kubectl cluster-info
kubectl get pods
# aus diesem pod heraus einen anderen Pod starten 
kubectl run my-nginx --image=nginx:1.23 
```

## Teardown 

```
kubectl delete -f pod.yaml
```
