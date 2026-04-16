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

## PersistentVolumeClaims(PVC)
A PVC is a user's request for storage similar to how a pod requests CPU and memory. It allows developers to spcify the amount of storage, access mode, and optionally a StorageClass that describes the type or performance tier of the desired storage.

### StorageClass
defines how dynamic volume provisioning works within a cluster.
It tells kubernetes which storage to use, how to create it, and which parameters or performance tiers to apply when a pod requests persistent storage. 

Without SC you would need to manually create PVs before binding them to PVCs a static and error-prone procss.

With a SC, kubernetes can automatically provision PVs on demand when a PVC is created. This dynamic provisioning improves scalability and reduces manual intervention, making it idealfor cloud and large-scale environments.

Each SC includes details such as:
- Provisioner
- parameters
- Reclaim policy
- Volume Binding Mode

Volume & Storage Objects
As mentioned earlier, containers were not originally designed to support persistent storage, especially when storage needs to exist across multiple nodes. Kubernetes provides mechanisms to manage persistent data, but these do not eliminate all of the underlying complexity—storage in distributed environments still requires careful planning and configuration.

While containers can mount volumes directly, Kubernetes abstracts this concept by attaching volumes to Pods rather than individual containers. This allows multiple containers in the same Pod to share storage and lets Kubernetes manage the lifecycle of both resources together.

Here’s an example of a hostPath volume mount, which works similarly to host-mounted volumes in Docker:

apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-pd
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      # directory location on host
      path: /data
      # this field is optional
      type: Directory

 



hostPath Volume Mount


 

Volumes allow data to be shared both between multiple containers in the same Pod and, depending on configuration, between Pods across the cluster. This is especially useful when implementing sidecar patterns where supporting containers need access to shared files. Volumes also help prevent data loss when a Pod restarts on the same node. Because Pods always start with a clean state, any data not written to a volume is lost when the Pod is terminated or rescheduled.

However, running Kubernetes across multiple nodes introduces additional challenges for persistent storage. Depending on the environment, storage may come from cloud-managed block devices such as Amazon EBS, Google Persistent Disks, Azure Disk Storage; distributed systems like Ceph or GlusterFS; or traditional network storage like NFS.

These are just a few examples of storage options Kubernetes can use. To provide a consistent interface across different storage providers, Kubernetes implements the Container Storage Interface (CSI) which allows vendors to supply storage drivers as plugins.

To work with persistent storage using this abstraction, Kubernetes introduces three additional objects.

PersistentVolumes (PV)
A PersistentVolume (PV) represents a piece of storage in the cluster that has been provisioned by an administrator (static provisioning) or dynamically through a StorageClass. It is an abstraction layer that hides the underlying storage technology—whether it’s a cloud disk, a network file system, or a local disk on a node.

Each PV object contains metadata that describes key properties of the storage resource, including:

Capacity: The total size of the storage volume (for example, 10Gi)
Access Modes: How the volume can be mounted by Pods:
ReadWriteOnce (RWO): The volume can be mounted as read-write by a single node.
ReadOnlyMany (ROX): The volume can be mounted as read-only by many nodes.
ReadWriteMany (RWX): The volume can be mounted as read-write by many nodes.
Volume Mode: Determines whether the volume is presented as a filesystem (Filesystem) or a raw block device (Block)
Reclaim Policy: Specifies what happens when a PV is released by a PVC. Possible values are:
Retain: Keeps the volume data for manual cleanup or reuse
Delete: Deletes the storage resource automatically (typical for cloud providers)
Recycle: Performs a basic cleanup and makes it available again (deprecated)
Storage Backend Information: Defines the specific storage type and parameters, such as NFS path, iSCSI target, Ceph pool, or cloud volume identifiers
Node Affinity: Indicates which nodes the volume can be attached to, especially important for zone-aware cloud volumes.
A PV’s lifecycle is independent of any individual Pod that uses it. This means the data stored inside persists even if the Pod using it gets deleted or rescheduled.

Let’s take a look at an example of PersistentVolume (NFS-based):

apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs
spec:
  capacity:
    storage: 20Gi
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /data/nfs-storage
    server: 192.168.1.10

This example defines a 20Gi NFS-backed PersistentVolume that can be mounted read-write by multiple nodes simultaneously and retains its data even after the PVC is released.

PersistentVolumeClaims (PVC)
A PersistentVolumeClaim is a user’s request for storage similar to how a Pod requests CPU and memory. It allows developers to specify the amount of storage, access mode, and optionally a StorageClass that describes the type or performance tier of the desired storage.

