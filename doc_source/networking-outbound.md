# Connecting to the Internet<a name="networking-outbound"></a>

Many containerized applications have a least a few components that need outbound access to the Internet\. For example, the backend for a mobile app that needs to talk to the Apple Push Notification service\. Or a social media application that allows users to post website links, and it needs to fetch the content of a user submitted website link in order to generate a preview card for the link based on the Open Graph tags on the HTML page\.

Amazon Virtual Private Cloud has two main methods for allowing communication between your VPC and the internet\.

**Topics**
+ [Using a public subnet and internet gateway](#networking-public-subnet)
+ [Using a private subnet and NAT gateway](#networking-private-subnet)

## Using a public subnet and internet gateway<a name="networking-public-subnet"></a>

![\[Diagram showing architecture of a public subnet connected to an internet gateway.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/public-network.png)

Using a public subnet that has a route to an internet gateway, your containerized application runs on a host inside a VPC on a public subnet\. The host that is running your container is assigned a public IP address which is routable from the Internet\. For more information, see [Internet gateways](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html) in the *Amazon VPC User Guide*\.

This network architecture allows direct communication between the host that is running your application and other hosts on the Internet\. The communication is bi\-directional; You can establish an outbound connection to any other host on the internet, but other hosts on the Internet may attempt to connect to your host as well\. As a result, you need to pay extra attention to security group and firewall rules to ensure that other hosts on the Internet can’t open any connections that you didn’t intend\.

For example, if you are running on Amazon EC2, you want to ensure that port 22 for SSH access is locked down; otherwise it is likely that your instance will receive constant SSH connection attempts from bots\. These bots trawl through public IP addresses, and once they find an open SSH port they attempt to brute\-force passwords in the hopes of gaining access to your instance\. For this reason, many organizations limit the usage of public subnets and prefer to have most, if not all, of their resources inside of private subnets\.

However, using public subnets for networking is ideal for applications that are customer\-facing and require large amounts of bandwidth or the lowest possible latency\. Examples of such applications might include video streaming, or video game servers\.

This networking approach is supported when using both Amazon ECS on Amazon EC2 and on AWS Fargate\.
+ Using Amazon EC2 — Launch EC2 instances on a public subnet\. Amazon ECS uses these EC2 instances as cluster capacity, and any containers running on the instances will be able to use the underlying public IP address of the host for outbound networking\. This applies when using the `host` and `bridge` network modes, but the `awsvpc` network mode does not provide task ENIs with public IP addresses, so they can’t make direct use of an internet gateway\.
+ Using Fargate — When creating your Amazon ECS service, specify public subnets for the networking configuration of your service, and ensure that the **Assign public IP address** option is enabled\. Each Fargate task is networked in the public subnet, and has its own public IP address for direct communication with the Internet\.

## Using a private subnet and NAT gateway<a name="networking-private-subnet"></a>

![\[Diagram showing architecture of a private subnet connected to a NAT gateway.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/private-network.png)

Using a private subnet and a NAT gateway, your containerized application runs on a host that is in a private subnet\. The host has a private IP address that is routable inside your VPC, but isn't routable from the Internet\. This means that other hosts inside the VPC can make connections to the host using its private IP address, but other hosts on the internet can not make any inbound communications to the host\.

With a private subnet, you can use a Network Address Translation \(NAT\) gateway to enable a host inside a private subnet to connect to the Internet\. Hosts on the Internet receive an inbound connection that appears to be coming from the NAT gateway’s public IP address inside a public subnet\. The NAT gateway is responsible for serving as a bridge between the Internet and the private VPC\. This configuration is often preferred for security reasons because it means that your VPC is protected from direct access by attackers from the Internet\. For more information, see [NAT gateways](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) in the *Amazon VPC User Guide*\.

This private networking approach is ideal for workloads that you want to protect from direct external access, such as payment processing or user data and passwords\. You are charged for creating and using a NAT gateway in your account\. NAT gateway hourly usage and data processing rates apply as well\. For redundancy, you should have a NAT gateway in each Availability Zone so that the loss of a single Availability Zone doesn't compromise your outbound connectivity\. For this reason, it may not be cost effective to use private subnets and NAT gateways if you have a small workload\.

This networking approach is supported when using both Amazon ECS on Amazon EC2 and on AWS Fargate\.
+ Using Amazon EC2 — Launch EC2 instances on a private subnet\. The containers that run on these EC2 hosts use the underlying hosts networking, and outbound requests will go through the NAT gateway\.
+ Using Fargate — When creating your Amazon ECS service, specify private subnets for the networking configuration of your service, and don't enable the **Assign public IP address** option\. Each Fargate task is hosted in a private subnet, and its outbound traffic is routed through any NAT gateway you have associated with that private subnet\.