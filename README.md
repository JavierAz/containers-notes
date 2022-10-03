# containers-notes

This repo is to take notes about kubernetes, docker, minikube, etc

## Docker

Coming soon!

## Kubernetes with  Minikube
For start a default instance, only needs the following command
```minikube
minikube start
```
thus minikube starts a simple node with the default configuration, but with some flags you can change others parameters, the `minikube start` and `stop` commands allow us to create custom profiles
`--profile` or `-p` flags.

Exits other flags or commands that allow custom clusters to be created with Minikube

| Parameter                      | Description                                                                                               |
|:-------------------------------|:----------------------------------------------------------------------------------------------------------|
| `--kubernetes-version=v1.22.2` | **Not required**. For select a specific version of kubernetes                                             |
| `--nodes=2`                    | **Not required**. Select how many nodes you want to start                                                 |
| `--driver=docker`              | **Not required**. Select the driver for your instances _(you have other options like virtualbox, podman)_ |
| `--disk-size=10g`              | **Not required**. Capacity for a single node                                                              |
| `--cni=calico`                 | **Not required**. Not clear yet                                                                           |
| `--container-runtime=cri-o`    | **Not required**. Select the container runtime in your cluster                                            |

For some flags you had been installed an extra software like docker and/or podman. Not in all cases in others flags Minikube can be set up, CNI (network plugin) for example. 
```
minikube start  --kubernetes-version=v1.20 --driver=docker --nodes=2 --cpus=2 -p  dockertest
```

The `minikube profile` command allows us to view the status of all clusters in a table formatted output
```minikube
minikube profile list
```

Others commands to use with Minikube to show a output and information about the node, some examples are `minikube node list` to show a list of the nodes of a cluster, and others like `minikube ip -n node-m02`, where ip can show the ip of but with `-n node-m02` only allow to show a specific ip from the node selected.

We can stop with the command `minikube -p dockertest` or delete clusters with `minikube delete -p dockertest`, reminder in delete you'll to select the specific cluster with `-p` or the command will delete the default or the first cluster.

### Accessing to the cluster
To access the Kubernetes cluster have some methods:

- kubectl
  - The Kubernetes Command Line Interface (CLI), is very flexible to integrate with other systems, it can be used remotely from anywhere to access a cluster
  - activate the autocomplete needs run the following command ```source <(kubectl completion zsh)```
  - NOTE for myself: check if minikube is running!
- from the dashboard
  - Kubernetes Dashboard provided a Web-based User Interface to interact with kubernetes cluster, it's still a preferred tool to users who are not as proficient with the CLI
  - Needs enable some metrics to display in the dashboard e.g: `minikube addons enable metrics-server`
  - To start the dashboard from the terminal you need run the following command: `minikube dashboard`
- via APIs
  - Is the main component of the Kubernetes control plane, can be divided in three independent group types:
    - Core Group
    - Named group
    - System-wide
  - Using `kubectl proxy` command you can authenticate to use the API server

### Building blocks

#### Namespaces
Kubernetes generally creates four Namespaces: `kube-syste`, `kube-public`, `kube-node-lease` and `default`
- kube-system contains the objects created by kubernetes system
- default contains the objects and the resources created by admins and devs
- kube-public is special, is unsecured and readable by anyone for exposing public information
- kube-node-lease used for node heartbeat data

#### Pods
A Pod is the smallest Kubernetes workload object, which represents a single instance of the app.
Single - and Multi-Container Pods for stand-alone pod object's definition manifest in YAML format:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    run: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.20.2
    ports:
    - containerPort: 80
```

With this YAML definition we can create a single pod, firstly create the file:
```
vim nginx.yaml
```
Secondly we apply the pod
```
kubectl apply -f nginx.yaml
```
Finally we check if was created
```
kubectl get pods -o wide
```
If something is wrong in ours pods we can get some information with `kubectl describe pods 'namepod'` or delete `kubectl delete -f nginx.yaml`.

Usually we don't deploy a stand-alone pod, so we need do more replicas like this example, thus we only run a command and start some pods.
ReplicaSet is, in part, teh next-generation ReplicationController, we can scale the number of Pods running a specific application container image

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
```

#### Labels
Labels are key-value pairs attached to kubernetes objects, used to organize and select subset of objects, based on the requirements in place


#### DaemonSets
Are operators designed to manage node agents, they are resemble ReplicaSet and Deployment operators when managing multiples Pod replicas and application updates.
