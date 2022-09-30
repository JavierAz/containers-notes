# containers-notes

This repo is to take notes about kubernetes, docker, minikube, etc

## Docker

Coming soon!

## Minikube
For start a default instance, only needs the following command
```minikube
minikube start
```
thus minikube starts a simple node with the default configuration, but with some flags you 
can change others parameters, the `minikube start` and `stop` commands allow us to create custom profiles
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

For some flags you had been installed an extra software like docker and/or podman. Not in all cases in others flags Minikube 
can be set up, CNI (network plugin) for example. 
```
minikube start  --kubernetes-version=v1.20 --driver=docker --nodes=2 --cpus=2 -p  dockertest
```

The `minikube profile` command allows us to view the status of all clusters in a table formatted output
```minikube
minikube profile list
```

Others commands to use with Minikube to show a output and information about the node, some examples are `minikube node list`
to show a list of the nodes of a cluster, and others like `minikube ip -n node-m02`, where ip can show the ip of but with `-n node-m02` 
only allow to show a specific ip from the node selected.

We can stop with the command `minikube -p dockertest` or delete clusters with `minikube delete -p dockertest`, reminder 
in delete you'll to select the specific cluster with `-p` or the command will delete the default or the first cluster.

### Accessing to the cluster
To access the Kubernetes cluster have some methods:

- kubectl
  - The Kubernetes Command Line Interface (CLI), is very flexible to integrate with other systems, it can be used remotely from anywhere to access a cluster
- from the dashboard
  - Kubernetes Dashboard provided a Web-based User Interface to interact with kubernetes cluster, it's still a preferred tool to users who are not as proficient with the CLI
- via APIs
  - Is the main component of the Kubernetes control plane, can be divided in three independent group types:
    - Core Group
    - Named group
    - System-wide

