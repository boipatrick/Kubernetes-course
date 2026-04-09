Kubernetes Objects
Roles within a cluster
How to interact with them effectively.

Once a cluster is setup either newly created or pre-existing you can begin deploying workload. 

Smallest compute in kubernetes is a pod,aand pods are only one part of the workload model. Kubernetes provides several workload objects that determine how pods are created, scaled, updated and managed throughout their lifecycle. 

Apart from deploying workloads kubernetes also provides solutions to operational challenges arising from running containers at scale, including configuration management, cross-node networking external traffic routing, load balancing, and automated pod scaling.  

## Kubernetes Objects
One Kubernetes core principles is that it represents system behavior through a collection of abstract resources called objects that define how workloads should run and be managed. 

Some objects focus on orchestration tasks such as scheduling, scaling, and self-healing, while others address fundamental container conccerns like configuration, networking and security. 
Kubernetes objects fall into two categories
1. Workload objects- which define and manage containerized applications
2. Infrastructure Objects- which provide supporting functionality such as networking, access control and configuration management. 

Many objects are scoped to a specific namespace, while others exist at the cluster level and apply globally. 

Users define these objects using YAML, a common data serialization format, and submit them to the Kubernetes API server, where the definition are validated and then used to create or modify resources in the cluster. 

It's important to understand that creating, updating, or deleting an object in kubernetes expresses intent-you're declaring the state you want, not manually starting vcontainers like you would on a local machine. Kubernetes evalutaes that desired state and works to reconcile the system so that it matches what you've defined. 


## Pod Concept
A pod represents one or more containers that share the same isolation boundaries- Such as namespaces and cgroups- and are managed as a single unit. Because a pod is the smallest deployable resource in kubernetes, the platform does not interact with containers directly; it always operates through pods.

The pod abstraction was created to support scenarios where multiple tightly coupled processes need to run together. All containers within a pod share the same IP address and can communicate over localhost, and they can also share storage volumes through a common filesystem.

It is possible to run multiple containers within a single pod to support your main application. However, be mindful that doing so means you can no longer scale those containers independently because the pod sales as a unit. When an additional container provides supporting functionality for the primary application, it is referred to as a sidecar container. \

By default all containers in  apod start at the same time, with no guaranteed order. I f you need a certain processes to run before the main application starts you can use initcontainers. These containers run sequentially and must complete successfully before the main application containers begin. 


## Pod Lifecycle
Pods move through a defined set of phases from creation to termination. A pod typically starts at
1. Pending Phase- The pod has been accepted by the cluster, but one or more containers have not yet been created. This phase includes time spent waiting for scheduling as well as time downloading container images.
2. Running - the pod has been assigned to a node and all of its containers have been created. At least one container is still running or is in the process of starting or restarting.
3. Succeeded - All containers in the pod have exited successfully and will not be restarted

4. Failed - All containers have stopped ad at least one exited with a non-zero exit code or was terminated by the system. 

5. Unknown- the pod's state cannot be retrieved, typically due to communication issues with the node where the pod is scheduled. 

## Workload objects
using pods alone isn't flexible enough for a full container orchestration platform. If a pod disappears-e.gbecause the node running it fails-it isn't automatically recreated. To rensure that a specific number of pod replicas are always running, kubernetes uses controler objects that manage pods on our behalf. 

### Kubernetes Objects
ReplicaSet is an essential kubernetes controller that maintains a stable set of identical pods at any given time. Its primary function is to ensure that the specified number of pod replicas defined in the replicas field are always active.

- Deployment - is one of the most powerful and commonly used controllers in kubernetes. It manages the full lifecycle of stateless applications, providing declarative updates to pods and ReplicaSets

- statefulSet is designed for managing stateful applications that require stable identities, persistent storage, and predictable pod ordering. Statefulsets assign each pod a unique, persistent identifier and ensure that the same identity is retained even if the pod is rescheduled. 

- DaemonSet ensures that a specific pod runs on every node in a kubernetes cluster
- Job is a kubernetes controller designed to run finite, one-time tasks to completion.

- A cronjob extends the job concept by adding time based scheduling. 

### Networking Objects
Managing networking for individual pods would require extensive manual configuration, so Kubernetes provides Service and Ingress objects to simplify and abstract network access

### Services
A serviceexposes one or more pods as a nnetwork endpoint. Depending on how you want traffic to flow, you can choose from several service types. 

#### Service Types
- ClusterIP -this is the default and most common type. It creates a virtual IP inside the cluster that acts as a single connection point for a groupofpods, often used for internal eoundrobin load balancing.

- NodePort the nodeport service type builds on ClusterIP by opening a port on every node and mapping it to the service
- Loadbalancer - extends Nodeport by provisioning an external load balancer. This requires an environment that supports automated load balancer creation such as AWS

