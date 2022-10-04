# containers-notes

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

Create namespace: `kubectl create namespace some-name`

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

#### Services
When we work with apps deployed to a Kubernetes cluster commonly needs other applications or clients, the solution would be port mapping but isn't "simple".
he solution is much more sophisticated, with the involvement of the kube-proxy node agent, IP tables, routing rules, cluster DNS server, all collectively implementing a micro-load balancing mechanism that exposes a container's port to the cluster, even to the outside world if desired. This mechanism is called a Service and it is the recommended method to expose any containerized app to the Kubernetes network.

### Security
#### Authentication
Kubernetes doesn't have an object like _user_ or _usernames_ but it supports two kinds of _users_:
- Normal Users
- Service Accounts: communicates with API servers.

#### Authorization
Node authorization is a special-purpose which specifically authorizes API requests made by kubelet.
Or attribute-Based Access Control where a some type of users have some privileges in the sources to:
- regulate the access to resources based on the Roles of individual users
- Restrict resources access by specific operations
- We can create two kind of roles:
  - Role: grant access to resources within a specific namespace
  - ClusterRole: grants the same permissions as Role does, but its scope is cluster-wide

#### Admission Control
Used to specify granular access control policies, where we force these policies using different admission controllers like, ResourceQuota, DefaultStorageClass, AlwaysPullImages, etc.
With this command `--enable-admision-plugins`, and can also be implemented though custom plugins, for dynamic admission control 

### Services
This is a object used to abstract the communication between cluster internal microservices or with the external world.
Labels and Selector use a **key-value** pair format, and using that we can group Pods into logical sets. We assign a name to the logical grouping, referred to as a Service. The following is an example of a Service object definition:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
spec:
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5000
```
#### kube-proxy
Is responsible for implementing the Service configuration on behalf of and administrator or dev, to enable traffic routing to an expose application running in Pods. Also with the iptables implement the load-balancing mechanism because if you don't apply some policies the load-balancing will be randomly. 

To get connection outside the cluster from the Pod, you'll need change the configuration `kubectl edit svc name-pod` and change TYPE from _ClusterIP_ to _NodePort_

#### ServiceTypes 
- **LoadBalancer:** Only work if the underlying infra supports the automatic creation of Load Balancer and have respective support in kubernetes.

  - NodePort and ClusterIP are automatically created, and the external load balancer will route to them
  - The service is exposed at a static port on each worker node
  - The service is exposed externally using underlying cloud provider's load balancer feature

- **ExternalIP:** Can be mapped to an _ExternalIP_ address iff it can route to one or more of the worker nodes. This gets routed to one of the services endpoints but this type of service requires an external cloud provider.
- **ExternalName:** is a special because that has no Selector and does not define any endpoints. When accessed within the cluster, it returns a CNAME record of an externally configured service.

### Deployment

