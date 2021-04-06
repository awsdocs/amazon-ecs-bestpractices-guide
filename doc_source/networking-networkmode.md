# Choosing a network mode<a name="networking-networkmode"></a>

The methods for architecting inbound and outbound network connections apply to all workloads on AWS, even if they aren’t inside a container\. When running containers on AWS, there is another level of networking to consider\. One of the main reasons to use containers is that you can pack multiple containers onto a single host\. When doing this, you need to choose how you want to network the containers that are running on the same host\. The following are the options to choose from\.

**Topics**
+ [Host mode](#networking-networkmode-host)
+ [Bridge mode](#networking-networkmode-bridge)
+ [AWSVPC mode](#networking-networkmode-awsvpc)

## Host mode<a name="networking-networkmode-host"></a>

The `host` network mode is the most basic network mode supported in Amazon ECS\. Using host mode, the networking of the container is tied directly to the underlying host that is running the container\.

![\[Diagram showing architecture of a network with containers using the host network mode.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/networkmode-host.png)

Using the example above, imagine you are running a Node\.js container with an Express application that listens on port `3000`\. When the `host` network mode is used, the container receives traffic on port 3000 using the IP address of the underlying host Amazon EC2 instance\. There is no mapping or virtual network\. Instead, the container ports are mapped directly to the same ports on the Elastic Network Interface \(ENI\) attached to the underlying Amazon EC2 instance\.

While this approach is simple, there are significant drawbacks to using this network mode\. You can’t run more than a single instantiation of a task on each host, as only the first task will be able to bind to its required port on the Amazon EC2 instance\. Also there is no way to remap a container port when using `host` network mode\. If the application the container is running wants to listen on a particular port number, you can’t remap that port number\. As a result, any port conflicts must be managed by changing the configuration of the application\.

There are also security implications when using the `host` network mode\. This mode allows containers to impersonate the host, and it allows containers to connect to private loopback network services on the host\.

The `host` network mode is only supported for Amazon ECS tasks hosted on Amazon EC2 instances\. It is not supported when using Amazon ECS on Fargate\.

## Bridge mode<a name="networking-networkmode-bridge"></a>

Using `bridge` mode, you are using a virtual network bridge to create a layer between the host and the container’s networking\. This allows you to create port mappings that remap a host port to a container port\. The mappings can be either static or dynamic\.

![\[Diagram showing architecture of a network using bridge network mode with static port mapping.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/networkmode-bridge.png)

With a static port mapping, you explicitly define which host port you want to map to a container port\. Using the example above, port `80` on the host is being mapped to port `3000` on the container\. To communicate to the containerized application, you send traffic to port `80` to the Amazon EC2 instance's IP address\. From the containerized application’s perspective it sees that inbound traffic on port `3000`\.

Static port mappings work well if you just want to change the traffic port, but they still have the same downside that using the `host` network mode does\. You can't run more than a single instantiation of a task on each host, as a static port mapping only allows a single container to be mapped to port 80\.

To solve this problem, many people use the `bridge` network mode with a dynamic port mapping as shown in the following diagram\.

![\[Diagram showing architecture of a network using bridge network mode with dynamic port mapping.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/networkmode-bridge-dynamic.png)

By not specifying a host port in the port mapping, we can have Docker choose a random, unused port from the ephemeral port range and assign that as the public host port for the container\. For example, the Node\.js application listening on port `3000` on the container might be assigned a random high number port like `47760` on the Amazon EC2 host\. This allows you to run multiple copies of that container on the host and each container will be assigned its own port on the host\. From the containerized application perspective each copy of the container is receiving traffic on port `3000` as usual, but clients that are sending traffic to these containers are using the randomly assigned host ports\.

Amazon ECS helps you to keep track of the randomly assigned ports for each task, by automatically updating load balancer target groups and AWS Cloud Map service discovery to have the list of task IP addresses and ports\. This makes it easier to use services operating using `bridge` mode with dynamic ports\.

However, one downside of using the `bridge` network mode is that it's much harder to lock down service to service communications\. Because services may be assigned to any random port, it is necessary to open broad port ranges between hosts and there is no easy way to create specific rules so that a particular service can only talk to one other specific service\. The services have no specific ports to use for security group networking rules\.

The `bridge` network mode is only supported for Amazon ECS tasks hosted on Amazon EC2 instances\. It is not supported when using Amazon ECS on Fargate\.

## AWSVPC mode<a name="networking-networkmode-awsvpc"></a>

With the `awsvpc` network mode, Amazon ECS creates and manages an Elastic Network Interface \(ENI\) for each task and each task receives its own private IP address within the VPC\. This ENI is separate from the underlying hosts ENI\. If an Amazon EC2 instance is running multiple tasks, then each task’s ENI is separate as well\.

![\[Diagram showing architecture of a network using the AWSVPC network mode.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/networkmode-awsvpc.png)

In the example above, the Amazon EC2 instance has its own ENI, which represents that EC2 instance’s IP address that is used for network communications at the host level\. Each task has its own ENI as well, and its own private IP address\. Because each ENI is separate, each container can bind to port `80` on the task ENI\. Rather than having to keep track of port numbers, you can send traffic to port `80` at the IP address of the task ENI\.

The advantage of using the `awsvpc` network mode is that each task can now have its own security group to allow or deny traffic\. This gives you the ability to control communication between tasks and services at a more granular level\. You can even configure a task to deny incoming traffic from another task that is located on the same host\.

The `awsvpc` network mode is supported for Amazon ECS tasks hosted on both Amazon EC2 and Fargate\. When using Fargate, the `awsvpc` network mode is required\.

When using the `awsvpc` network mode there are a few challenges you should be mindful of\.

### Increasing task density with ENI Trunking<a name="networking-networkmode-awsvpc-enitrunking"></a>

The biggest challenge when using the `awsvpc` network mode with tasks hosted on Amazon EC2 instances is that EC2 instances have a limit on the number of ENI’s that can be attached to them\. This imposes a limit on how many tasks you can place on each instance\. Amazon ECS provides the ENI trunking feature which increases the number of available ENIs to achieve more task density\.

![\[Diagram showing architecture of a network using AWSVPC network mode with ENI trunking.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/networkmode-awsvpc-eni.png)

When using ENI trunking, two ENI attachments are consumed by default\. The first is the primary ENI of the instance, which is used for any host level processes\. The second is the trunk ENI which Amazon ECS creates\. This feature is only supported on specific Amazon EC2 instance types\.

As an example, without ENI trunking a `c5.large` instance that has 2 vCPU’s can only host 2 tasks\. With ENI trunking, a `c5.large` instance that has 2 vCPU’s can host 10 tasks, each with its own IP address and security group\. For more information on the available instance types and their density, see [Supported Amazon EC2 instance types](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/container-instance-eni.html#eni-trunking-supported-instance-types) in the *Amazon Elastic Container Service Developer Guide*\.

ENI trunking has no impact on runtime performance in terms of latency or bandwidth, but it does increase task startup time\. If ENI trunking is used, you should ensure that your autoscaling rules and other workloads that depend on task startup time still behave as you expect\.

For more information, see [Elastic network interface trunking](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/container-instance-eni.html) in the *Amazon Elastic Container Service Developer Guide*\.

### Preventing IP address exhaustion<a name="networking-networkmode-awsvpc-ipexhaustion"></a>

Each task having its own IP address makes your infrastructure easier to understand and allows you to make more secure security groups\. However this can lead to IP exhaustion quickly\.

The default VPC on your AWS account has pre\-provisioned subnets that have a `/20` CIDR range, which means each subnet has 4,091 available IP addresses\. \(Several IP addresses within the `/20` range are reserved for AWS specific usage\.\) So if you are distributing your applications across three subnets in three Availability Zones for high availability you would have around 12,000 available IP addresses across those three subnets\.

Using ENI trunking, each Amazon EC2 instance you launch requires two IP addresses: one for the primary ENI and another for the trunk ENI\. Additionally, each Amazon ECS task on the instance requires an IP address\. If you are launching an extremely large workload you may run out of IP addresses at some point\. This would show up as Amazon EC2 launch or task launch failures, as the ENI’s would be unable to get IP addresses inside the VPC\.

When using the `awsvpc` network mode, you should evaluate your IP address needs and ensure that your subnet CIDR ranges meet your needs\. However, if you start with a VPC that has small subnets and begin to run out of address space you can add a secondary subnet to give yourself more room\.

![\[Diagram showing architecture of a network using AWSVPC network mode with ENI trunking.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/networkmode-awsvpc-secondary.png)

Using ENI trunking, the Amazon VPC CNI is configured to use ENIs in a different IP address space than the host\. This allows you to give your Amazon EC2 host and your tasks different IP address ranges that don't overlap\. For example, in the diagram above, the EC2 host IP address is in subnet that has the `172.31.16.0/20` IP range\. The tasks that are running on the host are actually being assigned IP addresses from the `100.64.0.0/19` range\. With two independent IP ranges you don’t have to worry about tasks consuming too many IP addresses and not leaving enough IP addresses for instances, or vice versa\.

### Using IPv6 dual stack mode<a name="networking-networkmode-awsvpc-dualstack"></a>

The `awsvpc` network mode is compatible with VPCs configured for IPv6 dual stack mode\. A VPC using dual stack mode can communicate over IPv4, IPv6, or both\. Each subnet in the VPC can have both an IPv4 CIDR range as well as an IPv6 CIDR range\. For more information, see [IP addressing in your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-ip-addressing.html) in the *Amazon VPC User Guide*\.

You cannot disable IPv4 support for your VPC and subnets, thus it will not address IPv4 exhaustion issues\. However, the IPv6 support does allow you to use some new capabilities, in particular the egress\-only internet gateway\. An egress\-only internet gateway allows tasks to use their publicly routable IPv6 address to initiate outbound connections to the internet\. But the egress\-only internet gateway does not allow inbound connections from the public internet\. For more information, see [Egress\-only internet gateways](https://docs.aws.amazon.com/vpc/latest/userguide/egress-only-internet-gateway.html) in the *Amazon VPC User Guide*\.