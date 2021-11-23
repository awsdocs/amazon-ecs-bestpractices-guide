# Task deployment<a name="service-options"></a>

To ensure that there's no application downtime, the deployment process is as follows:

1. Start the new application containers while keeping the exiting containers running\.

1. Check that the new containers are healthy\.

1. Stop the old containers\.

 Depending on your deployment configuration and the amount of free, unreserved space in your cluster it may take multiple rounds of this to complete replace all old tasks with new tasks\. 

There are two ECS service configuration options that you can use to modify the number:
+ `minimumHealthyPercent`: 100% \(default\)

  The lower limit on the number of tasks for your service that must remain in the `RUNNING` state during a deployment\. This is represented as a percentage of the desiredCount\. It's rounded up to the nearest integer\. This parameter enables you to deploy without using additional cluster capacity\.
+ `maximumPercent`: 200% \(default\)

   The upper limit on the number of tasks for your service that are allowed in the `RUNNING` or `PENDING` state during a deployment\. This is represented as a percentage of the desiredCount\. It's rounded down to the nearest integer\.

Consider the following service that has six tan tasks, deployed in a cluster that has room for eight tasks total\. The default Amazon ECS service configuration options don't allow the deployment to go below 100% of the six desired tasks\.

![\[Diagram showing six tasks in a cluster that has room for eight tasks.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/deployment-1.PNG)

The deployment process is as follows:

1. The goal is to replace the tan tasks with the blue tasks\.

1. The scheduler starts two new blue tasks because the default settings require that there are six running tasks\.

1. The scheduler stops two of the tan tasks because there will be a total of six tasks \(four tan and two blue\)\.

1. The scheduler starts two additional blue tasks\.

1. The scheduler shuts down two of the tan tasks\.

1. The scheduler starts two additional blue tasks\.

1. The scheduler shuts down the last two orange tasks\.

In the above example, if you use the default values for the options, there is a 2\.5 minute wait for each new task that starts\. Additionally, the load balancer might have to wait 5 minutes for the old task to stop\. 

You can speed up the deployment by setting the `minimumHealthyPercent` value to 50%\.

Consider the following service that has six tan tasks, deployed in a cluster that has room for eight tasks total\. 

![\[Diagram showing six tasks in a cluster that has room for eight tasks with a minimumHealthyPercent value of 50%.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/deployment-2.PNG)

The deployment process is as follows:

1. The goal is to replace the tan tasks with the blue tasks\.

1. The scheduler stops three of the tan tasks\. There are still three tan tasks running which meets the `minimumHealthyPercent` value\.

1. The scheduler starts five blue tasks\.

1. The scheduler stops the remain three tan tasks\.

1. The scheduler starts the final blue tasks\.

You could also add additional free space so that you can run additional tasks\.

![\[Diagram showing six tasks in a cluster that has room for eight tasks.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/deployment-3.PNG)

The deployment process is as follows:

1. The goal is to replace the tan tasks with the blue tasks\.

1. The scheduler stops three of the tan tasks

1. The scheduler starts six blue tasks

1. The scheduler stops the three tan tasks\.

Use the following values for the ECS service configuration options when your tasks are idle for some time and don't have a high utilization rate\.
+ `minimumHealthyPercent`: 50%
+ `maximumPercent`: 200% 