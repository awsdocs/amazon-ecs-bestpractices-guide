# Determining task size<a name="capacity-tasksize"></a>

One of the most important choices you will make when deploying containers on Amazon ECS is your container and task sizes\. These values are essential for scaling and capacity planning\. In Amazon ECS, there are two resource metrics used for capacity: CPU and memory\. CPU is measured in units of 1/1024 of a full vCPU \(where 1024 units is equal to 1 whole vCPU\), and memory is measured in megabytes\. In your task definition, you can declare resource reservations and limits\.

When declaring a reservation, you are declaring the minimum amount of resources a task requires\. Your task will receive at least the amount of resources requested\. Your application may be able to use more CPU or memory than the reservation, subject to any limits you may have declared\. Using more than the reservation amount is known as bursting\. In Amazon ECS, reservations are guaranteed\. For example, if you use Amazon EC2 instances to provide capacity, Amazon ECS will not place a task on an instance where the reservation cannot be fulfilled\.

A limit is the maximum amount of CPU units or memory that your container or task can use\. Any attempt to use more CPU will result in throttling\. Any attempt to use more memory will result in your container being stopped\.

Choosing these values can be challenging and will depend on the nature of your application\. Load testing your application is key to successful resource requirement planning\.

## Stateless applications<a name="capacity-tasksize-stateless"></a>

For stateless applications that scale horizontally, for example an application behind a load balancer, we recommend you first determine the amount of memory that your application consumes while serving requests\. To do this, you can use traditional tools such as `ps` or `top`, or monitoring solutions such as CloudWatch Container Insights\.

When determining a CPU reservation, consider how you want to scale your application to meet your business requirements\. Smaller CPU reservations, such as 256 CPU units \(or 1/4 vCPU\), allow you to scale out in a fine\-grained way that minimizes cost, but could scale out too slowly to meet significant spikes in demand\. Larger CPU reservations allow you to scale in and out more quickly, and can help match demand spikes more quickly, but can be more costly\.

## Other applications<a name="capacity-tasksize-other"></a>

For applications that do not scale horizontally, such as singleton workers or database servers, available capacity and cost are often the most important considerations\. You should choose the amount of memory and CPU that load testing indicates you need to serve traffic to meet your service\-level objective\. Amazon ECS will ensure that the application is placed on a host that has adequate capacity\.