When a PVC is created, Kubernetes looks for an available PV that matches its request (based on size, access mode, and storage class). If such a PV exists, it is bound to the PVC. If not, and if a StorageClass is defined, Kubernetes dynamically provisions a new PV that satisfies the claim. Once bound, the PV is dedicated exclusively to that PVC until it is released.

Let’s take a look at a PersistentVolumeClaim example:

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-data-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: ebs-gp2

This PVC requests a 10Gi storage volume that can be mounted read-write by a single node, using the ebs-gp2 StorageClass (for example, AWS EBS). Kubernetes either binds this PVC to an existing PV with compatible parameters or dynamically provisions a new one.

Once bound, Pods can mount the PVC as a volume:

apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app-container
    image: nginx
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: app-storage
    volumes:
    - name: app-storage
      persistentVolumeClaim:
        claimName: app-data-pvc

Here, the Pod mounts the PVC at /usr/share/nginx/html. Any data written to this path will persist even if the Pod is restarted or rescheduled on a different node.

StorageClass
A StorageClass in Kubernetes defines how dynamic volume provisioning works within a cluster. In simple terms, it tells Kubernetes which type of storage to use, how to create it, and which parameters or performance tiers to apply when a Pod requests persistent storage.

Without a StorageClass, you would need to manually create PersistentVolumes (PVs) before binding them to PersistentVolumeClaims (PVCs), a static and error-prone process. With a StorageClass, Kubernetes can automatically provision PVs on demand when a PVC is created. This dynamic provisioning improves scalability and reduces manual intervention, making it ideal for cloud and large-scale environments.

Each StorageClass includes details such as:

Provisioner: The driver or plugin that actually creates the volume (e.g., kubernetes.io/aws-ebs,, kubernetes.io/gce-pd, kubernetes.io/cinder, or CSI drivers like ebs.csi.aws.com)
Parameters: Key-value pairs that define characteristics such as storage type, performance level, or filesystem
Reclaim Policy: Determines what happens to a volume after the PVC is deleted (Retain, Delete, or Recycle)
Volume Binding Mode: Controls when the volume is provisioned and bound (Immediate or WaitForFirstConsumer)
Let’s look at an example of dynamic provisioning with a StorageClass using AWS EBS (Elastic Block Store):

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-gp2
provisioner: ebs.csi.aws.com
parameters:
  type: gp2
  fsType: ext4
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer

Explanation:

provisioner: Specifies the CSI (Container Storage Interface) driver for AWS EBS.
parameters: Requests a gp2 (general-purpose SSD) volume formatted with ext4.
reclaimPolicy: Delete means that when the PVC using this volume is deleted, the underlying storage volume will also be deleted automatically.
volumeBindingMode: WaitForFirstConsumer delays provisioning until a Pod using the PVC is scheduled, ensuring the volume is created in the same availability zone as the Pod.
Now, you can dynamically request a PersistentVolume using this StorageClass:

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-app-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: ebs-gp2

When this PVC is created, Kubernetes automatically provisions an EBS volume of 10Gi with the parameters defined in the StorageClass and binds it to the claim. The Pod can then mount it transparently through a volume reference.

Dynamic provisioning flow looks like this:

User creates a StorageClass (defines “how” to provision storage).
User creates a PVC referencing that StorageClass. 
Kubernetes dynamically provisions a PV using the parameters and binds it to the PVC. 
The Pod mounts the PVC and uses the volume. 
When the PVC is deleted, the PV is handled according to the reclaim policy.
StorageClasses help in dynamic volume provisioning and bring automation, flexibility, and standardization to Kubernetes storage management, making persistent storage truly dynamic, scalable, and cloud native.

To summarize:

PersistentVolumes (PVs) abstract the underlying storage infrastructure, providing a consistent and unified interface across cloud, on-premises, and hybrid environments.

PersistentVolumeClaims (PVCs) allow developers to request storage resources dynamically without needing to understand the backend storage implementation details.

StorageClasses (SCs) define how storage is provisioned within the cluster, enabling on-demand volume creation with specific performance tiers, encryption, or replication characteristics. Administrators can create multiple StorageClasses (for example, fast-ssd, standard-hdd, or encrypted-volume) to offer flexible options that suit different workload requirements.

Together, PVs, PVCs, and SCs provide a robust, self-service storage framework that is persistent, portable, and scalable. When used in combination, they enable dynamic provisioning, allowing Kubernetes to automatically allocate and manage storage resources as needed


### Configuration Objects
the 12 factor app mode recommends storing configuration in the environment. Apps require more than code and libraries; they also need onfiguration such as database credentials, connection strings, external service endpoints, and feature settings.

