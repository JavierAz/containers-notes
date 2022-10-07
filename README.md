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
Will be using the CLI:
- First we need to create a YAML file with the content required
- We'll create the Deployment from the YAML configuration file. example: `kubectl create -f name.yaml`
- If you want check Pods and Replicas, run the following command `kubectl get replicasets` & `kubectl get pods` or `kubectl get all`
- For expose an app we need to create a Service, with the following command `kubectl create -f nameofService.yaml` or wth a more direct method is Expose a deployment with **kubectl expose** command: `kubectl expose deployment webserver --name= --type=NodePort`
- For check list the Services: `kubectl get services`
- But if you want more information about some Service only run this command `kubectl describe service web-service`

What happened if the app inside the Pod doesn't respond, we need to restart it and we need check tje liveness command is checking the existence of a file _/tmp/healthy_, the existence of this file is configured to be checked every **n** seconds using the periodSeconds parameter.  

### Kubernetes Volume Management
We will manage other types of Pods, because these ones can be either data producers, data consumers, or both. That types need must outlive to the Pod in order to be aggregated and possibly loaded into analytics engines.
So we have Volumes of several types and a few other forms of storages resources containers data management, like **PersistentVolume** and **PersistentVolumeClaim**.

#### Volumes
As we know, containers in Pods are ephemeral, so we these ones failed, restarted, or deleted all the data inside is deleted. Kubernetes to overcome this problem uses Volumes, where this uses storage technologies to be a mount point on the container's file system backed by a storage medium. In Kubernetes a Volume is linked to a Pod and can be shared among the containers of that Pod.

A **PersistentVolumeClaim** is a request for storage by a user, this one for that one resource based on type, access mode, and size. There are three access mode:
- ReadWriteOnce (a single node)
- ReadOnlyMany (many nodes)
- ReadWriteMany (many nodes)

This is an example for a YAML file to create Volumes:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: blue-app
  name: blue-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: blue-app
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: blue-app
        type: canary
    spec:
      volumes:
      - name: host-volume
        hostPath:
          path: /home/docker/blue-shared-volume
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: host-volume
      - image: debian
        name: debian
        volumeMounts:
        - mountPath: /host-vol
          name: host-volume
        command: ["/bin/sh", "-c", "echo Welcome to BLUE App! > /host-vol/index.html ; sleep infinity"]
status: {}
```
### ConfigMaps and Secrets
While we need to deploy the applications these ones need authentication or something like that. Is easier config secrets to the costumer requirements.

#### ConfigMaps
Using ConfigMaps, we pass configuration data as key-value pairs, which can uses like environment variables, sets of commands and arguments, or volumes. 
Create the ConfigMap: 
```
kubectl create configmap my-config --from-literal=key1=value1 --from-literal=key2=value2
```
Display the ConfigMap details
```
kubectl get configmaps my-config -o yaml
```
Or if you have a YAML file you also can create the ConfigMap with the following command: `kubectl create -f file.yaml`

#### Secrets
In some cases we need to provided a password, certificate or something like that but in the YAML or JSON file that is dangerous and insecure. A Secret object can help to encode before sharing.
The Secret data is stored as plain text inside **etcd**, therefore admins must limit access to the API server and **etcd**. They can be created like this example:
```
kubectl create secret generic my-password --from-literal=password=mysqlpassword
```
We can get some information about the secret with `kubectl get secret my-password` or `kubectl describe secret my-password`

In the following example  is a Secret inside a Pod
```yaml
spec:
  containers:
  - image: wordpress:4.7.3-apache
    name: wordpress
    volumeMounts:
    - name: secret-volume
      mountPath: "/etc/secret-data"
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: my-password
```

### Ingress
According to kubernetes.io: _"An Ingress is a collection of rules that allow inbound connections to reach the cluster Services"_
Is the way to keep outside the Service the ingress to the Pod. If you want to check if Ingress is available you can run the following command: `minikube addons list` so, default of Ingress is disabled and for starting run `minikube addons enable ingress` 
