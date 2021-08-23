# Network security<a name="security-network"></a>

Network security is a broad topic that encompasses several subtopics\. These include encryption\-in\-transit, network segmentation and isolation, firewalling, traffic routing, and observability\.

## Encryption in transit<a name="security-network-encryption"></a>

Encrypting network traffic prevents unauthorized users from intercepting and reading data when that data is transmitted across a network\. With Amazon ECS, network encryption can be implemented in any of the following ways\.
+ **With a service mesh \(TLS\):**

  With AWS App Mesh, you can configure TLS connections between the Envoy proxies that are deployed with mesh endpoints\. Two examples are virtual nodes and virtual gateways\. The TLS certificates can come from AWS Certificate Manager \(ACM\)\. Or, it can come from your own private certificate authority\.
  + [Enabling Transport Layer Security \(TLS\)](https://docs.aws.amazon.com/app-mesh/latest/userguide/tls.html)
  + [Enable traffic encryption between services in AWS App Mesh using ACM certificates or customer provided certs](http://aws.amazon.com/blogs/containers/enable-traffic-encryption-between-services-in-aws-app-mesh-using-aws-certificate-manager-or-customer-provided-certificates/)
  + [TLS ACM walkthrough](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/howto-k8s-tls-acm)
  + [TLS file walkthrough](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/howto-mutual-tls-file-provided)
  + [Envoy](https://www.envoyproxy.io)
+ **Using Nitro instances:**

  By default, traffic is automatically encrypted between the following Nitro instance types: C5n, G4, I3en, M5dn, M5n, P3dn, R5dn, and R5n\. Traffic isn't encrypted when it's routed through a transit gateway, load balancer, or similar intermediary\.
  + [Encryption in transit](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/data-protection.html#encryption-transit)
  + [What's new accouncement from 2019](https://aws.amazon.com/about-aws/whats-new/2019/10/introducing-amazon-ec2-m5n-m5dn-r5n-and-r5dn-instances-featuring-100-gbps-of-network-bandwidth/)
  + [This talk from re:Inforce 2019](https://youtu.be/oqHLLbOoxDg?t=2409)
+ **Using Server Name Indication \(SNI\) with an Application Load Balancer:**

  The Application Load Balancer \(ALB\) and Network Load Balancer \(NLB\) support Server Name Indication \(SNI\)\. By using SNI, you can put multiple secure applications behind a single listener\. For this, each has its own TLS certificate\. We recommend that you provision certificates for the load balanacer using AWS Certificate Manager \(ACM\) and then add them to the listener's certificate list\. The AWS load balancer uses a smart certificate selection algorithm with SNI\. If the hostname that's provided by a client matches a single certificate in the certificate list, the load balancer chooses that certificate\. If a hostname that's provided by a client matches multiple certificates in the list, the load balancer selects a certificate that the client can support\. Examples include self\-signed certificate or a certificate generated through the ACM\.
  + [SNI with Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/create-https-listener.html#https-listener-certificates)
  + [SNI with Network Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/create-tls-listener.html#tls-listener-certificates)
+ **End\-to\-end encryption with TLS certificates:**

  This involves deploying a TLS certificate with the task\. This can either be a self\-signed certificate or a certificate from a trusted certificate authority\. You can obtain the certificate by referencing a secret for the certificate\. Otherwise, you can choose to run an container that issues a Certificate Signing Request \(CSR\) to ACM and then mounts the resulting secret to a shared volume\.
  + [Maintaining transport layer security all the way to your containers using the Network Load Balancer with Amazon ECS part 1](http://aws.amazon.com/blogs/compute/maintaining-transport-layer-security-all-the-way-to-your-container-using-the-network-load-balancer-with-amazon-ecs/)
  + [Maintaining Transport Layer Security \(TLS\) all the way to your container part 2: Using AWS Certificate Manager Private Certificate Authority](http://aws.amazon.com/blogs/compute/maintaining-transport-layer-security-all-the-way-to-your-container-part-2-using-aws-certificate-manager-private-certificate-authority/)

## Task networking<a name="security-network-task-networking"></a>

The following recommendations are in consideration of how Amazon ECS works\. Amazon ECS doesn't use an overlay network\. Instead, tasks are configured to operate in different network modes\. For example, tasks that are configured to use `bridge` mode acquire a non\-routable IP address from a Docker network that runs on each host\. Tasks that are configured to use the `awsvpc` network mode acquire an IP address from the subnet of the host\. Tasks that are configured with `host` networking use the host's network interface\. `awsvpc` is the preferred network mode\. This is because it's the only mode that you can use to assign security groups to tasks\. It's also the only mode that's available for AWS Fargate tasks on Amazon ECS\.

### Security groups for tasks<a name="security-network-task-networking-security-group"></a>

We recommend that you configure your tasks to use the `awsvpc` network mode\. After you configure your task to use this mode, the Amazon ECS agent automatically provisions and attaches an Elastic Network Interface \(ENI\) to the task\. When the ENI is provisioned, the task is enrolled in an AWS security group\. The security group acts as a virtual firewall that you can use to control inbound and outbound traffic\.

## Service mesh and Mutual Transport Layer Security \(mTLS\)<a name="security-network-service-mesh"></a>

You can use a service mesh such as AWS App Mesh to control network traffic\. By default, a virtual node can only communicate with its configured service backends, such as the virtual services that the virtual node will communicate with\. If a virtual node needs to communicate with a service outside the mesh, you can use the `ALLOW_ALL` outbound filter or by creating a virtual node inside the mesh for the external service\. For more information, see [Kubernetes Egress How\-To Walkthrough](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/howto-k8s-egress)\.

App Mesh also gives you the ability to use Mutual Transport Layer Security \(mTLS\) where both the client and the server are mutually authenticated using certificates\. The subsequent communication between client and server are then encrypted using TLS\. By requiring mTLS between services in a mesh, you can verify that the traffic is coming from a trusted source\. For more information, see the following topics: 
+ [mTLS authentication](https://docs.aws.amazon.com/app-mesh/latest/userguide/mutual-tls.html)
+ [mTLS Secret Discovery Service \(SDS\) walkthrough](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/howto-k8s-mtls-sds-based)
+ [mTLS File walkthrough](https://github.com/aws/aws-app-mesh-examples/tree/master/walkthroughs/howto-mutual-tls-file-provided)

## AWS PrivateLink<a name="security-network-privatelink"></a>

AWS PrivateLink is a networking technology that allows you to create private endpoints for different AWS services, including Amazon ECS\. The endpoints are required in sandboxed environments where there is no Internet Gateway \(IGW\) attached to the Amazon VPC and no alternative routes to the Internet\. Using AWS PrivateLink ensures that calls to the Amazon ECS service stay within the Amazon VPC and do not traverse the internet\. For instructions on how to create AWS PrivateLink endpoints for Amazon ECS and other related services, see [Amazon ECS interface Amazon VPC endpoints](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/vpc-endpoints.html)\.

**Important**  
AWS Fargate tasks don't require a AWS PrivateLink endpoint for Amazon ECS\.

Amazon ECR and Amazon ECS both support endpoint policies\. These policies allow you to refine access to a service's APIs\. For example, you could create an endpoint policy for Amazon ECR that only allows images to be pushed to registries in particular AWS accounts\. A policy like this could be used to prevent data from being exfiltrated through container images while still allowing users to push to authorized Amazon ECR registries\. For more information, see [Use VPC endpoint policies](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-access.html#vpc-endpoint-policies)\.

The following policy allows all AWS principals in your account to perform all actions against only your Amazon ECR repositories:

```
{
  "Statement": [
    {
      "Sid": "LimitECRAccess",
      "Principal": "*",
      "Action": "*",
      "Effect": "Allow",
      "Resource": "arn:aws:ecr:region:your_account_id:repository/*"
    },
  ]
}
```

You can enhance this further by setting a condition that uses the new `PrincipalOrgID` property\. This prevents pushing and pulling of images by an IAM principal that isn't part of your AWS Organizations\. For more information, see [aws:PrincipalOrgID](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-principalorgid)\.

We recommended applying the same policy to both the `com.amazonaws.region.ecr.dkr` and the `com.amazonaws.region.ecr.api` endpoints\.

## Amazon ECS container agent settings<a name="security-network-ecs-agent-settings"></a>

The Amazon ECS container agent configuration file includes several environment variables that relate to network security\. `ECS_AWSVPC_BLOCK_IMDS` and `ECS_ENABLE_TASK_IAM_ROLE_NETWORK_HOST` are used to block a task's access to Amazon EC2 metadata\. `HTTP_PROXY` is used to configure the agent to route through a HTTP proxy to connect to the internet\. For instructions on configuring the agent and the Docker runtime to route through a proxy, see [HTTP Proxy Configuration](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/http_proxy_config.html)\.

**Important**  
These settings aren't available when you use AWS Fargate\.

## Recommendations<a name="security-network-recommendations"></a>

We recommend that you do the following when setting up your Amazon VPC, load balancers, and network\.

### Use network encryption where applicable<a name="security-network-recommendations-network-encryption"></a>

You should use network encryption where applicable\. Certain compliance programs, such as PCI DSS, require that you encrypt data in transit if the data contains cardholder data\. If your workload has similar requirements, configure network encryption\.

Modern browsers warn users when connecting to insecure sites\. If your service is fronted by a public facing load balancer, use TLS/SSL to encrypt the traffic from the client's browser to the load balancer and re\-encrypt to the backend if warranted\. 

### Use `awsvpc` network mode and security groups when you need to control traffic between tasks or between tasks and other network resources<a name="security-network-recommendations-awsvpc-networking-mode"></a>

You should use `awsvpc` network mode and security groups when you need to control traffic between tasks and between tasks and other network resources\. If your service behind an ALB, use security groups to only allow inbound traffic from other network resources using the same security group as your ALB\. If your application is behind an NLB, configure the task's security group to only allow inbound traffic from the Amazon VPC CIDR range and the static IP addresses assigned to the NLB\. 

Security groups should also be used to control traffic between tasks and other resources within the Amazon VPC such as Amazon RDS databases\. 

### Create clusters in separate Amazon VPCs when network traffic needs to be strictly isolated<a name="security-network-recommendations-separate-vpcs-for-isolated-traffic"></a>

You should create clusters in separate Amazon VPCs when network traffic needs to be strictly isolated\. Avoid running workloads that have strict security requirements on clusters with workloads that don't have to adhere to those requirements\. When strict network isolation is mandatory, create clusters in separate Amazon VPCs and selectively expose services to other Amazon VPCs using Amazon VPC endpoints\. For more information, see [Amazon VPC endpoints](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints.html)\. 

### Configure AWS PrivateLink endpoints when warranted<a name="security-network-privatelink-endpoints"></a>

You should configure AWS PrivateLink endpoints endpoints when warranted\. If your security policy prevents you from attaching an Internet Gateway \(IGW\) to your Amazon VPCs, configure AWS PrivateLink endpoints for Amazon ECS and other services such as Amazon ECR, AWS Secrets Manager, and Amazon CloudWatch\.

### Use Amazon VPC Flow Logs to analyze the traffic to and from long\-running tasks<a name="security-network-vpc-flow-logs"></a>

You should use Amazon VPC Flow Logs to analyze the traffic to and from long\-running tasks\. Tasks that use `awsvpc` network mode get their own ENI\. Doing this, you can monitor traffic that goes to and from individual tasks using Amazon VPC Flow Logs\. A recent update to Amazon VPC Flow Logs \(v3\), enriches the logs with traffic metadata including the vpc ID, subnet ID, and the instance ID\. This metadata can be used to help narrow an investigation\. For more information, see [Amazon VPC Flow Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html#flow-logs-basics)\.

**Note**  
Because of the temporary nature of containers, flow logs might not always be an effective way to analyze traffic patterns between different containers or containers and other network resources\.