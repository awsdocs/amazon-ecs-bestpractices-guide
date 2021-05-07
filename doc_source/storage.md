# Best Practices \- Persistent storage<a name="storage"></a>

You can use Amazon ECS to run stateful containerized applications at scale by using AWS storage services, such as Amazon EFS, Amazon EBS, or Amazon FSx for Windows File Server, that provide data persistence to inherently ephemeral containers\. The term *data persistence* means that the data itself outlasts the process that created it\. Data persistence in AWS is achieved by coupling compute and storage services\. Similar to Amazon EC2, you can also use Amazon ECS to decouple the lifecycle of your containerized applications from the data they consume and produce\. Using AWS storage services, Amazon ECS tasks can persist data even after tasks terminate\.

By default, containers don't persist the data they produce\. When a container is terminated, the data that it wrote to its writable layer gets destroyed with the container\. This makes containers suitable for stateless applications that don't need to store data locally\. Containerized applications that require data persistence need a storage backend that isn't destroyed when the application’s container terminates\.

![\[Diagram showing storage layers of a container.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/storage.jpg)

A container image is built off a series of layers\. Each layer represents an instruction in the Dockerfile that the image was built from\. Each layer is read\-only, except for the container\. That is, when you create a container, a writable layer is added over the underlying layers\. Any files the container creates, deletes, or modifies are written to the writable layer\. When the container terminates, the writable layer is also deleted simultaneously\. A new container that uses the same image has its own writable layer\. This layer doesn't include any changes\. Therefore, a container’s data must always be stored outside of the container writable layer\.

With Amazon ECS, you can run stateful containers using volumes\. Amazon ECS is integrated with Amazon EFS natively, and uses volumes that are integrated with Amazon EBS\. For Windows containers, Amazon ECS integrates with Amazon FSx for Windows File Server to provide persistent storage\.

**Topics**
+ [Choosing the right storage type for your containers](#storage-choosing)
+ [Amazon EFS volumes](storage-efs.md)
+ [Docker volumes](storage-dockervolumes.md)
+ [Amazon FSx for Windows File Server](storage-fsx.md)

## Choosing the right storage type for your containers<a name="storage-choosing"></a>

Applications that are running in an Amazon ECS cluster can use a variety of AWS storage services and third\-party products to provide persistent storage for stateful workloads\. You should choose your storage backend for your containerized application based on the architecture and storage requirements of your application\. For more information about AWS storage services, see [Cloud Storage on AWS](http://aws.amazon.com/products/storage/)\.

For Amazon ECS clusters that contain Linux instances or are used with Fargate, Amazon ECS integrates with Amazon EFS and Amazon EBS to provide container storage\. The most distinctive difference between Amazon EFS and Amazon EBS is that you can simultaneously mount an Amazon EFS filesystem on thousands of Amazon ECS tasks\. In contrast, Amazon EBS volumes don't support concurrent access\. Given this, Amazon EFS is the recommended storage option for containerized applications that scale horizontally\. This is because it supports concurrency\. Amazon EFS stores your data redundantly across multiple Availability Zones and offers low latency access from Amazon ECS tasks, regardless of Availability Zone\. Amazon EFS supports tasks that run on both Amazon EC2 and Fargate\.

Suppose you have an application such as a transactional database that requires sub\-millisecond latency and doesn’t need a shared filesystem when it's scaled horizontally\. For such an application, we recommend using Amazon EBS volumes for persistent storage\. Currently, Amazon ECS supports Amazon EBS volumes for tasks hosted on Amazon EC2 only\. Support for Amazon EBS volumes isn't available for tasks on Fargate\. Before using Amazon EBS volumes with Amazon ECS tasks, you must first attach Amazon EBS volumes to container instances and manage volumes separately from the lifecycle of the task\.

For clusters that contain Windows instances, Amazon FSx for Windows File Server provides persistent storage for containers\. Amazon FSx for Windows File Server filesystems supports multi\-AZ deployments\. Through these deployments, you can share a filesystem with Amazon ECS tasks running across multiple Availability Zones\.

You can also use Amazon EC2 instance storage for data persistence for Amazon ECS tasks that are hosted on Amazon EC2 using bind mounts or Docker volumes\. When using bind mounts or Docker volumes, containers store data on the container instance file system\. One limitation of using a host filesystem for container storage is that the data is only available on a single container instance at a time\. This means that containers can only run on the host where the data resides\. Therefore, using host storage is only recommended in scenarios where data replication is handled at the application level\.