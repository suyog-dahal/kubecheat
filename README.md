## Ports Required
# IMPORTANT!!!! If 8472/udp is not opened network (internet) is done for in flannel !!!!
```
Master: 6443/tcp 2379/tcp 2380/tcp 10250/tcp 10251/tcp 10252/tcp 10255/tcp 80/tcp 443/tcp 8285/tcp 8443/tcp 8472/udp
OS: 10250/tcp 30000-32767/tcp 10255/tcp 8472/udp 8443/tcp
```
## Kubernetes Installation Basics
```
- The config file in  the .kube directory is copied from /etc/kubernetes/config file
- The Path /etc/kubernetes/manifests contains the yml for kube-proxy, schedular, etcd and other yaml file. This folder is read during kubeadm init which is   used to run the pods of etcd and other.  
- NOTE: While creating secrets the secrets should be created under a desired namespace since the secrets created has a local scope and can be used only in the desired namespace. 
```

## Regeneration a join token for the worker node
```
kubeadm token generate -> This will generate the token for creating a join command
kubeadm token create [token] --print-join-command -> This will print the new join command with new token valid for 24hrs
```

## Get pod and Cluster information
```
kubectl get pod cid network
kubectl cluster-info dump | grep -m 1 cluster-cidr
```
## ConfigMap Settings
```
kubectl get config map of kubeadm
kubectl -n kube-system get cm kubeadm-config -o yaml
```
## Generate worker token
```
# Create a new token for worker to join with(expiry 2 hours)
Kubeadm create token

# Create a new token and join token 
kubeadm token create --print-join-command
```

## Ingress
# Error Findings
```
Error: Error from server (InternalError): error when creating "nginx-ingress.yaml": Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io": failed to call webhook: Post "https://ingress-nginx-controller-admission.ingress-nginx.svc:443/networking/v1/ingresses?timeout=29s": context deadline exceeded
```
# Resolution
``` 
1. kubectl get validatingwebhookconfigurations 
2. kubectl delete validatingwebhookconfigurations [configuration-name]
```
# Routing the external comming traffic to multiple worker nodes in Ingress controller 
```
ExternalTrafficPolicy is required in "Ingress controller service" for routing the ingress traffic to the module.
Example
   externalIPs:
    - 213.23.12.42
    externalTrafficPolicy: Cluster
```
## Terminologies
```
kubernetes service is used to cover up the issue of dynamic ip of pods. so when service is used even if the ip is changes the pods do not get affected.

kubernetes ingress is used to tell where the traffic should be directed.

kubernetes deployment is used for deploying unstateful things like application.
 
kubernetes statefulsets is used for deploying stateful things like databases where data is stateful(should be in sync)

kubernetes deployment is used to manage replicasets, replicasets is used to manage pods and pods manages container.
```

## Create and Edit Deployment
```
kubectl create deployment nginx-depl --image=nginx
kubectl edit deployment nginx-depl
```
# ex: Deployment kind yaml
``` 
Configuration file has 3 part:
1. metadata
2. specification
3. status
```
## Deployment Yaml Template
```
apiVersion: apps/v1 -> for each component such as service deployment etc there is a different api version
kind: Deployment -> spec attributes depend on the kind
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec: -> deployment specification
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec: -> container specification
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 80
```
## Service YAML Template
```
without nodeport or ingress or loadbalance cannot expose to internet        
service kind
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```    
# Now with nodeport
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  externalIPs:
  - 123.34.12.41.34 -> must provide for service to get an external ip
  selector:
    app: nginx
  ports:
  - name: nginx
    nodePort: 31716 -> nodeport
    protocol: TCP
    port: 80
    targetPort: 80 
  type: LoadBalancer -> Must use loadbalancer
```

## To check if right pod is used by service
``` 
kubectl describe svc nginx-service
Name:              nginx-service
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=nginx
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                15.10234.1147.211
IPs:               15.10234.1147.211
Port:              <unset>  80/TCP
TargetPort:        8080/TCP
Endpoints:         15.10234.1147.211:8080
Session Affinity:  None
Events:            <none>
```
## create secret for docker registry.
```
kubectl create secret docker-registry jcr-credentials --docker-server=abc.com --docker-username=abc --docker-password=test -n application-environment
```

## create configMap
```
kubectl create cm application-environment -n application --from-file=env 
```

## Create secret for password for db (sets to base64)
```
kubectl create secret generic my-secret --from-literal=password=abc
```

## Create namespace in Kubernetes
```
kubectl create namespace THISNAME
```

## Removing old replicasets (Defaults to 10 old Replica Sets)
```
Removing old replicasets is part of the Deployment object, but it is optional. You can set .spec.revisionHistoryLimit to tell the Deployment how many old replicasets to keep around.
Here is a YAML example:
apiVersion: apps/v1
kind: Deployment
spec:
  revisionHistoryLimit: 0 # Default to 10 if not specified
  
```

## Adding the host alias in the container:Put These in the container spec
```
hostAliases:
      - hostnames:
        - abc.com
        - www.cde.com
        - check-sum.org
        ip: IP_TO_MAP_TO
```

## Kubernetes Delete command
### Delete all resources in a namespace at once
```
kubectl delete all --all -n kubernetes-dashboard
```

### FACTS about HPA
```You target the deployment file for horizontal scaling.```
### Adhoc HPA command
```  kubectl autoscale deployment [Deploymen-Name] --cpu-percent=80 --min=1 --max=1 ```
## HPA YAML File
```
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  annotations:
  name: testhpa
  namespace: namespace
spec:
  maxReplicas: 5
  minReplicas: 2
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpa-deployment
  targetCPUUtilizationPercentage: 80
```

## Checking the resources of pods with hpa(horizontal pod autoscaler)
``` kubectl get hpa -n namespace --watch ```
## for single deployment hpa resource check
```kubectl describe hpa -n [namespace] [hpa name]```
## Rollout Deployement 
``` kubectl rollout restart deploy [deployment_name] -n [namespace]```
## Rollout History check
``` kubectl rollout history -n [namespace] deploy [deployment_name] ```
## Rollout undo 
``` kubectl rollout undo -n [namespace] deploy [deployment_name] ```
## Rollout undo to a particular revision [Note: you must annotate deployment with change-cause annotation ]
``` kubectl rollout undo -n [namespace] deploy [deployment_name] --to-revision=25 ```
## Annotating a deployment for rollout with change-cause
``` kubectl annotate deployments.apps [deployment_name] kubernetes.io/change-cause="blabla"```
## To use top command for pods in kubectl must install metric api, below link contains the solution to metric api installation 
```https://stackoverflow.com/questions/52694238/kubectl-top-node-error-metrics-not-available-yet-using-metrics-server-as-he```
## Kubernetes Official Cheatsheet Link
```https://kubernetes.io/docs/reference/kubectl/cheatsheet/```

# Kubernetes Certificates Cheatsheet
## Rotating kubectl CA certificate
### First check Cert Expiration
```kubeadm certs check-expiration```
### Rotate kubectl Cert Command
```kubeadm certs renew all```
