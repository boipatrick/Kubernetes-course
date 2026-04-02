## Introduction
Kubernetes is one of the most widely used open source platforms for orchestrating containers
It automates the deployment, scalig and management of applications. 

## Kubernetes Architecture
Deployed as a cluster, meaning it runs across multiple servers that share workloads, improvereliability and scale applications efficiently. 

Kubernetes clusters consist of two different server node types that make up a cluser:
### Control plane node(s)
These nodes act as the brains of the cluster. They run components that manage the overall system, including workload scheduling, deployments and self-healing behaviors

### Worker nodes
worker nodes are responsible for running applications-nothing more. They do not make decisions on their own; instead, they follow instructions from the control plane such as when to start or stop containers. 


An important aspect of Kubernetes' design is that workloads aleady running on a worker node will continue to operate even if the control plane becomes unavailable. However, while the control plane is offline, key functions- such as scheduling new workloads or scaling existing ones- canot take place. 

Kubernetes also provides the concept of namespaces, which shouldnot be confused with the kenrel namespaces used to isolate containers.

Kunernetes namespaces allow you to organize a cluster into logical groups, makin it easier to support multitenancy when multiple eams share the same cluster. They are not meant to provide strong isolation; instead, you can them of them like directories on a computer , helping you organize resources and control access


## Control Plane Node: kube-apiserver
This is the entry point for all commands to a kubernetes cluster.

It acts as the central communication hub that all other components depend on. It exposes the Kubernetes API, which serves as the single interface through which every component, whether a user issuing a kubectl command, an internal controller, a scheduler, or an external client, interacts with the cluster.

Its primamry responsibility is to handle RESTufl API requests, ensuring secure and auhenticated communication. Every CRUD operation on kubernetes resources such as pods, deployments, Nodes, Services and ConfigMaps flows through the API server making it the authoratative interface for both humans users and automated systems. 

Each incoming reques to the kune-apiserver undergoes authentication and authorixation. Authentication verifies who is making the request, using mechanisms such as service accounts, client certificatesor bearer tokens

Authorization then checs systems sch as Role-Based Access Control, Attribute-based Access Control or webhook-based authorization. 

After authentication and authorization, the request passes through admission controllers, which are built-in modules that enforce cluster-wide policies before the request is persisted in the cluster's datastore.

## Control Plane Node: etcd
This is the central data store of the kubernetes control plane. It records and maintains the complete desired and observed state of the cluster.
It is a highly consitent and distributed key-value store that serves as the single source of truth for the entire cluster. 

Every configuration detail, resource definition, and state change within the cluster is recorded in etcd.

Whenever a new object is create, updated or deleted, whether it's a pod, Deployment, ConigMap, or any other Kubernetes reource, the corresponding data is stored or modified inetcd through the kune-aapiserver.

The Kube-apiserver is the only control plabe component that directly communicates with etcd. All other components communicate through the API server.

This strict design separation ensures data integrity, consistency, an security. 

The API server acts as a gatekeeper, validating and authenticating every  request before it reaches etcd, preventing direct or unauthorized modifications to the cluster's state. 

Etcd's performance and stability directly affect the reliability of the entire kubernetes control plane. If etcd becomes unavailable or slow, the cluster's ability to process updates, schedule new workloads, or maintain state consistency is impacted. 
For this reason, etcd is usually deployed on dedicated control plane nodes with fast SSD storage and strong backup policies. 
Regular snapshots of etcd data are essentail for disaster recovery, as restoing a corrupted or lost etcd database is the only way to recover the cluster state. 

Etcd stores data in a simple hierarchical structure using key-value pairs. Each key represents a kubernetes object or configuration path, and the corresponding value contains the serialized data in JSON or protocol Buffers format. The structure is designed to efficiently handle watch operation, allowing the kube-apiserver to monitor changes and notify other components in real time. 

## Control Plane Node:kube-scheduler
It is the decision maker of the kubernetes workload placement. 

