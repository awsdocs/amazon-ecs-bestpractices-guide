# Best Practices \- Speeding up deployments<a name="deployment"></a>

You can choose rolling updates for your Amazon ECS service\. Deployments might take longer than you expect, but you can modify a few options to speed up your deployments\. For context, by choosing this deployment type, you tell the Amazon ECS service scheduler to replace any currently running tasks with new tasks whenever a new service deployment is started\. The deployment configuration determines the specific number of tasks that Amazon ECS adds to or removes from the service during a rolling update\. The following is an overview of the deployment process:

1. The scheduler starts your application\.

1. The scheduler then decides if your application is ready for web traffic\.

1. When you scale down or create a new version of the application, the scheduler decides whether your application is safe to stop\. At the same time, it must maintain the availability of the application during the rolling deployment\.

The strategy of maintaining task availability can cause deployments to take longer than you expect\.

To speed up deployment times, modify the default load balancer, Amazon ECS agent, service and task definition options\. The following sections detail how you can modify all of these to speed up deployments\.

**Topics**
+ [Load balancer health check parameters](load-balancer-healthcheck.md)
+ [Load balancer connection draining](load-balancer-connection-draining.md)
+ [Container image type](container-type.md)
+ [Container image pull behavior](pull-behavior.md)
+ [Task deployment](service-options.md)