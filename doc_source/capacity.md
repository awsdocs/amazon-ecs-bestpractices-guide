# Best Practices \- Auto scaling and capacity management<a name="capacity"></a>

Amazon ECS is used to run containerized application workloads of all sizes\. This includes both the extremes of minimal testing environments and large production environments operating at a global scale\.

With Amazon ECS, like all AWS services, you pay only for what you use\. When architected appropriately, you can save costs by having your application consume only the resources that it needs at the time that it needs them\. This best practices guide shows how to run your Amazon ECS workloads in a way that meets your service\-level objectives while still operating in a cost\-effective manner\.

**Topics**
+ [Determining task size](capacity-tasksize.md)
+ [Configuring service auto scaling](capacity-autoscaling.md)
+ [Capacity and availability](capacity-availability.md)
+ [Cluster capacity](capacity-cluster.md)
+ [Choosing Fargate task sizes](fargate-task-size.md)
+ [Speeding up cluster capacity provisioning with capacity providers on Amazon EC2](capacity-cluster-speed-up-ec2.md)
+ [Choosing the Amazon EC2 instance type](ec2-instance-type.md)
+ [Using Amazon EC2 Spot and FARGATE\_SPOT](ec2-and-fargate-spot.md)