# Labels and Selectors

Labels are key/value pairs that are attached to objects, such as pods. Labels are intended to be used to specify identifying attributes of objects that are meaningful and relevant to users, but do not directly imply semantics to the core system. Labels can be used to organize and to select subsets of objects. Labels can be attached to objects at creation time and subsequently added and modified at any time. Each object can have a set of key/value labels defined. Each Key must be unique for a given object.

```
"metadata": {
  "labels": {
    "key1" : "value1",
    "key2" : "value2"
  }
}
```

# Motivation

Labels enable users to map their own organizational structures onto system objects in a loosely coupled fashion, without requiring clients to store these mappings.

Service deployments and batch processing pipelines are often multi-dimensional entities (e.g., multiple partitions or deployments, multiple release tracks, multiple tiers, multiple micro-services per tier). Management often requires cross-cutting operations, which breaks encapsulation of strictly hierarchical representations, especially rigid hierarchies determined by the infrastructure rather than by users

Example labels:

- ``` "release" : "stable", "release" : "canary" ```
- ``` "environment" : "dev", "environment" : "qa", "environment" : "production"```
- ```"tier" : "frontend", "tier" : "backend", "tier" : "cache"```

# Syntax and character set
Labels are key/value pairs. Valid label keys have two segments: an optional prefix and name, separated by a slash (/). The name segment is required and must be 63 characters or less, beginning and ending with an alphanumeric character ([a-z0-9A-Z]) with dashes (-), underscores (_), dots (.), and alphanumerics between. 

For example, here’s the configuration file for a Pod that has two labels environment: production and app: nginx :
```
apiVersion: v1
kind: Pod
metadata:
  name: label-demo
  labels:
    environment: production
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```
# Label selectors
Unlike names and UIDs, labels do not provide uniqueness. In general, we expect many objects to carry the same label(s).

Via a label selector, the client/user can identify a set of objects. The label selector is the core grouping primitive in Kubernetes.

The API currently supports two types of selectors: equality-based and set-based. A label selector can be made of multiple requirements which are comma-separated. In the case of multiple requirements, all must be satisfied so the comma separator acts as a logical AND (&&) operator.

### Equality-based requirement
Equality- or inequality-based requirements allow filtering by label keys and values. Matching objects must satisfy all of the specified label constraints, though they may have additional labels as well. Three kinds of operators are admitted =,==,!=. The first two represent equality (and are simply synonyms), while the latter represents inequality. For example:

```
environment = production
tier != frontend
```

The former selects all resources with key equal to environment and value equal to production. The latter selects all resources with key equal to tier and value distinct from frontend, and all resources with no labels with the tier key. One could filter for resources in production excluding frontend using the comma operator: environment=production,tier!=frontend

```
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
        - name: nodejs-port
          containerPort: 80
      nodeSelector:
        hardware: virtual
```

## Run the deployment with node selector
```
```

## Check the Deployment Pod Status
```
```

## Describe Pod
```

```


## Assign label to Nodes
```
kubectl label nodes worker01 hardware=virtual
```

## Check the Deployment Pod Status, all pod must be running on worker01
```
```

## Assign label to worker02
```
kubectl label nodes worker01 hardware=virtual
```

## Scale the Deployment. 
```

```

## Check the Deployment Pod Status, all pod must be running on worker01
```
```


## Mult-Node Selector

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: helloworld-deployment-2
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: helloworld-1
    spec:
      containers:
      - name: k8s-demo
        image: amitvashist7/k8s-tiny-web
        ports:
        - name: nodejs-port
          containerPort: 80
      nodeSelector:
        hardware: virtual
        env: prod
```

## Run the deployment with multi node selector
```
```

## Check the Deployment Pod Status
```
```

## Describe Pod
```

```


## Assign label to Nodes
```
kubectl label nodes worker01 hardware=virtual
kubectl label nodes worker01 hardware=virtual
kubectl label nodes worker01 env=prod
kubectl label nodes worker01 env=prod
```
