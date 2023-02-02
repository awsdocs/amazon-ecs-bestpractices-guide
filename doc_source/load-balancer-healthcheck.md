# Load balancer health check parameters<a name="load-balancer-healthcheck"></a>

The following diagram describes the load balancer health check process\. The load balancer periodically sends health checks to the Amazon ECS container\. The Amazon ECS agent monitors and waits for the load balancer to report on the container health\. It does this before it considers the container to be in a healthy status\.

![\[Diagram showing the load balancer health checks.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/load-balancer-healthcheck.png)

Two Elastic Load Balancing health check parameters affect deployment speed:
+ Health check interval: Determines the approximate amount of time, in seconds, between health checks of an individual container\. By default, the load balancer checks every 30 seconds\.

  This parameter is named:
  + `HealthCheckIntervalSeconds` in the Elastic Load Balancing API
  + **Interval** on the Amazon EC2 console
+ Healthy threshold count: Determines the number of consecutive health check successes required before considering an unhealthy container healthy\. By default, the load balancer requires five passing health checks before it reports that the target container is healthy\.

  This parameter is named:
  + `HealthyThresholdCount` in the Elastic Load Balancing API
  + **Healthy threshold** on the Amazon EC2 console

With the default setting, the total time to determine the health of a container is two minutes and 30 seconds \(`30 seconds * 5 = 150 seconds`\)\.

You can speed up the health\-check process if your service starts up and stabilizes in under 10 seconds\. To speed up the process, reduce the number of checks and the interval between the checks\.
+ `HealthCheckIntervalSeconds` \(Elastic Load Balancing API name\) or **Interval** \(Amazon EC2 console name\): 5
+ `HealthyThresholdCount` \(Elastic Load Balancing API name\) or **Healthy threshold** \(Amazon EC2 console name\): 2

With this setting, the health\-check process takes 10 seconds compared to the default of two minutes and 30 seconds\.

**Setting the Elastic Load Balancing health check parameters to speed up deployment**

1. Go to the [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. From the left navigation, under **Load Balancing**, select **Target Groups**\.

1. On the **Target groups** page, select the target group\.

1. On the target group page, select the **Health checks** tab\.

1. Click **Edit**\.

1. Expand **Advanced health check settings**\.

1. Set the health check parameters\.

1. Click **Save changes**\.

For more information about the Elastic Load Balancing health check parameters, see [TargetGroup](https://docs.aws.amazon.com/elasticloadbalancing/latest/APIReference/API_TargetGroup.html) in the *Elastic Load Balancing API Reference*\.