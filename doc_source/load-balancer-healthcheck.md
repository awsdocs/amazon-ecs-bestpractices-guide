# Load balancer health check parameters<a name="load-balancer-healthcheck"></a>

The following diagram describes the load balancer health check process\. The load balancer periodically sends health checks to the Amazon ECS container\. The Amazon ECS agent monitors and waits for the load balancer to report on the container health\. It does this before it considers the container to be in a healthy status\.

![\[Diagram showing the load balancer health checks.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/load-balancer-healthcheck.PNG)

There are two options in the health check that affect the deployment speed\. One is for the Amazon ECS task definition\. The other is for the load balancer\.
+ `HealthCheckInterval`: 30 seconds \(default\)

  For more information about the task definition health check interval parameter, see [Health Check](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html#container_definition_healthcheck) in the *Amazon Elastic Container Service Developer Guide*\.
+ `HealthyThresholdCount`: 5 \(default\)

By default, the load balancer requires five passing health checks before it reports that the target container is healthy\. Each check is made 30 seconds apart\. In total, each time takes two minutes and 30 seconds \(`5*30/60`\)\. 

If your service starts up and stabilizes in under 10 seconds, set the options to the following values to have the load balancer wait a total of 10 seconds:
+ `HealthCheckInterval`: 5 
+ `HealthyThresholdCount`: 2