These values often change between environments such as development, staging and production

embedding configurations directly into container images is considered bad practice because any configuration change would require rebuilding the image and redeploying the container or pod. this becomes increasingly problematic when managing multiple environments, each with its own configuration. the 12 factoe app approach addresses this by keeping configuration separate from the applicatio build

this separation is achieved using ConfigMaps, which let you store configuration as key-value pairs or full configuration files indeendently of the pods. 

ConfigMaps can be consumed in two ways:
Mounted as files inside a Pod using a volume
Mapped to environment variables that the container can read at runtime



kubernetes also includes a built-in object for storing sensitive values such as passwords, API keys, and authentication credentials.

These objects are called Secets. Although similar to ConfigMaps, they are intended specifically for confidential data and are stored in an encoded format to prevent accidental exposure.


NB: The base64 encoding is not a security mechanism- it merely ensures data is safely transmitted and stored in a standardized format. Without additional protection, Secrets are not encrypted by default and may be readable in memory or etcd, depending on clcusterconfiguration.

In production environments, many organizations integrate Kubernetes with dedicated secret-management systems that provide encryption, audit controls, and automated rotation. For example, tools like HashiCorp Vault, AWS Secrets Manager, and Google Secret Manager are commonly used to store sensitive values securely while still exposing them to Pods at runtim

#### Autoscaling Objects
kubernetes supports multiple mechanisms for automatically scaling workloads, depending on whether you need to scale pods, nodes or resource allocations

## Autoscaling Mechanisms
Horizontal pod Autoscaler(HPA) is the most commonly used autoscaler. It monitors metrics such as CPU or memory usage and increases or decreases the no. of pod replicas when threshholds are reached.

## Cluster Autoscaler
Scaling pods isn't useful if the cluster itself has no remaining compute capacity. The Cluster Autoscaler adds (or removes) worker nodes when resource demand changes. It is often used alongside the Horizontal pod Autoscaler so that pods scale up first, and the cluster expands only when neeeded. 

### Vertical Pod Autoscaler

it adjusts a pod's cpu and memory requests and limis dynamically rather than adding more replicas. this is useful for workloads that scale better by increasing resources per pod rather than adding more pods. However, vertical scaling is constrained by the capacity o individual nodes and becomes less effective once limits are reacheddd.


### Scheduling Objects

### Kubernetes Security
Because kubernetes runs as part of a distributed ecosystem, securing the cluster requires hardening not just kubernetes itself, but the full stack: hardware, firmware, OS, container runtimes, and the configuration layers that tie them together.

Security must begin at system design time, not after deployment.

After the platform is secured, the kube-apiserver must be carefully configured with policies, access controls, and authentication tools that formalize and restrict access in a managable way.

Since kubernetes is a highly networked environment, network security is critical both outside and inside the cluster. External protections may include firewalls and perimeter controls, while internal security relies on pod-to-pod encryption, NetworkPolicy, and other in-cluster enforcement mechanisms

Container security also plays a significant role. Reducing base image size enforcing container immutability, and performing static and runtime analysis help ensure workloads are safe before they reach production.

Many of these safeguards begin in development and must be integrated into the CI/CD Pipeline. Additional tools such as AppArmor and SELinux can add another layer of defense by restricting whatprocesses inside containers are allowed to do. 

#### Accessing the API

To perform any action in a kubernetes cluster, you need to access the APIand go theough several steps
1. Authentication(token)
2. Authorization (RBAC)
3. Admission Controllers

#### Pod Security standards and Admission
Kubernetes offers a higly flexible environment for running containerized workloads, but without proper guardrails, workloads can operate with unnecessary privileges, increasing risk.

To address this pod security standards define three consistent and predictable security levels that control how pods interact with the host, other workloads, and system- level resources. These levels, such as Privileged, Baseline, and Restricted, each represent increasing degrees of security.

The privileged level is fully permissive and intended only for trusted system services or administrative tasks

The baseline is suitable for most workloads, blocking dangerous configurations like host networking and privilege escalation

The restricted level enforces the strongest protections, requiring non-root execution, limited linux capabilities, and strict seccomp profiles

Together, these policies help maintain consistency, ensure compliance with best practices, and prevent unsafe configurations across the cluster.


The Pod Security Admission controller is the enforcement mechanism or these standards. It evaluates every pod at creation time to confirm compliance with the assigned security level for its namespace. 

PSA operates through simple namespace labels, which define enforcement, warning and audit behaviors.

This design allows cluster administrators to apply different security levels per namespace and gradually tighten policies. 