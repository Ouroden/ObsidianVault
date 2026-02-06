### What is Ingress?

The concept of an Ingress exists outside of Kubernetes and extends to other [open source platforms](https://konghq.com/products/kong-gateway). For the process of implementing an Ingress, a set of rules must be established to assist with influencing incoming traffic. In its most basic essence, it relays an HTTP or HTTPS to a service. These services include routing users to a specific component or pod of a service based on how the link is required to interact with the defined rules. 

An Ingress acts as an entry point for incoming traffic into a system or platform. It provides a way to manage and control the routing of external requests to the appropriate internal services or resources. The Ingress is responsible for analyzing incoming requests based on predefined rules and forwarding them to the correct destination within the system. If no rules are specified, all traffic will be sent to a single default backend, which is conventionally a configuration option of the Ingress controller and is not specified in your Ingress resources. This method, also known as the DefaultBackend, is the backend that should handle requests in that case.

The primary purpose of an Ingress is to simplify the management of traffic routing and load balancing for services exposed to the internet or other external networks. It acts as a single point of entry, eliminating the need to configure individual services with their own external IP addresses or load balancers.

## What is Ingress in Kubernetes (K8s)?

Kubernetes itself is an open source platform used to containerize various components like software, applications, and more. These containers are defined as pods, and multiple pods are called a cluster. Using various services or types to define the cluster, the developer can choose how traffic is routed to the cluster or individual pods in specific ways. One of the key components in Kubernetes is Ingress, which acts as a gateway for external traffic to reach the cluster. Ingress can be configured to route traffic to different backend services, such as an app, based on specific rules and configurations.

Ingress is a Kubernetes resource that manages external access to services within a cluster. It acts as a single entry point for incoming traffic, routing it to the appropriate services based on the routing rules the developer sets. Kubernetes Ingress provides a way to securely and efficiently expose services. It eliminates the need for manual management of load balancers or public IP addresses for individual services.

When using ingress to influence how traffic is routed to respective clusters, the developer can define a set of rules to enable specific directions for the incoming traffic. In a cluster, there can be multiple sets of pods that are defined as different services. Using the prior set of rules, specific links used by incoming traffic will send them to a corresponding service with the precedence given to the longest matching path.

The key components of Kubernetes Ingress are:

1. Ingress Controller: This is a specialized load balancer that watches the Ingress resource and processes the rules defined within it. It is responsible for routing incoming traffic to the appropriate services based on the Ingress rules.
2. Ingress Resource: This is a Kubernetes resource that defines the rules for routing incoming traffic. It specifies the hostname, path, and destination services for different types of requests.
3. Ingress Rules: These are the rules defined within the Ingress resource that determine how incoming traffic should be routed. Rules can be based on host, path, or other criteria




https://konghq.com/blog/learning-center/what-is-kubernetes-ingress

#kubernetes #ingress #devops 