- ExternalName - a special service type that has no routing whatsoever. 



Together, these service types help route internal and external traffic to workloads without manually configuring network rules for each pod.


## Headless Services
In some situations, you don't need load balancing or a single virtual service IP. In these cases, you can create a headless service by setting .spec.clusterIP to None

Headless services allow applications to handle service discovery independently rather than relying on Kuberntes built-inmechanisms. With a headless service:
No Cluster IP is assigned
Kube-proxy does not configure routing rules
Kubernetes does not provide load balancing or proxying. 

A common example is using a headless Service with a StatefulSet, where each Pod needs a stable network identity rather than a shared load-balanced address.


## Ingress: Exposing apps to the outside world
when you need more flexible or HTTP-aware routing,you can use an ingress resource. It exposes HTTP and HTTPSendpoits from outside the cluster to internal services by applying user-defined routing rules. These rules are enforced by an Ingress Controler, which implements the actual networking logic

Typical features of ingress controllers include:
- loadBalancing
- TLS offloading or termination
- Name-based virtual hosting
- Path-based routing

Many controllers offer aditional capabilities such as 
- redirects
- custom error pages
- authentication and session affinity
- monitoring and logging
- weighted routing
- rate limiting 


## Kubernetes Service Exposure: Ingress vs Gateway API
Both serve the purpose of exposing internal aplications to the outside world. 

### Ingress API
It allows routing of HTTP and HTTPS traffic intoa cluster based on hostnames and URL paths

### Gateway API
was introduced as a modern replacement and enhancement to ingress, developed under Kuberntes' SIG Nretwork as a more flexible and extensible model. 

It addresses ingress's shortcomings by introducing a role-oriented, modular and protocol-agnostic architecture.

Instead of one monolithic resource, the Gateway API is composed of several resource types: GatewayClass, Gateway HTTPRoute, TCPRoute, UDPRoute, and others with each serving a distinct purpose. 


### Comparison and Practical Cnsiderations
Ingress remains a simple and widely supported option for straight-forward HTTP-based routing. It is suitable for small to medium environments or legacy workloads that only need basic routing and TLS support.

However it is constrained by annotation sprawl, limited protocol diversity and poor standardization across controllers. 

The Gateway API represents the next generation of traffic management model for kubernetes. It is modular,controller-neutral, and designed for modern cloud-native environments. 

It supports advanced use cases such as header-based routingtraffic splitting, websocketrs, gRPC, and non-HTTP protocols out of the box.

 Its architecture also encourages the separation of responsibilities; for example, platform engineers can manage Gateways, while application teams can define their own Routes, making it ideal for large, multi-tenant clusters.
 
Many existing Ingress Controllers, such as NGINX and Traefik, now support both Ingress and Gateway API, enabling organizations to migrate gradually without re-architecting their systems. For new Kubernetes deployments, the Gateway API is increasingly recommended as the preferred standard because of its cleaner design, broader protocol support, and future-proof architecture that aligns with Kubernetes’ long-term networking vision. Also, Ingress is now feature-frozen, and all new functionality and enhancements are being developed under the Gateway API.


#### NetworkPolicy
Network policies are simple IP firewalls (OSI Layer 3 or 4) that can control traffic based on rules. You can define rules for incoming(ingress) and outgoing(egress). A typical use case for Network Polices is restricting traffic between two namespaces. 

A kubernetes Network Policy is a declarative resource that defines how pods are allowed to communicate with each other and with external endpoints inside and outside a cluster.

It acts as a pod-level firewall, enforcing fine-grained network access control in one line with zero-trust security principles. 
By default, all pods ain Kubernetes can send and receive traffic from any source, creating an open and permissive network. Applying a NetworkPolicy restricts this behavior by defining explicit ingress(incoming) and egress(outgoing) rules based on pod lebels, namespace labels, IP blocks, ports and protocols.

Enforcement of these policies requires a compatible CNI(Container Network Interface) plugin such as Calico, Cilium, Weave Net or AWS VPC


### Volume and Storage Objects
To provide a consistent interface across diffeent storage providers, Kubernetes implemetns the Container Storage Interface(CFI) which allows vendors to supply storage drivers as plugins


### Persistent Volumes
A PV represents a piece of storage in the cluster that has been provisioned by an administrator (static provisioning) or dynamically through a StorageClass. 

It is an abstraction layer that hides the underlying storage technology- whether it's a cloud disk, a network file system, or a local disk on a node. 

Each PV object contains metadat that describes key properties of the storage resource including
- Capacity
- Access Modes
- Volume Mode
- Reclaim Policy
- Storage Backend information
- Node Affinity

A PV’s lifecycle is independent of any individual Pod that uses it. This means the data stored inside persists even if the Pod using it gets deleted or rescheduled.
