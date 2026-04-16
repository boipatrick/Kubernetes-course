### Overview

Devops introduced new practices and tools that emphasized collaboration, automation, and rapid iteration.

One of the most impactful outcomes was the automation of the deployment pipelines, enabling faster, more frequent, and more reliable releases.


## Application Delivery Fundamentals
Before deployment apps are validated via automated testing
Tests verify functionality, prevent regressions, and ensure that new changes don't break existing behavior. Automated testing is essential for scaling collaboration and enabling rapid release cycles

After code is built and tested, it must be deployed. If the target platform is Kubernetes, the app is packaged into a container and described using YAML manifests. The container image is pushed to a registry and kubernetes pulls and runs that image based on the configuration defined in Deployment or other controller objects.

This ensures a consistent deployment process across clusters and environments

Modern DevOps workflows don't stop at application code. Increasingly, infrastructure and operational configurations are also managed through version control.

This approach known as Infrastructure as code allows teams to define infrastructure- networks, compute resources,storage, security policies-as configuration files rather than a manual, ad-hoc setup. iac makes environments reproducible, reduces human error, and enables developers to participate directly in provisioning via cloud APIs rather than relying solely on operational teams.

Together, version-controlled source code, automated builds, testing pipelines, containerization and IaC form the foundation for automated delivery processes like CI/CD and GitOps- enabling faster, safer and more scalable software releases

### CI/CD
With apps becoming smaller and deployments more frequent automating the deployment process became a natural and necessary evolution.

The DevOps movement emphasized delivering software rapidly and repeatedly but traditional deployment workflows relied on manual steps involving both developers and system admins. These processes were slow, error-prone and often carried a high risk of failure. 

Automation solves these challenges. Modern software teams rely on Continous Integration and Continous Delivery(CI/CD) to automate the build, test, and deployment stages of application delivery-not just code, but for configuration and even infrastructure. 

CI focuses on automaticaly building and testing code whenever changes are made.Combined with version control, CI allows multiple teams to work on the same codebase safely and efficiently.

CD on the other hand automates the deployment of built artifacts to environemtns. In cloud-native workflows, deployments typically progress via development and staging environments before being promoted to production

A CI/CD pipeline automates this entire workflow as a sequence of scripted steps executed on a server or in a containerized environment. Modern pipelines go beyond basic scripting-  they integrate directly with platforms like kubernetes to provide real-time feedback, status reporting and automated rollouts. 


### GitOps
IaC was a breakthrough in how infrastructure is provisioned and managed. By defining environments as code rather than configuring them manually, organizations can deliver infrastructure faster, more consistently and with fewer errors. 

GitOps builds on Iac by using Git as the single source of truth for infrastructure and application state. Instead of applying changes manually or through ad-hoc scripts, all modifications flow via version-controlled operations such as commits, mergers and pull requests. 

Just as developers submit pull requests to propose changes to software, Gitops applies the same workfloow toplatform and infrastructure changes. Each change is reviewed,validated and tested using CI pipelines before being merged, allowing infrastructure updates to follow the same auditing,approval, and collaboration patterns as application code. 

There are two primary models for applying changes in GitOps workflows
- push-based - A CI/CD pipeline executes tooling that applies changes directly to the target platform. Updates are triggered by a commit or merge event.

- pull-based - an agent running inside the platform continously monitors the Git repo and compares the desired state (in Git) to the live state, applying changes automatically when differences are detected. 

Popular pull-based Gitops tools include Flux and Argo CD:
- Argo CD acts as a kuberntes controller that continously reconciles cluster resources with repo configuration

- Flux is built on the GitOps toolkit- a collection of APIs and controllers that can be exended or used to build custom delivery plans


### Argo CD
Powerful GitOps-based CD tool built for Kubernetes. 

It automates app deployment and lifecycle management by using Git repos as the single source of truth.

Whatever configuration is stored in Git defines how tyour cluster should look.

It continuosly monitoes your runnin app in kubernetes and compares them with the manifests stored in Git

If it detects any difference ("drift") it can automatically or manually synchronize the cluster to match the desired in Git.

Some of its key features include:
support for popular configuration tools include plain YA,L
multi-cluster deployment capabilities wjich allow a single Argo CD instance to manage multiple Kuberntes clusters.
fine grained role based access control for multi-team or multi-tenant enviroments.

ability to rollback to previous git commits
Built in health monitoring for kubernetes
A clean web UI and CLI tool for developers and DevOps teams to viualize ans manage application deployments