# Deployments

A Deployment provides declarative updates for Pods and ReplicaSets.

You describe a desired state in a Deployment, and the Deployment Controller changes the actual state to the desired state at a controlled rate. You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.


# Use Case

The following are typical use cases for Deployments:

- ***Create a Deployment to rollout a ReplicaSet.*** The ReplicaSet creates Pods in the background. Check the status of the rollout to see if it succeeds or not.
- ***Declare the new state of the Pods*** by updating the PodTemplateSpec of the Deployment. A new ReplicaSet is created and the Deployment manages moving the Pods from the old ReplicaSet to the new one at a controlled rate. Each new ReplicaSet updates the revision of the Deployment.
- ***Rollback to an earlier Deployment revision*** if the current state of the Deployment is not stable. Each rollback updates the revision of the Deployment.
- ***Scale up the Deployment to facilitate more load.***
- ***Pause the Deployment*** to apply multiple fixes to its PodTemplateSpec and then resume it to start a new rollout.
- ***Use the status of the Deployment*** as an indicator that a rollout has stuck.
- ***Clean up older ReplicaSets*** that you don’t need anymore.


## Creating a Deployment
```
# cd k8s-terraform-cgi-25May2020/03-K8s/

# cat 04-Deployment/helloworld.yml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: helloworld-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
      - name: k8s-demo
        image: amitvashist7/k8s-tiny-web
        ports:
        - name: node-port
          containerPort: 80
```

## Create the Deployment by running the following command:
```
kubectl create -f 04-Deployment/helloworld.yml
```
# Check the status of the Deployment using this command:

```
kubectl get deployment
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
helloworld-deployment   3/3     3            3           3m3s
```

```
kubectl get replicaset
NAME                               DESIRED   CURRENT   READY   AGE
helloworld-deployment-759fc84489   3         3         3       3m42s

```

```
kubectl get pods
NAME                                     READY   STATUS    RESTARTS   AGE
helloworld-deployment-759fc84489-94f2t   1/1     Running   0          4m8s
helloworld-deployment-759fc84489-ltldf   1/1     Running   0          4m8s
helloworld-deployment-759fc84489-pclvw   1/1     Running   0          4m8s
```

```
kubectl describe deployment helloworld-deployment
Name:                   helloworld-deployment
Namespace:              default
CreationTimestamp:      Wed, 27 May 2020 11:16:12 +0000
Labels:                 app=helloworld
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=helloworld
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  1 max unavailable, 1 max surge
Pod Template:
  Labels:  app=helloworld
  Containers:
   k8s-demo:
    Image:        amitvashist7/k8s-tiny-web
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   helloworld-deployment-759fc84489 (3/3 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  4m54s  deployment-controller  Scaled up replica set helloworld-deployment-759fc84489 to 3
```

## Let's Scaleout the Pods from 3 to 10.
```
kubectl scale --replicas=10 deployment helloworld-deployment
```

```
kubectl get pods
NAME                                     READY   STATUS    RESTARTS   AGE
helloworld-deployment-759fc84489-5n7vs   1/1     Running   0          48s
helloworld-deployment-759fc84489-5t8sj   1/1     Running   0          48s
helloworld-deployment-759fc84489-94f2t   1/1     Running   0          7m4s
helloworld-deployment-759fc84489-bzr75   1/1     Running   0          48s
helloworld-deployment-759fc84489-cpmdk   1/1     Running   0          48s
helloworld-deployment-759fc84489-ltldf   1/1     Running   0          7m4s
helloworld-deployment-759fc84489-nsrh9   1/1     Running   0          48s
helloworld-deployment-759fc84489-pclvw   1/1     Running   0          7m4s
helloworld-deployment-759fc84489-rb9nz   1/1     Running   0          48s
helloworld-deployment-759fc84489-w79wt   1/1     Running   0          48s
```

## Let's ScaleIn the Pods from 10 to 3.
```
kubectl scale --replicas=3 deployment helloworld-deployment
```

```
kubectl get pods
NAME                                     READY   STATUS    RESTARTS   AGE
helloworld-deployment-759fc84489-94f2t   1/1     Running   0          9m21s
helloworld-deployment-759fc84489-ltldf   1/1     Running   0          9m21s
helloworld-deployment-759fc84489-pclvw   1/1     Running   0          9m21s
```


## Let's try an detele the pod to check deployment in actions

```
kubectl delete pod helloworld-deployment-759fc84489-94f2t helloworld-deployment-759fc84489-lt ldf
pod "helloworld-deployment-759fc84489-94f2t" deleted
pod "helloworld-deployment-759fc84489-ltldf" deleted
```
```
kubectl get pods
NAME                                     READY   STATUS    RESTARTS   AGE
helloworld-deployment-759fc84489-g4szx   1/1     Running   0          25s
helloworld-deployment-759fc84489-m9v46   1/1     Running   0          25s
helloworld-deployment-759fc84489-pclvw   1/1     Running   0          10m
```

## Expose the deployment to outsider world

### Exposing the service on NodePort
```
kubectl expose deploy helloworld-deployment --type=NodePort
```

### Status of Deployment service exposed on NodePort
```
kubectl get svc
NAME                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
helloworld-deployment   NodePort    10.109.13.14   <none>        80:31702/TCP   81s
kubernetes              ClusterIP   10.96.0.1      <none>        443/TCP        5h55m
```

