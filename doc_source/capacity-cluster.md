# Cluster capacity<a name="capacity-cluster"></a>

We have discussed how to scale out the replica account for your by using scaling metrics\. Your tasks also need resources on which to run, including CPU and memory, which we refer to as capacity\. In Amazon ECS, capacity is provided via two primary providers: AWS Fargate and Amazon EC2\.

You can provide capacity to an Amazon ECS cluster in several ways\. The historical way is to launch Amazon EC2 instances and register them with the cluster at startup using the Amazon ECS container agent\. However, this method is challenging to work with, since you must manage scaling yourself\. Today, we recommend using Amazon ECS capacity providers because they are powerful and flexible, and manage resource scaling for you\. There are three kinds of capacity providers: Amazon EC2, Fargate, and Fargate Spot\.

The Fargate and Fargate Spot capacity providers handle the lifecycle of Fargate tasks for you, using on\-demand and Spot capacity respectively\. When a task is launched, ECS will provision a Fargate instance with the memory and CPU units that correspond to the task\-level limits in your task definition\. Each task will get its own instance, so there is a 1:1 relationship between a task and compute in Fargate\. 

Tasks running on Fargate Spot are subject to interruption, with a 2\-minute warning, during periods of heavy demand\. Fargate Spot works best for interruption\-tolerant workloads such as batch jobs, development or staging environments, or other situations where high availability and low latency is not required\.

You can run Fargate Spot tasks in addition to Fargate on\-demand tasks\. This allows you to receive provision “burst” capacity at low cost\.

ECS can also manage Amazon EC2 instance capacity for your tasks\. Each Amazon EC2 Capacity Provider is associated with an Amazon EC2 Auto Scaling Group that you specify\. If enabled, ECS cluster auto scaling will maintain the size of the Amazon EC2 Auto Scaling Group to ensure all scheduled tasks can be placed\.

## Cluster capacity best practices<a name="capacity-cluster-best-practices"></a>

**Add headroom to your service, not to the Capacity Provider\.** Amazon EC2 Capacity Providers offer a target capacity value\. If you set the value lower than 100%, ECS provisions more Amazon EC2 instances than necessary for accommodating your tasks\. Although having more Amazon EC2 instances ready to accept tasks can be beneficial, when you use Amazon Virtual Private Cloud, launching new tasks will require additional time to download the image, and attach a network interface\. This added latency might be critical\.

Instead of reducing the Capacity Provider’s target capacity, you can increase the number of replicas in your service by modifying the service's target tracking scaling metric or the step scaling thresholds\. For more information, see [Target Tracking Scaling Policies](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-autoscaling-targettracking.html) or [Step Scaling Policies](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-autoscaling-stepscaling.html) in the *Amazon Elastic Container Service Developer Guide*\. The Amazon EC2 Capacity Provider provisions the capacity for additional tasks by adding additional instances to the Auto Scaling Group\. This helps to ensure that both the compute and application resources are available when you need them, for example, doubling the number of tasks in an ECS Service will accommodate an immediate 100% burst in demand\. 