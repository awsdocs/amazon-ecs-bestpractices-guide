# Determining task size<a name="capacity-tasksize"></a>

One of the most important choices to make when deploying containers on Amazon ECS is your container and task sizes\. Your container and task sizes are both essential for scaling and capacity planning\. In Amazon ECS, there are two resource metrics used for capacity: CPU and memory\. CPU is measured in units of 1/1024 of a full vCPU \(where 1024 units is equal to 1 whole vCPU\)\. Memory is measured in megabytes\. In your task definition, you can declare resource reservations and limits\.

When you declare a reservation, you're declaring the minimum amount of resources that a task requires\. Your task receives at least the amount of resources requested\. Your application might be able to use more CPU or memory than the reservation that you declare\. However, this is subject to any limits that you also declared\. Using more than the reservation amount is known as bursting\. In Amazon ECS, reservations are guaranteed\. For example, if you use Amazon EC2 instances to provide capacity, Amazon ECS doesn't place a task on an instance where the reservation can't be fulfilled\.

A limit is the maximum amount of CPU units or memory that your container or task can use\. Any attempt to use more CPU more than this imit results in throttling\. Any attempt to use more memory results in your container being stopped\.

Choosing these values can be challenging\. This is because the values that are the most well suited for your application greatly depend on the resource requirements of your application\. Load testing your application is the key to successful resource requirement planning and better understanding your application's requirements\.

## Stateless applications<a name="capacity-tasksize-stateless"></a>

For stateless applications that scale horizontally, such as an application behind a load balancer, we recommend that you first determine the amount of memory that your application consumes when it serves requests\. To do this, you can use traditional tools such as `ps` or `top`, or monitoring solutions such as CloudWatch Container Insights\.

When determining a CPU reservation, consider how you want to scale your application to meet your business requirements\. You can use smaller CPU reservations, such as 256 CPU units \(or 1/4 vCPU\), to scale out in a fine\-grained way that minimizes cost\. But, they might not scale fast enough to meet significant spikes in demand\. You can use larger CPU reservations to scale in and out more quickly and therefore match demand spikes more quickly\. However, larger CPU reservations are more costly\.

## Other applications<a name="capacity-tasksize-other"></a>

For applications that don't scale horizontally, such as singleton workers or database servers, available capacity and cost represent your most important considerations\. You should choose the amount of memory and CPU based on what load testing indicates you need to serve traffic to meet your service\-level objective\. Amazon ECS ensures that the application is placed on a host that has adequate capacity\.