### Identify the Nodes where our deploument Pod is running
```
kubectl get pods -o wide
NAME                                     READY   STATUS    RESTARTS   AGE     IP             NODE      NOMINATED NODE   READINESS GATES
helloworld-deployment-759fc84489-g4szx   1/1     Running   0          9m12s   192.168.1.13   worker1   <none>           <none>
helloworld-deployment-759fc84489-m9v46   1/1     Running   0          9m12s   192.168.2.16   worker2   <none>           <none>
helloworld-deployment-759fc84489-pclvw   1/1     Running   0          19m     192.168.2.11   worker2   <none>           <none>
```

### Now we can access the pod with combination of Node/workers IP/Hostname/FQDN with NodePort : workerX:NodePort

```
curl worker1:31702
<html>
<body>
<h1>Hello World</h1>
<h3>Tiny Web Server</h3>
</body>
</html>
```
```
curl worker2:31702
<html>
<body>
<h1>Hello World</h1>
<h3>Tiny Web Server</h3>
</body>
</html>
```

## Rolling Updates

### Increase the Replicas 
```
kubectl scale --replicas=10 deployment helloworld-deployment
```

### Update the Container Image to KickOff Rolling Update
```
kubectl set image deployment helloworld-deployment k8s-demo=amitvashist7/k8s-tiny-web:2
```

### Check the rollout status 
```
kubectl rollout status  deployment helloworld-deployment
Waiting for deployment "helloworld-deployment" rollout to finish: 9 out of 10 new replicas have been updated...
Waiting for deployment "helloworld-deployment" rollout to finish: 9 out of 10 new replicas have been updated...
Waiting for deployment "helloworld-deployment" rollout to finish: 9 out of 10 new replicas have been updated...
Waiting for deployment "helloworld-deployment" rollout to finish: 9 out of 10 new replicas have been updated...
Waiting for deployment "helloworld-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "helloworld-deployment" rollout to finish: 1 old replicas are pending termination...
```


### New Set of Pods post RollOut
```
kubectl get pods
NAME                                     READY   STATUS    RESTARTS   AGE
helloworld-deployment-5d9488c877-j6q52   1/1     Running   0          29s
helloworld-deployment-5d9488c877-jrrbm   1/1     Running   0          16s
helloworld-deployment-5d9488c877-lh5n6   1/1     Running   0          35s
helloworld-deployment-5d9488c877-mwxbs   1/1     Running   0          19s
helloworld-deployment-5d9488c877-pf57f   1/1     Running   0          21s
helloworld-deployment-5d9488c877-qr8h9   1/1     Running   0          41s
helloworld-deployment-5d9488c877-s5wcs   1/1     Running   0          24s
helloworld-deployment-5d9488c877-skhrp   1/1     Running   0          41s
helloworld-deployment-5d9488c877-xcxl7   1/1     Running   0          26s
helloworld-deployment-5d9488c877-zf2x7   1/1     Running   0          34s
```

### Check the Applocation WebPage
```
root@master:~/k8s-terraform-cgi-25May2020/03-K8s/04-Deployment# curl worker1:31702
<html>
<body>
<h1>Hello World</h1>
<h3>Tiny Web Server</h3>
<h5>Version: v2</h5>
</body>
</html>
```

### Check the rollout history 
```
kubectl rollout history  deployment helloworld-deployment
deployment.extensions/helloworld-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         <none>

```


### Check the rollout history / Revsion Version Detials
```
 kubectl rollout history  deployment helloworld-deployment --revision=1
deployment.extensions/helloworld-deployment with revision #1
Pod Template:
  Labels:       app=helloworld
        pod-template-hash=759fc84489
  Containers:
   k8s-demo:
    Image:      amitvashist7/k8s-tiny-web
    Port:       80/TCP
    Host Port:  0/TCP
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>
```

```
kubectl rollout history  deployment helloworld-deployment --revision=2
deployment.extensions/helloworld-deployment with revision #2
Pod Template:
  Labels:       app=helloworld
        pod-template-hash=5d9488c877
  Containers:
   k8s-demo:
    Image:      amitvashist7/k8s-tiny-web:2
    Port:       80/TCP
    Host Port:  0/TCP
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>

```



### Create the rollout history Record
```
kubectl set image deployment helloworld-deployment k8s-demo=amitvashist7/k8s-tiny-web:4 --record
```

```
kubectl rollout history  deployment helloworld-deployment
deployment.extensions/helloworld-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         <none>
4         kubectl set image deployment helloworld-deployment k8s-demo=amitvashist7/k8s-tiny-web:4 --record=true
```


### Check the Applocation WebPage
```
curl worker1:31702
<html>
<body>
<h1>Hello World</h1>
<h3>Tiny Web Server</h3>
<h5>Version: v4</h5>
</body>
</html>
```


## Rollback 



## Rollback to revsion version
### Check the version history

### Revsion Version Record

## How to delete the Deployment
```
kubectl get rc
NAME                    DESIRED   CURRENT   READY   AGE
helloworld-controller   1         1         1       29m
```
```
kubectl delete rc helloworld-controller
replicationcontroller "helloworld-controller" deleted
```
