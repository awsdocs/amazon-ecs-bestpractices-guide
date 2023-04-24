# Amazon Elastic Container Service Best Practices Guide

-----
*****Copyright &copy; Amazon Web Services, Inc. and/or its affiliates. All rights reserved.*****

-----
Amazon's trademarks and trade dress may not be used in
connection with any product or service that is not Amazon's,
in any manner that is likely to cause confusion among customers,
or in any manner that disparages or discredits Amazon. All other
trademarks not owned by Amazon are the property of their respective
owners, who may or may not be affiliated with, connected to, or
sponsored by Amazon.

-----
## Contents
+ [Introduction](intro.md)
+ [Best Practices - Running your application with Amazon ECS](application.md)
+ [Best Practices - Networking](networking.md)
   + [Connecting to the internet](networking-outbound.md)
   + [Receiving inbound connections from the internet](networking-inbound.md)
   + [Choosing a network mode](networking-networkmode.md)
      + [Host mode](networking-networkmode-host.md)
      + [Bridge mode](networking-networkmode-bridge.md)
      + [AWSVPC mode](networking-networkmode-awsvpc.md)
   + [Connecting to AWS services from inside your VPC](networking-connecting-vpc.md)
   + [Networking between Amazon ECS services in a VPC](networking-connecting-services.md)
   + [Networking services across AWS accounts and VPCs](networking-connecting-services-crossaccount.md)
   + [Optimizing and troubleshooting](networking-troubleshooting.md)
+ [Best Practices - Auto scaling and capacity management](capacity.md)
   + [Determining task size](capacity-tasksize.md)
   + [Configuring service auto scaling](capacity-autoscaling.md)
   + [Capacity and availability](capacity-availability.md)
   + [Cluster capacity](capacity-cluster.md)
   + [Choosing Fargate task sizes](fargate-task-size.md)
   + [Speeding up cluster capacity provisioning with capacity providers on Amazon EC2](capacity-cluster-speed-up-ec2.md)
   + [Choosing the Amazon EC2 instance type](ec2-instance-type.md)
   + [Using Amazon EC2 Spot and FARGATE_SPOT](ec2-and-fargate-spot.md)
+ [Best Practices - Persistent storage](storage.md)
   + [Amazon EFS volumes](storage-efs.md)
   + [Docker volumes](storage-dockervolumes.md)
   + [FSx for Windows File Server](storage-fsx.md)
+ [Best Practices - Speeding up task launch](task.md)
+ [Best Practices - Speeding up deployments](deployment.md)
   + [Load balancer health check parameters](load-balancer-healthcheck.md)
   + [Load balancer connection draining](load-balancer-connection-draining.md)
   + [Container image type](container-type.md)
   + [Container image pull behavior](pull-behavior.md)
   + [Task deployment](service-options.md)
+ [Best Practices - Operating Amazon ECS at scale](operating-at-scale.md)
   + [Service quotas and API throttling limits](operating-at-scale-service-quotas.md)
   + [Handling throttling issues](operating-at-scale-dealing-with-throttles.md)
+ [Best Practices - Security](security.md)
   + [Shared responsibility model](security-shared.md)
   + [AWS Identity and Access Management](security-iam.md)
   + [Using IAM roles with Amazon ECS tasks](security-iam-roles.md)
   + [Network security](security-network.md)
   + [Secrets management](security-secrets-management.md)
   + [Using temporary security credentials with API operations](temp-credientials.md)
   + [Compliance and security](security-compliance.md)
   + [Logging and monitoring](security-logging-and-monitoring.md)
   + [AWS Fargate security](security-fargate.md)
      + [AWS Fargate security considerations](fargate-security-considerations.md)
   + [Task and container security](security-tasks-containers.md)
   + [Runtime security](security-runtime.md)
   + [AWS Partners](security-partners.md)
+ [Document history for the Amazon ECS Best Practices Guide](doc-history.md)