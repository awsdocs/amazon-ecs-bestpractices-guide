# Amazon Elastic Container Service Best Practices Guide

-----
*****Copyright &copy;  Amazon Web Services, Inc. and/or its affiliates. All rights reserved.*****

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
+ [Best Practices - Networking](networking.md)
   + [Connecting to the internet](networking-outbound.md)
   + [Receiving inbound connections from the internet](networking-inbound.md)
   + [Choosing a network mode](networking-networkmode.md)
   + [Connecting to AWS services from inside your VPC](networking-connecting-vpc.md)
   + [Networking between Amazon ECS services in a VPC](networking-connecting-services.md)
   + [Networking services across AWS accounts and VPCs](networking-connecting-services-crossaccount.md)
   + [Optimizing and troubleshooting](networking-troubleshooting.md)
+ [Best Practices - Persistent storage](storage.md)
   + [Amazon EFS volumes](storage-efs.md)
   + [Docker volumes](storage-dockervolumes.md)
   + [Amazon FSx for Windows File Server](storage-fsx.md)
+ [Document history for the Amazon ECS Best Practices Guide](doc-history.md)