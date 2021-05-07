# Docker volumes<a name="storage-dockervolumes"></a>

Docker volumes are a feature of the Docker container runtime that allow containers to persist data by mounting a directory from the filesystem of the host\. Docker volume drivers \(also referred to as plugins\) are used to integrate container volumes with external storage systems, such as Amazon EBS\. Docker volumes are only supported when hosting Amazon ECS tasks on Amazon EC2 instances\.

Amazon ECS tasks can use Docker volumes to persist data using Amazon EBS volumes\. This is done by attaching an Amazon EBS volume to an Amazon EC2 instance and then mounting the volume in a task using Docker volumes\. A Docker volume can be shared among multiple Amazon ECS tasks on the host\.

The limitation of Docker volumes is that the file system the task uses is tied to the specific Amazon EC2 instance\. If the instance stops for any reason and the task gets placed on another instance, the data is lost\. You can assign tasks to instances to ensure the associated EBS volumes are always available to tasks\.

For more information, see [Docker volumes](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/docker-volumes.html) in the *Amazon Elastic Container Service Developer Guide*\.

## Amazon EBS volume lifecycle<a name="storage-dockervolumes-ebsvolume"></a>

There are two key usage patterns with container storage and Amazon EBS\. The first is when an application needs to persist data and prevent data loss when its container terminates\. An example of this type of application would be a transactional database like MySQL\. When a MySQL task terminates, another task is expected to replace it\. In this scenario, the lifecycle of the volume is separate from the lifecycle of the task\. When using EBS to persist container data, it's a best practice to use task placement constraints to limit the placement of the task to a single host with the EBS volume attached\.

The second is when the lifecycle of the volume is independent from the task lifecycle\. This is especially useful for applications that require high\-performing and low latency storage but don’t need to persist data after the task terminates\. For example, an ETL workload that process large volumes of data may require a high throughput storage\. Amazon EBS is suitable for this type of workload as it provides high performance volumes that provideup to 256,000 IOPS\. When the task terminates, the replacement replica can be safely placed on any Amazon EC2 hosts in the cluster\. As long as the task has access to a storage backend that can meet its performance requirements, the task can perform its function\. Therefore, no task placement constraints are necessary in this case\.

If the Amazon EC2 instances in your cluster have multiple types of Amazon EBS volumes attached to them, you can use task placement constraints to ensure that tasks are placed on instances with an appropriate Amazon EBS volume attached\. For example, suppose a cluster has some instances with a `gp2` volume, while others use `io1` volumes\. You can attach custom attributes to instances with `io1` volumes and then use task placement constraints to ensure your I/O intensive tasks are always placed on container instances with `io1` volumes\.

The following AWS CLI command is used to place attributes on an Amazon ECS container instance\.

```
aws ecs put-attributes \ 
     --attributes name=EBS,value=io1,targetId=<your-container-instance-arn>
```

## Amazon EBS data availability<a name="storage-dockervolumes-dataavailability"></a>

Containers are typically short lived, frequently created, and terminated as applications scale in and out horizontally\. As a best practice, you can run workloads in multiple Availability Zones to improve the availability of your applications\. Amazon ECS provides a way for you to control task placement using task placement strategies and task placement constraints\. When a workload persists its data using Amazon EBS volumes, its tasks need to be placed in the same Availability Zone as the Amazon EBS volume\. We also recommend that you set a placement constraint that limits the Availability Zone a task can be placed in\. This ensures that your tasks and their corresponding volumes are always located in the same Availability Zone\.

When running standalone tasks, you can control which Availability Zone the task gets placed by setting placement constraints using the availability zone attribute\.

```
attribute:ecs.availability-zone == us-east-1a
```

When running applications that would benefit from running in multiple Availability Zones, consider creating a different Amazon ECS service for each Availability Zone\. This ensures that tasks that need an Amazon EBS volume are always placed in the same Availability Zone as the associated volume\.

We recommend creating container instances in each Availability Zone, attaching Amazon EBS volumes using [launch templates](https://docs.aws.amazon.com/autoscaling/ec2/userguide/create-launch-template.html), and adding [custom attributes](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-placement-constraints.html#add-attribute) to the instances to differentiate them from other container instances in the Amazon ECS cluster\. When creating services, configure task placement constraints to ensure that Amazon ECS places tasks in the right Availability Zone and instance\. For more information, see [Task placement constraint examples](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-placement-constraints.html#constraint-examples) in the *Amazon Elastic Container Service Developer Guide*\.

## Docker volume plugins<a name="storage-dockervolumes-plugins"></a>

Docker plugins such as Portworx provide an abstraction between the Docker volume and the Amazon EBS volume\. These plugins can dynamically create an Amazon EBS volume when your task that needs a volume starts\. Portworx can also attach a volume to a new host when a container terminates, and its subsequent replica gets placed on a different container instance\. It also replicates each container’s volume data among Amazon ECS nodes and across Availability Zones\. For more information, see [Portworx](https://portworx.com/)\.