It watches newly created pods that o not yet have a nod assigned and determines the most suitable node for each based on a set of scheduling rules, constraints, and policies. Once a decisio is made, the scheduler updates the pod specification through the kube-apiserver, recording which node the pod will run on. From there, the kubelet on that node takes over, handling pod creation and execution.

It ensures that workloads are distributed efficiently across available resurces while respecting all operational requirements and constriants. The scheduler contnously monitors the cluster's state,looking for unscheduled pods and for each pod, i evaluates which node best fits the pod's needs. 

The scheduling process can be divided into two main phases: filtering and scoring.
In the filtering phase the scheduler eliminates all nodes that cannot accomodate the pod this might be due to insufficient momory or cpu, mismatched node selectors, taints and tolerations, or anti-affinity rules. 

In the scoring phase, the scheduler ranks the remaining nodes to determine the optimal placement. 
Scoring is based on a collection of policies and plugins that consider factors such as resource utilization, data locality, topology spread and pod afffinity. The node with the highest score is selected for the pod, and the scheduler records this binding through the kube-apiserver.
The scheduler also respects affinity and anti-afinity rules, robust mechanisms for influencing pod placement.
Pod affinity allows grouping related pods on the same node or within the same zone to improve performance or reduce latency. At the same time, anti-affinity spreads replicas across different nodes or zoness to increase fault tolerance. 
Similarly, taints and tolerations enable operators to control which pods can be scheduled on specific nodes, ensuring critical workloads are isolated or that certain nodes are reserved for particular purposes.

Without the kube-scheduler, pods would remain unscheduled, and the cluster would lack the automation required to manage workloads effectively. 

 ## Worker Node Components

 Each worker node is responsible for running the following components

 ### Container runtime
 It is the key component of every kubernetes worker node, serving as the engine that runs and manages container lifecycles within pods. 
 It communicates with the kubelet usinf CRI, a standardized gRPC-based protocol that enables kubernetes to support multiple runtimes without modification. 
 For efficient resource management, the kubelet and container runtime must use the same cgroup drivrt-either systemd or cgroupfs. When systemd is the Linux init system, both should use the systemd driver to avoid conflicts and ensure stable performance. 

 ### Kubelet
 main worker-side agent in kubernetes, acting as the foreman that ensures whatever the control panel orders actually gets done on its assigned machine(node). It receives PodSpecs, which are blueprints describing which containers should run and how, and makes sure those containers are up and healthy. 
 The kubelet only manages containers created via kubernetess, not any others running locally. It also registers its node with the API server, continously reports the node's health and available resources, and communicates with the container runtime. 
 In short if kubernetes were a construction site, the control plane would be the architect, amd the kubelt would be the on-site manager, ensuring everything is built and running exactly as planned. 

 ### kube-proxy
 it is a vital network component that runs on every worker node in a kubernetes cluster, responsible for routing and managing network traffic between services and pods. 

 It ensures that when traffic is sent to a kubernetes service, a stable frontend virtual IP is properly directed to one of the healthy backend pods that implement that service.

 Acting as a simple load balancer,kube-proxy handles TCP, UDP and SCTP traffic and uses the operating system's networking layer, such as iptables, ipvs,or ntables on lunix, to manage and enforce packet forwarding rules.
 If the system's packet filtering layer is unavailable, kube-proxy performs traffic forwarding itself. 

 While it was once mandatory, kube-proxy is now optional when using advanced network pluins that already handle service routing and proxying. 
 In essence, if the kubelet is the on-site foreman ensuring that containers are running, the kube-proxy is the traffic contoller, constantly updating the node's internal routing maps so that incoming requests are efficiently sent to the correct pos, keeping communication within the cluster seamless and reliable. 


 ## Kubernetes API 
 It is the central component of a kubernetes cluster. It enables communication with the cluster-every user,tool and internal component relies on the API server to interact with and manage the system.
 Before Kubernetes processes a request, it must pass theough three key stages:
 Authentication: the requester must prove their identity to the API server. This is commonlydone using an X.509 client certificate or with an exernal identity management system, 
 Authorization: once authenticated, kubernetes determines what the requester is allowed to do. This is typically handled through Role Based Access Control (RBAC) which defines permissions for users and service accounts
 Admission Control:In the final stage admission controllers can validate or modify the request before it is accepted. For instance, if a user attempts to deploy an image from an untrusted registry, an admission controller could block it. External tools such as Open Policy Agent can also be used to enforce admission policies. 

 Kubernetes exposes its functionality through a RESTful API served over HTTPS. Using this API, users and services can create, update, delere or retrieve any resource in the cluster.

 ## Running Containers on Kubernetes
 Unlike on your local machine in kubernetes youdon't run containers on their own-instead, you create pods, the smallest compute unit in the system.
 Kubernetes then turns the pod definition into one or more running containers.
 Running Containers on Kubernetes
