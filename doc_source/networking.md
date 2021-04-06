# Best Practices \- Networking<a name="networking"></a>

Modern applications are typically built out of multiple components that communicate with each other as part of a distributed system\. For example, you may have a mobile or web application that communicates with an API endpoint\. That API may be powered by multiple microservices that communicate with each other\. Some of those microservices may require internet access to talk to other supporting APIs on the Internet, or they need to talk to the APIs of AWS services like Amazon DynamoDB or Amazon SQS\.

In this guide, we discuss best practices for building the networking that makes it so the distributed components of your application can communicate with each other securely, and in a scalable manner\.

**Topics**
+ [Connecting to the Internet](networking-outbound.md)
+ [Receiving inbound connections from the Internet](networking-inbound.md)
+ [Choosing a network mode](networking-networkmode.md)
+ [Connecting to AWS services from inside your VPC](networking-connecting-vpc.md)
+ [Networking between Amazon ECS services in a VPC](networking-connecting-services.md)
+ [Networking services across AWS accounts and VPCs](networking-connecting-services-crossaccount.md)
+ [Optimizing and troubleshooting](networking-troubleshooting.md)