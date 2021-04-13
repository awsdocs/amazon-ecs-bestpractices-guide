# Best Practices \- Networking<a name="networking"></a>

Modern applications are typically built out of multiple distributed components that communicate with each other\. For example, a mobile or web application might communicate with an API endpoint, and the API might be powered by multiple microservices that communicate over the internet\. 

 This guide presents the best practices for building a network where the components of your application can communicate with each other securely and in a scalable manner\.

**Topics**
+ [Connecting to the internet](networking-outbound.md)
+ [Receiving inbound connections from the internet](networking-inbound.md)
+ [Choosing a network mode](networking-networkmode.md)
+ [Connecting to AWS services from inside your VPC](networking-connecting-vpc.md)
+ [Networking between Amazon ECS services in a VPC](networking-connecting-services.md)
+ [Networking services across AWS accounts and VPCs](networking-connecting-services-crossaccount.md)
+ [Optimizing and troubleshooting](networking-troubleshooting.md)