How is running a container on your local machine different from running one in Kubernetes?

Locally, you start containers directly. In Kubernetes, you don’t run containers on their own—instead, you create Pods, the smallest compute unit in the system. Kubernetes then turns the Pod definition into one or more running containers. We’ll explore Pods in more detail later; for now, think of a Pod as a wrapper around a container.

When you create a Pod in Kubernetes, multiple components work together behind the scenes before the containers finally start on a worker node.

Runtimes like containerd and CRI-O focus on providing only the core functionality needed to run containers. Even so, they can integrate with sandboxing technologoes that address security concerns related to sharing the host kernel across containers. Common  options include: gvisor and kata containers. 

## Networking
Kubernetes must support communication among many contianers running across many nodes. To handle this, Kubernetes breaks networking into four keycommunication paths:
1. Container to container communications: handled within a pod, which provides a shared network namespace.
2. Pod to pod communications: typically implemented using an overlay network. 
3. pod to service communications: managed on each node by kube-proxy and the node's packet filtering mechanism.
4. External-to-service communications: also handled by kube-proxy and the node's packet filtering rules

In kubernetes all solutions must meet three core requirements:
1. Every pod must be able to communicate with every other pod across nodes

2. Every node must be able to communicate with every pod
3. No network Address Translation(NAT) should be required within the cluster

To meet these requirements, Kubernetes supports a wide range of networking providers, including 
Project Calico
Weave
Cilium 

In Kubernetes, every Pod receives its own IP address, removing the need for manual network configuration. Most clusters also include a DNS addon, CoreDNS, which provides service discovery and name resolution within the cluster.

By default, all Pods can communicate with each other. If you need to restrict or control this traffic at the IP or port level, you use Network Policies. Network Policies act like internal firewalls for the cluster. They let you define which Pods or namespaces can communicate by using label selectors, and they can also include IP-based rules using CIDR ranges.

Network Policies are enforced by the cluster’s network plugin. To use them, your networking solution must support the NetworkPolicy resource—otherwise, creating a policy will have no effect.

## Scheduling
It refers to the process of automatically selecting the most appropriate worker node on which to run a containerized workload. 

The kube-scheduler is responsible for making these scheduling decisions. Process begins whenever a new Pod is created. Because Kubernetes uses a declarative model, the pod is first defined and then the sceduler decides which node it should run on. The kkubelet on that node, along with the container runtime, handles actually starting the continers. 

The scheduler relies on information provided by the user, such as CPU a memory requests, or specific node characteristics. For instance you might specify that an application needs 2 CPU core, 4GB of memory and ideally runs on a node with fast storage. 

The scheduler uses this information to filter out nodes that do not meet the requirements. If multiple nodes are suitable, Kubernetes chooses the one with the fewest running Pods. If no node meets the requirements—such as when insufficient resources are available—the scheduler continues retrying until the desired state is achieved.

