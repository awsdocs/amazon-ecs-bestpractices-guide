# Service quotas and API throttling limits<a name="operating-at-scale-service-quotas"></a>

Amazon ECS is integrated with several AWS services, including Elastic Load Balancing, AWS Cloud Map, and Amazon EC2\. With this tight integration, Amazon ECS includes several features such as service load balancing, service discovery, task networking, and cluster auto scaling\. Amazon ECS and the other AWS services that it integrates with all maintain service quotas and API rate limits to ensure consistent performance and utilization\. These service quotas also prevent the accidental provisioning of more resources than needed and protect against malicious actions that might increase your bill\.

By familiarizing yourself with your service quotas and the AWS API rate limits, you can plan for scaling your workloads without worrying about unexpected performance degradation\. For more information, see [Amazon ECS service quotas](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-quotas.html) and [Request throttling for the Amazon ECS API](https://docs.aws.amazon.com/AmazonECS/latest/APIReference/request-throttling.html)\.

When scaling your workloads on Amazon ECS, we recommend that you consider the following service quota\. For instructions on how to request a service quota increase, see [Managing your Amazon ECS and AWS Fargate service quotas in the AWS Management Console](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-quotas.html#service-quotas-manage)\.
+ AWS Fargate has quotas that limit the number of concurrent running tasks in each AWS Region\. There are quotas for both On\-Demand and Fargate Spot tasks on Amazon ECS\. Each service quota also includes any Amazon EKS pods that you run on Fargate\. For more information about the Fargate quotas, see [AWS Fargate service quotas](https://docs.aws.amazon.com/AmazonECS/latest/userguide/service-quotas.html#service-quotas-fargate) in the *Amazon Elastic Container Service User Guide for AWS Fargate*\.
+ For tasks that run on Amazon EC2 instances, the maximum number of Amazon EC2 instances that you can register for each cluster is 5,000\. If you use Amazon ECS cluster auto scaling with an Auto Scaling group capacity provider, or if you manage Amazon EC2 instances for your cluster on your own, this quota might become a deployment bottleneck\. If you require more capacity, you can create more clusters or request a service quota increase\.
+ If you use Amazon ECS cluster auto scaling with an Auto Scaling group capacity provider, when scaling your services consider the `Tasks in the PROVISIONING state per cluster` quota\. This quota is the maximum number of tasks in the `PROVISIONING` state for each cluster for which capacity providers can increase capacity\. When you launch a large number of tasks all at the same time, you can easily meet this quota\. One example is if you simultaneously deploy tens of services, each with hundreds of tasks\. When this happens, the capacity provider needs to launch new container instances to place the tasks when the cluster has insufficient capacity\. While the capacity provider is launching additional Amazon EC2 instances, the Amazon ECS service scheduler likely will continue to launch tasks in parallel\. However, this activity might be throttled because of insufficient cluster capacity\. The Amazon ECS service scheduler implements a back\-off and exponential throttling strategy for retrying task placement as new container instances are launched\. As a result, you might experience slower deployment or scale\-out times\. To avoid this situation, you can plan your service deployments in one of the following ones\. Either deploy a large number of tasks don't require increasing cluster capacity, or keep spare cluster capacity for new task launches\.

In addition to considering Amazon ECS service quota when scaling your workloads, consider also the service quota for the other AWS services that are integrated with Amazon ECS\. The following section covers the key rate limits for each service in detail, and provides recommendations to deal with potential throttling issues\.

## Elastic Load Balancing<a name="operating-at-scale-service-quotas-elb"></a>

You can configure your Amazon ECS services to use Elastic Load Balancing to distribute traffic evenly across the tasks\. For more information and recommended best practices for how to choose a load balancer, see [Service load balancing considerations](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-load-balancing.html#load-balancing-considerations) and [Load balancer health check parameters](https://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/load-balancer-healthcheck.html)\.

### Elastic Load Balancing service quotas<a name="elb-service-quotas"></a>

When you scale your workloads, consider the following Elastic Load Balancing service quotas\. Most Elastic Load Balancing service quotas are adjustable, and you can request an increase in the Service Quotas console\.

**Application Load Balancer**

When you use an Application Load Balancer, depending on your use case, you might need to request a quota increase for:
+  The `Targets per Application Load Balancer` quota which is the number of targets behind your Application Load Balancer\.
+ The `Targets per Target Group per Region` quota which is the number of targets behind your Target Groups\.

For more information, see [Quotas for your Application Load Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-limits.html)\.

**Network Load Balancer**

There are stricter limitations on the number of targets you can register with a Network Load Balancer\. When using a Network Load Balancer, you often will want to enable cross\-zone support, which comes with additional scaling limitations on `Targets per Availability Zone Per Network Load Balancer` the maximum number of targets per Availability Zone for each Network Load Balancer\. For more information, see [Quotas for your Network Load Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-limits.html)\.

### Elastic Load Balancing API throttling<a name="elb-service-api-throttling"></a>

When you configure an Amazon ECS service to use a load balancer, the target group health checks must pass before the service is considered healthy\. For performing these health checks, Amazon ECS invokes Elastic Load Balancing API operations on your behalf\. If you have a large number of services configured with load balancers in your account, you might slower service deployments because of potential throttling specifically for the `RegisterTarget`, `DeregisterTarget`, and `DescribeTargetHealth` Elastic Load Balancing API operations\. When throttling occurs, throttling errors occur in your Amazon ECS service event messages\.

If you experience AWS Cloud Map API throttling, you can contact AWS Support for guidance on how to increase your AWS Cloud Map API throttling limits\. For more information about monitoring and troubleshooting such throttling errors, see [Handling throttling issues](https://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/operating-at-scale-dealing-with-throttles.html)\. 

## Elastic network interfaces<a name="elastic-network-interfaces"></a>

With your tasks use the `awsvpc` network mode, Amazon ECS provisions a unique elastic network interface \(ENI\) for each task\. When your Amazon ECS services use an Elastic Load Balancing load balancer, these network interfaces are also registered as targets to the appropriate target group defined in the service\.

### Elastic network interface service quotas<a name="eni-service-quotas"></a>

When you run tasks that use the `awsvpc` network mode, a unique elastic network interface is attached to each task\. If those tasks must be reached over the internet, assign a public IP address to the elastic network interface for those tasks\. When you scale your Amazon ECS workloads, consider these two important quotas:
+ The `Network interfaces per Region` quota which is the maximum number of network interfaces in an AWS Region for your account\.
+ The `Elastic IP addresses per Region` quota which is the maximum number of elastic IP addresses in an AWS Region\.

Both of these service quotas are adjustable and you can request an increase from your Service Quotas console for these\. For more information, see [Amazon VPC service quotas](https://docs.aws.amazon.com/vpc/latest/userguide/amazon-vpc-limits.html#vpc-limits-enis)\.

For Amazon ECS workloads that are hosted on Amazon EC2 instances, when running tasks that use the `awsvpc` network mode consider the `Maximum network interfaces` service quota, the maximum number of network instances for each Amazon EC2 instance\. This quota limits the number of tasks that you can place on an instance\. You cannot adjust the quota and it's not available in the Service Quotas console\. For more information, see [IP addresses per network interface per instance type](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html#AvailableIpPerENI) in the *Amazon EC2 User Guide*\.

Even though you can't change the number of network interfaces that's for an Amazon EC2 instance isn't adjustable, you can use the elastic network interface trunking feature to increase the number of available network interfaces\. For example, by default a `c5.large` instance can have up to three network interfaces\. The primary network interface for the instance counts as one\. So, you can attach an additional two network interfaces to the instance\. Because each task that uses the `awsvpc` network mode requires a network interface, you can typically only run two such tasks on this instance type\. This can lead to under\-utilization of your cluster capacity\. If you enable elastic network interface trunking, you can increase the network interface density to place a larger number of tasks on each instance\. With trunking turned on, a `c5.large` instance can have up to 12 network interfaces\. The instance has the primary network interface and Amazon ECS creates and attaches a "trunk" network interface to the instance\. As a result, with this configuration you can run 10 tasks on the instance instead of the default two tasks\. For more information, see [Elastic network interface trunking](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/container-instance-eni.html)\.

### Elastic network interface API throttling<a name="eni-api-throttles"></a>

When you run tasks that use the `awsvpc` network mode, Amazon ECS relies on the following Amazon EC2 APIs\. Each of these APIs have different API throttles\. For more information, see [Request throttling for the Amazon EC2 API](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/throttling.html)\.
+ CreateNetworkInterface
+ AttachNetworkInterface
+ DetachNetworkInterface
+ DeleteNetworkInterface
+ DescribeNetworkInterfaces
+ DescribeVpcs
+ DescribeSubnets
+ DescribeSecurityGroups
+ DescribeInstances

If the Amazon EC2 API calls are throttled during the elastic network interface provisioning workflows, the Amazon ECS service scheduler automatically retries with exponential back\-offs\. These automatic retires might sometimes lead to a delay in launching tasks, which results in slower deployment speeds\. When API throttling occurs, you will see the message `Operations are being throttled. Will try again later.` on your service event messages\. If you consistently meet Amazon EC2 API throttles, you can contact AWS Support for guidance on how to increase your API throttling limits\. For more information about monitoring and troubleshooting throttling errors, see [Handling throttling issues](https://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/operating-at-scale-dealing-with-throttles.html)\.

## AWS Cloud Map<a name="cloudmap"></a>

Amazon ECS service discovery uses AWS Cloud Map APIs to manage namespaces for your Amazon ECS services\. If your services have a large number of tasks, consider the following recommendations\. For more information, see [Amazon ECS service discovery considerations](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-discovery.html#service-discovery-considerationsl)\.

### AWS Cloud Map service quotas<a name="cloudmap-service-quotas"></a>

When Amazon ECS services are configured to use service discovery, the `Tasks per service` quota which is the maximum number of tasks for the service, is affected by the AWS Cloud Map `Instances per service` service quota which is the maximum number of instances for that service\. In particular, the AWS Cloud Map service quota decreases the amount of tasks that you can run to at most 1,0000 tasks for service\. You cannot change the AWS Cloud Map quota\. For more information, see [AWS Cloud Map service quotas](https://docs.aws.amazon.com/general/latest/gr/cloud_map.html)\.

### AWS Cloud Map API throttling<a name="cmap-api-throttles"></a>

Amazon ECS calls the `ListInstances`, `GetInstancesHealthStatus`, `RegisterInstance`, and `DeregisterInstance` AWS Cloud Map APIs on your behalf\. They help with service discovery and perform health checks when you launch a task\. When multiple services that use service discovery with a large number of tasks are deployed at the same time, this can result in exceeding the AWS Cloud Map API throttling limits\. When this happens, you likely will see the following message: `Operations are being throttled. Will try again later` in your Amazon ECS service event messages and slower deployment and task launch speed\. AWS Cloud Map doesn't document throttling limits for these APIs\. If you experience throttling from these, you can contact AWS Support for guidance on increasing your API throttling limits\. For more recommendations on monitoring and troubleshooting such throttling errors, see [Handling throttling issues](https://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/operating-at-scale-dealing-with-throttles.html)\.