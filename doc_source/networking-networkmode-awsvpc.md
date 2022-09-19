# AWSVPC mode<a name="networking-networkmode-awsvpc"></a>

With the `awsvpc` network mode, Amazon ECS creates and manages an Elastic Network Interface \(ENI\) for each task and each task receives its own private IP address within the VPC\. This ENI is separate from the underlying hosts ENI\. If an Amazon EC2 instance is running multiple tasks, then each task’s ENI is separate as well\.

![\[Diagram showing architecture of a network using the AWSVPC network mode.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/networkmode-awsvpc.png)

In the preceding example, the Amazon EC2 instance is assigned to an ENI\. The ENI represents the IP address of the EC2 instance used for network communications at the host level\. Each task also has a corresponding ENI and a private IP address\. Because each ENI is separate, each container can bind to port `80` on the task ENI\. Therefore, you don't need to keep track of port numbers\. Instead, you can send traffic to port `80` at the IP address of the task ENI\.

The advantage of using the `awsvpc` network mode is that each task has a separate security group to allow or deny traffic\. This means you have greater flexibility to control communications between tasks and services at a more granular level\. You can also configure a task to deny incoming traffic from another task located on the same host\.

The `awsvpc` network mode is supported for Amazon ECS tasks hosted on both Amazon EC2 and Fargate\. Be mindful that, when using Fargate, the `awsvpc` network mode is required\.

When using the `awsvpc` network mode there are a few challenges you should be mindful of\.

## Increasing task density with ENI Trunking<a name="networking-networkmode-awsvpc-enitrunking"></a>

The biggest disadvantage of using the `awsvpc` network mode with tasks that are hosted on Amazon EC2 instances is that EC2 instances have a limit on the number of ENIs that can be attached to them\. This limits how many tasks you can place on each instance\. Amazon ECS provides the ENI trunking feature which increases the number of available ENIs to achieve more task density\.

![\[Diagram showing architecture of a network using AWSVPC network mode with ENI trunking.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/networkmode-awsvpc-eni.png)

When using ENI trunking, two ENI attachments are used by default\. The first is the primary ENI of the instance, which is used for any host level processes\. The second is the trunk ENI, which Amazon ECS creates\. This feature is only supported on specific Amazon EC2 instance types\.

Consider this example\. Without ENI trunking, a `c5.large` instance that has two vCPUs can only host two tasks\. However, with ENI trunking, a `c5.large` instance that has two vCPU’s can host up to ten tasks\. Each task has a different IP address and security group\. For more information about available instance types and their density, see [Supported Amazon EC2 instance types](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/container-instance-eni.html#eni-trunking-supported-instance-types) in the *Amazon Elastic Container Service Developer Guide*\.

ENI trunking has no impact on runtime performance in terms of latency or bandwidth\. However, it increases task startup time\. You should ensure that, if ENI trunking is used, your autoscaling rules and other workloads that depend on task startup time still function as you expect them to\.

For more information, see [Elastic network interface trunking](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/container-instance-eni.html) in the *Amazon Elastic Container Service Developer Guide*\.

## Preventing IP address exhaustion<a name="networking-networkmode-awsvpc-ipexhaustion"></a>

By assigning a separate IP address to each task, you can simplify your overall infrastructure and maintain security groups that provide a great level of security\. However, this configuration can lead to IP exhaustion\.

The default VPC on your AWS account has pre\-provisioned subnets that have a `/20` CIDR range\. This means each subnet has 4,091 available IP addresses\. Note that several IP addresses within the `/20` range are reserved for AWS specific usage\. Consider this example\. You distribute your applications across three subnets in three Availability Zones for high availability\. In this case, you can use approximately 12,000 IP addresses across the three subnets\.

Using ENI trunking, each Amazon EC2 instance that you launch requires two IP addresses\. One IP address is used for the primary ENI, and the other IP address is used for the trunk ENI\. Each Amazon ECS task on the instance requires one IP address\. If you're launching an extremely large workload, you could run out of available IP addresses\. This might result in Amazon EC2 launch failures or task launch failures\. These errors occur because the ENIs can't add IP addresses inside the VPC if there are no available IP addresses\.

When using the `awsvpc` network mode, you should evaluate your IP address requirements and ensure that your subnet CIDR ranges meet your needs\. If you have already started using a VPC that has small subnets and begins to run out of address space, you can add a secondary subnet\.

![\[Diagram showing architecture of a network using AWSVPC network mode with ENI trunking.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/networkmode-awsvpc-secondary.png)

By using ENI trunking, the Amazon VPC CNI can be configured to use ENIs in a different IP address space than the host\. By doing this, you can give your Amazon EC2 host and your tasks different IP address ranges that don't overlap\. In the example diagram, the EC2 host IP address is in a subnet that has the `172.31.16.0/20` IP range\. However, tasks that are running on the host are assigned IP addresses in the `100.64.0.0/19` range\. By using two independent IP ranges, you don’t need to worry about tasks consuming too many IP addresses and not leaving enough IP addresses for instances\.

## Using IPv6 dual stack mode<a name="networking-networkmode-awsvpc-dualstack"></a>

The `awsvpc` network mode is compatible with VPCs that are configured for IPv6 dual stack mode\. A VPC using dual stack mode can communicate over IPv4, IPv6, or both\. Each subnet in the VPC can have both an IPv4 CIDR range and an IPv6 CIDR range\. For more information, see [IP addressing in your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-ip-addressing.html) in the *Amazon VPC User Guide*\.

You can't disable IPv4 support for your VPC and subnets to address IPv4 exhaustion issues\. However, with the IPv6 support, you can use some new capabilities, specifically the egress\-only internet gateway\. An egress\-only internet gateway allows tasks to use their publicly routable IPv6 address to initiate outbound connections to the internet\. But the egress\-only internet gateway doesn't allow connections from the internet\. For more information, see [Egress\-only internet gateways](https://docs.aws.amazon.com/vpc/latest/userguide/egress-only-internet-gateway.html) in the *Amazon VPC User Guide*\.
