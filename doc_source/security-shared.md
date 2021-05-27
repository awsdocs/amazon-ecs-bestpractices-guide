# Shared responsibility model<a name="security-shared"></a>

The security and compliance of a managed service like Amazon ECS is a shared resposibility of both you and AWS\. Generally speaking, AWS is responsible for security "of" the cloud whereas you, the customer, are responsible for security "in" the cloud\. AWS is responsible for managing of the Amazon ECS control plane, including the infrastructure that's needed to deliver a secure and reliable service\. And, you're largely responsible for the topics in this guide\. This includes data, network, and runtime security, as well as logging and monitoring\.

![\[Diagram showing security layers for Amazon ECS on Amazon EC2.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/amazon-ecs-on-ec2-responsibilities.png)

With respect to infrastructure security, AWS assumes more responsibility for AWS Fargate resources than it does for other self\-managed instances\. With Fargate, AWS manages the security of the underlying instance in the cloud and the runtime that's used to run your tasks\. Fargate also automatically scales your infrastructure on your behalf\.

![\[Diagram showing security layers for Amazon ECS on Fargate.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/amazon-ecs-on-fargate-responsibilities.png)

Before extending your services to the cloud, you should understand what aspects of security and compliance that you're responsible for\.

For more information about the shared responsibility model, see [Shared Responsibility Model](http://aws.amazon.com/compliance/shared-responsibility-model)\.