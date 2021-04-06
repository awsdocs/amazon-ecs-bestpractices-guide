# Networking between Amazon ECS services in a VPC<a name="networking-connecting-services"></a>

Applications often start out as monolithic codebases, but over time various business functions need to be split out into their own deployments that can be deployed or scaled independently\. At that point you need your Amazon ECS services to be able to network with each other\. For example, you might have a customer facing API service that makes use of an internal user service that stores your customers personal identifying information in a compliant manner\. You would need to ensure that the API service can locate and communicate with the user service\.

There are multiple ways to solve this problem, depending on how complex your architecture is and how many services you have\.

## Using service discovery<a name="networking-connecting-services-direct"></a>

One method for service to service communication is direct communication using service discovery\. In this approach, you can use the AWS Cloud Map service discovery integration with Amazon ECS\. Using service discovery, Amazon ECS syncs the list of launched tasks to AWS Cloud Map, which maintains a DNS hostname that resolves to the internal IP addresses of one or more tasks from that particular service\. Other services in the Amazon VPC can use this DNS hostname to send traffic directly to another container using its internal IP address\. For more information, see [Service discovery](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-discovery.html) in the *Amazon Elastic Container Service Developer Guide*\.

![\[Diagram showing architecture of a network using service discovery.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/servicediscovery.png)

In the diagram above there are three services\. `serviceA` has one container and needs to talk to `serviceB` that has two containers\. `serviceB` needs to talk to `serviceC` which has one container\. Each container in these services are able to use the internal DNS names from AWS Cloud Map to find the internal IP addresses of a container from the downstream service that it needs to communicate to\.

This approach for service to service communication has low latency, and at first glance it is also simple as there are no extra components between the containers\. Traffic just travels directly from one container to the other container\.

This approach works best when using the `awsvpc` network mode, where each task has its own unique IP address\. Most software only supports the use of DNS `A` records, which resolve directly to IP addresses\. When using the `awsvpc` network mode, the IP address for each task would be an `A` record\. If you are using `bridge` network mode, multiple containers may be sharing the same IP address\. Additionally, dynamic port mappings cause the containers to be randomly assigned port numbers on that single IP address\. At this point, an `A` record would no longer be enough for service discovery\. You would need to use an `SRV` record\. This type of record can keep track of both IP addresses and port numbers, but will usually require some extra effort for your application to make use of\. Some prebuilt applications that you use may not support `SRV` records\.

Additionally, the `awsvpc` network mode allows you to have a unique security group for each service\. This security group can be configured to allow incoming connections from only the specific upstream services that need to talk to that service\.

The main downside of direct service to service communication using service discovery is that you will need to implement extra logic to have retries and deal with connection failures\. DNS records have a time to live \(TTL\) period that controls how long they are cached for\. It takes some time for the DNS record to be updated and for the cache to expire so that your applications can pick up the latest version of the DNS record\. So your application may end up resolving the DNS record to point at another container that is no longer there\. Your application needs to handle retries and have logic to ignore bad backends\.

## Using an internal load balancer<a name="networking-connecting-services-elb"></a>

Another method for service to service communication is to use an internal load balancer\. An internal load balancer lives entirely inside of your VPC and is only accessible to services inside of your VPC\.

![\[Diagram showing architecture of a network using an internal load balancer.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/loadbalancer-internal.png)

The load balancer maintains high availability by deploying redundant resources into each subnet\. When a container from `serviceA` wants to talk to a container from `serviceB` it opens a connection to the load balancer\. The load balancer then opens a connection to a container from `service B`\. The load balancer serves as a centralized place for managing all connections between each service\.

If a container from `serviceB` stops, then the load balancer is able to remove that container from the pool\. The load balancer also does health checks against each downstream target in its pool and can automatically remove bad targets from the pool until they become healthy again\. The applications no longer need to be aware of how many downstream containers are there\. They just open their connections to the load balancer\.

This approach works well with all network modes\. The load balancer is capable of keeping track of task IP addresses when using the `awsvpc` network mode, as well as more advanced combinations of IP address and port when using the `bridge` network mode\. It will evenly distribute traffic across all the IP address and port combinations, even if several containers are actually hosted on the same Amazon EC2 instance, just on different ports\.

The one downside of using an internal load balancer is cost\. In order to be highly available, the load balancer needs to have resources in each Availability Zone\. This adds extra cost because of the overhead of paying for the load balancer and for the amount of traffic that goes through the load balancer\.

You can reduce the cost overhead by having multiple services share a load balancer\. This works especially well with REST style services that use an Application Load Balancer\. You can create path based routing rules that route traffic to different services\. For example `/api/user/*` might go to a container that is part of the `user` service, while `/api/order/*` might go to the associated `order` service\. With this approach, you only pay for one Application Load Balancer, and have one consistent URL for your API, but you can split the traffic off to various microservices on the backend\.

## Using a service mesh<a name="networking-connecting-services-appmesh"></a>

AWS App Mesh is a service mesh, which helps you when you have a large number of services and want more control over how traffic gets routed between multiple services\. App Mesh could be considered a halfway point between basic service discovery, and load balancing\. Applications no longer talk directly to each other, but they also don’t use a centralized load balancer\. Instead, each copy of your task is accompanied by an Envoy proxy sidecar\. For more information, see [What is AWS App Mesh](https://docs.aws.amazon.com/app-mesh/latest/userguide/what-is-app-mesh.html) in the *AWS App Mesh User Guide*\.

![\[Diagram showing architecture of a network using a service mesh.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/appmesh.png)

In the diagram above, each task has an Envoy proxy sidecar\. This sidecar is responsible for proxying all inbound and outbound traffic for the task\. The App Mesh control plane uses AWS Cloud Map to get the list of available services and the IP addresses of specific tasks\. Then App Mesh delivers the configuration to the Envoy proxy sidecar\. This configuration includes the list of available containers that can be connected to\. The Envoy proxy sidecar will also do its own healthchecks against each target to ensure that they are available\.

This approach combines the power of service discovery, with the ease of the managed load balancer\. Applications do not have to implement as much load balancing logic within their application code, because the Envoy proxy sidecar handles that load balancing\. The Envoy proxy can be configured to detect failures and retry failed requests\. Additionally, the Envoy proxy can be configured to use mTLS to encrypt traffic in transit, as well as ensure that your applications are communicating to a verified destination\.

There are a few differences between an Envoy proxy and a load balancer like an Application Load Balancer and Network Load Balancer\. Rather than paying for a fully managed load balancer, you are responsible for deploying and managing your own Envoy proxy sidecar which uses some of the CPU and memory that you allocate to the Amazon ECS task\. This adds a little bit of extra overhead to the task resource consumption, as well as additional operational workload to maintain and update the proxy when needed\.

App Mesh and an Envoy proxy enables extremely low latency between tasks because the Envoy proxy runs collocated to each task\. There is only one instance to instance network jump, between one Envoy proxy and another Envoy proxy\. This means there is less network overhead compared to the external load balancer approach, where there are two network jumps: from the upstream task to the load balancer, then from the load balancer to the downstream task\.