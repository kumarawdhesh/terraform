# ReplicationController

A ReplicationController ensures that a specified number of pod replicas are running at any one time. In other words, a ReplicationController makes sure that a pod or a homogeneous set of pods is always up and available.


## How a ReplicationController Works

If there are too many pods, the ReplicationController terminates the extra pods. If there are too few, the ReplicationController starts more pods. Unlike manually created pods, the pods maintained by a ReplicationController are automatically replaced if they fail, are deleted, or are terminated. For example, your pods are re-created on a node after disruptive maintenance such as a kernel upgrade. For this reason, you should use a ReplicationController even if your application requires only a single pod. A ReplicationController is similar to a process supervisor, but instead of supervising individual processes on a single node, the ReplicationController supervises multiple pods across multiple nodes.

## Running an example ReplicationController

```
# cd k8s-terraform-cgi-25May2020/03-K8s/03-Replication-controller

# cat helloworld-repl-controller.yml
apiVersion: v1
kind: ReplicationController
metadata:
  name: helloworld-controller
spec:
  replicas: 5
  selector:
    app: rc-scaling-pods
  template:
    metadata:
      labels:
        app: rc-scaling-pods
    spec:
      containers:
      - name: rc-scaling-pods
        image: amitvashist7/k8s-tiny-web
        ports:
        - name: nodejs-port
          containerPort: 80

```

# Run the helloworld RC with following command:

```
kubectl apply -f helloworld-repl-controller.yml
```

# Check on the status of the ReplicationController using this command:

```
kubectl get rc
NAME                    DESIRED   CURRENT   READY   AGE
helloworld-controller   5         5         2       8s
```

```
kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
helloworld-controller-27z74   1/1     Running   0          18s
helloworld-controller-f6dgr   1/1     Running   0          18s
helloworld-controller-k2snd   1/1     Running   0          18s
helloworld-controller-m86x5   1/1     Running   0          17s
helloworld-controller-nxnkg   1/1     Running   0          17s

```

```
kubectl describe rc helloworld-controller
Name:         helloworld-controller
Namespace:    default
Selector:     app=rc-scaling-pods
Labels:       app=rc-scaling-pods
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"v1","kind":"ReplicationController","metadata":{"annotations":{},"name":"helloworld-controller","namespace":"default"},"spec... Replicas:     1 current / 1 desired
Pods Status:  1 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=rc-scaling-pods
  Containers:
   rc-scaling-pods:
    Image:        amitvashist7/k8s-tiny-web
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age                From                    Message
  ----    ------            ----               ----                    -------
  Normal  SuccessfulCreate  23m                replication-controller  Created pod: helloworld-controller-k2snd
  Normal  SuccessfulCreate  23m                replication-controller  Created pod: helloworld-controller-f6dgr
  Normal  SuccessfulCreate  23m                replication-controller  Created pod: helloworld-controller-27z74
  Normal  SuccessfulCreate  23m                replication-controller  Created pod: helloworld-controller-nxnkg
  Normal  SuccessfulCreate  23m                replication-controller  Created pod: helloworld-controller-m86x5
  Normal  SuccessfulDelete  22m                replication-controller  Deleted pod: helloworld-controller-m86x5
  Normal  SuccessfulDelete  22m                replication-controller  Deleted pod: helloworld-controller-27z74
```

# Common usage patterns

## Rescheduling
As mentioned above, whether you have 1 pod you want to keep running, or 1000, a ReplicationController will ensure that the specified number of pods exists, even in the event of node failure or pod termination (for example, due to an action by another control agent).

## Scaling
The ReplicationController makes it easy to scale the number of replicas up or down, either manually or by an auto-scaling control agent, by simply updating the replicas field.

## Rolling updates
The ReplicationController is designed to facilitate rolling updates to a service by replacing pods one-by-one.


## Let's Scaleout the Pods from 5 to 10.
```
kubectl scale --replicas=10 rc helloworld-controller
```

```
kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
helloworld-controller-22prs   1/1     Running   0          15s
helloworld-controller-5bzp2   1/1     Running   0          15s
helloworld-controller-6jzv8   1/1     Running   0          15s
helloworld-controller-f6dgr   1/1     Running   0          117s
helloworld-controller-fclsh   1/1     Running   0          15s
helloworld-controller-k2snd   1/1     Running   0          117s
helloworld-controller-kt8w4   1/1     Running   0          15s
helloworld-controller-lz7j4   1/1     Running   0          15s
helloworld-controller-nxnkg   1/1     Running   0          116s
helloworld-controller-wj22v   1/1     Running   0          15s
```

## Let's ScaleIn the Pods from 10 to 3.
```
kubectl scale --replicas=3 rc helloworld-controller
```

```
kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
helloworld-controller-f6dgr   1/1     Running   0          84s
helloworld-controller-k2snd   1/1     Running   0          84s
helloworld-controller-nxnkg   1/1     Running   0          83s
```


## Let's try an detele the pod to check RC in actions
```
kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
helloworld-controller-nxnkg   1/1     Running   0          26m
```
```
kubectl delete pod helloworld-controller-nxnkg
pod "helloworld-controller-nxnkg" deleted
```
```
kubectl get pods
NAME                          READY   STATUS              RESTARTS   AGE
helloworld-controller-wzlcv   0/1     ContainerCreating   0          4s
```
```
kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
helloworld-controller-wzlcv   1/1     Running   0          12s
```

## How to delete the Replication Cantroller
```
kubectl get rc
NAME                    DESIRED   CURRENT   READY   AGE
helloworld-controller   1         1         1       29m
```
```
kubectl delete rc helloworld-controller
replicationcontroller "helloworld-controller" deleted
```

