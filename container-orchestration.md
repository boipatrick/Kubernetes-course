The evolution of application development has always been closely tied to how applications are packaged for different platforms and OS's

The process of setting up infrastructure, installing dependencies and configuring the environment that the application runs on can be error-prone and difficult to maintain, which is why servers are often configured for a single-dedicated purpose- such as hosting a database and then connected via a network. 

To maximize server hardware efficiency, organizations use   Vms which emulate complete servers. This approach allows multiple isolated environments to operate on a single physical machine.

Before containers, VMs were the standard however because each VM runs its own os and kernel it introduces resource overhead when scaling to many instances.

Containers solve these issues by packaging applications with their dependencies while sharing the host system's kernel. This makes them lighter, faster, and more efficient than running multiple full virtual machines.

## Running Containers
you don't need docker you can follow the OCI runtime-spec standard instead. 

## Building Container Images
container images make containers portable and easy to reuse across a vaiety of systems. 

A docker container image is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings.

## Security
Containers have different security requirements than vms. 
one of the most serious security risks both in containerized and traditional environments is running processes with excessive privileges, especially as the root or administrator use. 

Dockerhub and other registries make it easy to access and share container images, but they also pose potential threats. It is essential to verify that publicly available images have not been tampered with or contain malicious software before using them. 

The 4C's of Cloud Native security provide a helpful framework for identifying which layters to protect when using containers. It's essential to secure each layer, as every layer strengthens the protection of those within it. 

## Container Orchestration Fundamentals
Running a few containers on your local machine or a single server is relativelly simple, but scaling this approach introduces new operational challenges. 

Because containers are lightweight and efficient, modern applications are often built from many small, specialized components. 

This shift has led to the rise of microservice architectures, where applications are composed of numerous small, independent containers. 

Each container encapsulates a specific piece of business logic, allowing the overall system to be more modular, flexible, and easier to maintain. 

When managing and deploing a large number of containers, it quickly becomes clear that you need a dedicated system to orchestrate and manage them efficiently. Problems to be solved can include:
1. provide compute resources, like vms, where containers can run on
2. Schedule containers to servers in an efficient way.
3. allocate resources like cpu and memory to containers
4. manage the availability of containers and replace them if they fail.
scale cotainers if the load increases
5. provide networking to connect containers
6. provision storage if containers need to persist data

container orchestration systems allow you to create a cluster of multiple servers and run containers across them efficiently. 

These systems typically have two main components : a control panel, which manages the cluster and container operations, and worker nodes which actually run the containers.

## Networking

Microservice architecture relies heavily on network communication

Unlike monolithic apps, each microservice exposes an interface that other services cab call, for instance, a service that returns a product list for an ecommerce application. 

To support this, network namespaces give each container its won IP address, allowing multiplecontainers touse the same port- such as several web servers all listening on port 8080. Containers can also map internal ports to host system ports, making applications accessible externally.


Most modern container networking systems use the container network interface, a standard that defines how network plugins should connect containers. CNI makes it simple to configure, manage, and replace different networking plugins across container orchestration platforms.

## Service Discovery & DNS

In traditional data centers, server management was relatively simple many system administraors could even recall the IP addresses of key systems from memory.

Lists of servers, hostnames, and purposes were often maintained manually. 

In modern container orchestrartion platforms, however, this approach no longer works. You may have:
Hundreds or thousands of containers, each with its own IP address.
Containers distributed across mutlitple hosts, data centere or regions
The need for DNS-based communication, since tracking IPs manually is impractical
Dynamic environments where containers are constantly created and deleted.

The solution is automation through a service registry- a sytem that automatically tracks and updates information about running services. This enables service discovery, allowing containers to find and communicate with each oher dynamically without manual configuration.

## Service Mesh
Beyond simply connecting containers,teams often need advanced capabilities, such as monitoing, access control, and encryption, to ensure secure communication between services

Instead of building these capabilities directly into every application, developers can delegate them to a proxy. This dedicated server application sits between clients and servers to manage, filter, or modify network traffic. 

A service mesh extends this concept by automatically deploying a lightweight proxy alongside every container in your system. These proxies handle all network communicaation between services, abstracting away complexity from application code

## Storage
The challenge with the read-write layer is that it disappears when the container is stopped or removed much like your computer's memory is cleared when you shut it down. To preserve data beyond the container's lifecycle, it must be written to persistent storage. 

This is where volumes come in. When a container needs to store data permanently on the host, a volume provides a simple solution: instead of isolating the entire filesystem, specific directories from the host are mounted directly into the container.

However this reduces isolation. By using a volume, you are effectively giving the container controlled access to part of the hosts' filesystem 