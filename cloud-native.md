Monolithic vs Microservice Architecture
In mono, the UI, business logic and data access layer are tightly integrated intoa single deployable unit that runs on one server.

The microservice breaks the app into multiple independent microservices, each handling a specific function and connected to its own db or storage.

The UI communicates with these services individually, allowing greater flexibility, scalability and ease of updates compared to mono


## Cloud Native Architecture Characteristics
High level Automation 
Self-healing
Scalable
(Cost-) Efficient
Easy to Maintain
Secure by Default

## Autoscaling
the pattern describes the dynamic adjustment of resources in response to current demand.

Here we mainly talk about horizontal scaling, which describes the process of spawning new compute resources-new copies f your application, virtual machines, or, in a less immediate way, new racks of servers and other hardware

vertical scaling refers to the change in size of the underlying hardware, which works within certain hardware limits for both bare-metal and virtual machines.

In simple terms vertical scaling adds more resources(CPU, memory, storage) to a single server to increase its capacity. Horizontal scaling adds multiple servers(A,B,C,D)to distribute workload and improve performance across systems. 


## Serverless
cloud providers claim it is very easy to deploy applications, but they require you to prepare and configure several resourcs, such as networks, virtual machines, os etc to run a simple web app. 
The idea of serveless computing is to relieve developers of these complicated tasks.

## Open Standards
under the Linux foundation, the open container initiative provides two standards that define hoe to build and run containers. The image-spec defines how to build and package container images.
At the same tim, the runtime-spec specifies the configuration, execution environment, and lifecycle of containers. 

Open standards like this help and complement other systems, such as Kubernetes, which is the de facto standard for orchestrating containers. A few standards that will be discovered in the following chapters are:

OCI Spec: image, runtime, and distribution specification on how to run, build, and distribute containers.
Container Network Interface (CNI): A specification on how to implement networking for Containers.
Container Runtime Interface (CRI): A specification on how to implement container runtimes in container orchestration systems.
Container Storage Interface (CSI): A specification on how to implement storage in container orchestration systems.
Service Mesh Interface (SMI): A specification on how to implement Service Meshes in container orchestration systems with a focus on Kubernetes.
Following this approach, other systems like Prometheus and OpenTelemetry evolved and thrived in this ecosystem, providing additional standards for monitoring and observability.

# CNCF Graduation Criteria
Sandbox Stage
Incubating Stage
Grauation Stage