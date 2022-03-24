## APP Used
```
kubectx
kubens
```

## Commands
```
kubens
kubectl create namespace custom-default
kubens custom-default
```

## HELM
```
helm install --dry-run --debug . --generate-name (to test)
helm install name .
helm delete appname
```

## To save secret from kustomization.yaml and see status
```
kubectl apply -k ./
kubectl get secrets
```
## Verify that a PersistentVolume got dynamically provisioned
```
kubectl get pvc
```
## Namespaces and see error in container and delete
```
kubectl get deployments --all-namespaces
kubectl describe pvc <Name>
kubectl delete pvc <Name>
```
## Get pods and delete
```
kubectl get pods
kubectl delete <podname>
```
## Get Services and delete
```
kubectl get services
kubectl delete svc <servicename>
```
## To pach if stuck in terminating both pv and pvc
```
kubectl patch pvc <NAME> -p '{"metadata":{"finalizers":null}}'
kubectl patch pv <NAME> -p '{"metadata":{"finalizers":null}}'
```
## Debugging
```
kubectl get deployments <deployment-name> –o yaml
kubectl logs -n ingress-nginx ingress-nginx-controller-f9d6fc8d8-d7xv9 -f
```
## Inital Setup
```
kubectl get validatingwebhookconfigurations 
kubectl delete validatingwebhookconfigurations ingress-nginx-admission
```
