## Let run our first deployment with command line. 

```
kubectl run k8s-get-started --image=amitvashist7/k8s-tiny-web  --port=80
``` 

## Check status. 

```
kubectl get pods
``` 


## Let run our first pod with yaml file definition: 

```
# cd k8s-terraform-cgi-25May2020/03-K8s/02-First-App


# cat helloworld.yml
apiVersion: v1
kind: Pod
metadata:
  name: nodehelloworld
  labels:
    app: helloworld
spec:
  containers:
  - name: k8s-demo
    image: amitvashist7/k8s-tiny-web
    ports:
    - name: nodejs-port
      containerPort: 80

# kubectl create -f  helloworld.yml
```

## Check status. 

```
# kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
hello-kube-7594d6bc45-4pffj   1/1     Running   0          9m51s
nodehelloworld                1/1     Running   0          11m
``` 

## Delete the Pod & Deployment what we have created.
```
# kubectl get pod,deploy
NAME                              READY   STATUS    RESTARTS   AGE
pod/hello-kube-7594d6bc45-4pffj   1/1     Running   0          155m
pod/nodehelloworld                1/1     Running   0          157m

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/hello-kube   1/1     1            1           155m


# kubectl delete pod,deploy --all
```

