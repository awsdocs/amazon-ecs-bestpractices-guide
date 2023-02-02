# Networking between Amazon ECS services in a VPC<a name="networking-connecting-services"></a>

Using Amazon ECS containers in a VPC, you can split monolithic applications into separate parts that can be deployed and scaled independently in a secure environment\. However, it can be challenging to make sure that all of these parts, both in and outside of a VPC, can communicate with each other\. There are several approaches for facilitating communication, all with different advantages and disadvantages\.

## Using service discovery<a name="networking-connecting-services-direct"></a>

One approach for service\-to\-service communication is direct communication using service discovery\. In this approach, you can use the AWS Cloud Map service discovery integration with Amazon ECS\. Using service discovery, Amazon ECS syncs the list of launched tasks to AWS Cloud Map, which maintains a DNS hostname that resolves to the internal IP addresses of one or more tasks from that particular service\. Other services in the Amazon VPC can use this DNS hostname to send traffic directly to another container using its internal IP address\. For more information, see [Service discovery](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-discovery.html) in the *Amazon Elastic Container Service Developer Guide*\.

![\[Diagram showing architecture of a network using service discovery.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/servicediscovery.png)

In the preceding diagram, there are three services\. `serviceA` has one container and communicates with `serviceB`, which has two containers\. `serviceB` must also communicate with `serviceC`, which has one container\. Each container in all three of these services can use the internal DNS names from AWS Cloud Map to find the internal IP addresses of a container from the downstream service that it needs to communicate to\.

This approach to service\-to\-service communication provides low latency\. At first glance, it's also simple as there are no extra components between the containers\. Traffic travels directly from one container to the other container\.

This approach is suitable when using the `awsvpc` network mode, where each task has its own unique IP address\. Most software only supports the use of DNS `A` records, which resolve directly to IP addresses\. When using the `awsvpc` network mode, the IP address for each task are an `A` record\. However, if you're using `bridge` network mode, multiple containers could be sharing the same IP address\. Additionally, dynamic port mappings cause the containers to be randomly assigned port numbers on that single IP address\. At this point, an `A` record is no longer be enough for service discovery\. You must also use an `SRV` record\. This type of record can keep track of both IP addresses and port numbers but requires that you configure applications appropriately\. Some prebuilt applications that you use might not support `SRV` records\.

Another advantage of the `awsvpc` network mode is that you have a unique security group for each service\. You can configure this security group to allow incoming connections from only the specific upstream services that need to talk to that service\.

The main disadvantage of direct service\-to\-service communication using service discovery is that you must implement extra logic to have retries and deal with connection failures\. DNS records have a time\-to\-live \(TTL\) period that controls how long they are cached for\. It takes some time for the DNS record to be updated and for the cache to expire so that your applications can pick up the latest version of the DNS record\. So, your application might end up resolving the DNS record to point at another container that's no longer there\. Your application needs to handle retries and have logic to ignore bad backends\.

## Using an internal load balancer<a name="networking-connecting-services-elb"></a>

Another approach to service\-to\-service communication is to use an internal load balancer\. An internal load balancer exists entirely inside of your VPC and is only accessible to services inside of your VPC\.

![\[Diagram showing architecture of a network using an internal load balancer.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/loadbalancer-internal.png)

The load balancer maintains high availability by deploying redundant resources into each subnet\. When a container from `serviceA` needs to communicate with a container from `serviceB`, it opens a connection to the load balancer\. The load balancer then opens a connection to a container from `service B`\. The load balancer serves as a centralized place for managing all connections between each service\.

If a container from `serviceB` stops, then the load balancer can remove that container from the pool\. The load balancer also does health checks against each downstream target in its pool and can automatically remove bad targets from the pool until they become healthy again\. The applications no longer need to be aware of how many downstream containers there are\. They just open their connections to the load balancer\.

This approach is advantageous to all network modes\. The load balancer can keep track of task IP addresses when using the `awsvpc` network mode, as well as more advanced combinations of IP addresses and ports when using the `bridge` network mode\. It evenly distributes traffic across all the IP address and port combinations, even if several containers are actually hosted on the same Amazon EC2 instance, just on different ports\.

The one disadvantage of this approach is cost\. To be highly available, the load balancer needs to have resources in each Availability Zone\. This adds extra cost because of the overhead of paying for the load balancer and for the amount of traffic that goes through the load balancer\.

However, you can reduce overhead costs by having multiple services share a load balancer\. This is particularly suitable for REST services that use an Application Load Balancer\. You can create path\-based routing rules that route traffic to different services\. For example, `/api/user/*` might route to a container that's part of the `user` service, whereas `/api/order/*` might route to the associated `order` service\. With this approach, you only pay for one Application Load Balancer, and have one consistent URL for your API\. However, you can split the traffic off to various microservices on the backend\.

## Using a service mesh<a name="networking-connecting-services-appmesh"></a>

AWS App Mesh is a service mesh that can help you manage a large number of services and have better control of how traffic gets routed among services\. App Mesh functions as an intermediary between basic service discovery and load balancing\. With App Mesh, applications don't directly interact with each other, but they also don’t use a centralized load balancer either\. Instead, each copy of your task is accompanied by an Envoy proxy sidecar\. For more information, see [What is AWS App Mesh](https://docs.aws.amazon.com/app-mesh/latest/userguide/what-is-app-mesh.html) in the *AWS App Mesh User Guide*\.

![\[Diagram showing architecture of a network using a service mesh.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/appmesh.png)

In the preceding diagram, each task has an Envoy proxy sidecar\. This sidecar is responsible for proxying all inbound and outbound traffic for the task\. The App Mesh control plane uses AWS Cloud Map to get the list of available services and the IP addresses of specific tasks\. Then App Mesh delivers the configuration to the Envoy proxy sidecar\. This configuration includes the list of available containers that can be connected to\. The Envoy proxy sidecar also conducts health checks against each target to ensure that they're available\.

This approach provides the features of service discovery, with the ease of the managed load balancer\. Applications don't implement as much load balancing logic within their code because the Envoy proxy sidecar handles that load balancing\. The Envoy proxy can be configured to detect failures and retry failed requests\. Additionally, it can also be configured to use mTLS to encrypt traffic in transit, and ensure that your applications are communicating to a verified destination\.

There are a few differences between an Envoy proxy and a load balancer\. In short, with Envoy proxy, you're responsible for deploying and managing your own Envoy proxy sidecar\. The Envoy proxy sidecar uses some of the CPU and memory that you allocate to the Amazon ECS task\. This adds some overhead to the task resource consumption, and additional operational workload to maintain and update the proxy when needed\.

App Mesh and an Envoy proxy allows for extremely low latency between tasks\. This is because Envoy proxy runs collocated to each task\. There's only one instance to instance network jump, between one Envoy proxy and another Envoy proxy\. This means there's also less network overhead compared to when using load balancers\. When using load balancers, there are two network jumps\. The first is from the upstream task to the load balancer, and the second is from the load balancer to the downstream task\.