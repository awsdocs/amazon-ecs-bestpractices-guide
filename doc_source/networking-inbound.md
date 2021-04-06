# Receiving inbound connections from the Internet<a name="networking-inbound"></a>

When running a public\-facing service, it's likely you need to accept inbound traffic from the Internet as well\. For example, you might have a public website that needs to accept inbound HTTP requests from browsers, or a public API that powers a web or mobile phone application\. In this scenario, other hosts on the internet must be able to initiate an inbound connection to your application's host instance\.

A simple method is to launch your containers on hosts that are in a public subnet with a public IP address\. However, this approach is rarely used at scale\. It is generally preferred to have a scalable ingress layer that sits between the Internet and your application instead of having arbitrary hosts on the Internet communicate directly with your application\. There are a few different AWS services you can use as an ingress\.

**Topics**
+ [Application Load Balancer](#networking-alb)
+ [Network Load Balancer](#networking-nlb)
+ [Amazon API Gateway HTTP API](#networking-apigateway)

## Application Load Balancer<a name="networking-alb"></a>

An Application Load Balancer functions at the application layer, the seventh layer of the Open Systems Interconnection \(OSI\) model which makes them ideal for web\-facing HTTP services\. For example, if you have a website or an HTTP REST API then an Application Load Balancer is a great load balancer for this workload type\. For more information, see [What is an Application Load Balancer?](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) in the *User Guide for Application Load Balancers*\.

![\[Diagram showing architecture of a network using an Application Load Balancer.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/alb-ingress.png)

With this architecture, you create an Application Load Balancer in a public subnet so that it has a public IP address and can receive inbound connections from the Internet\. When the Application Load Balancer receives an inbound connection and sees an HTTP request, it will open a connection to the application using its private IP address and forwards the request over the internal connection\.

Application Load Balancers have the following useful features\.
+ SSL/TLS termination — Application Load Balancers can handle secure HTTPS communication and certificates for communicating with clients, and can optionally terminate the SSL connection at the load balancer level so that you don’t have to handle certificates in your own application\.
+ Advanced routing — Application Load Balancers can have multiple DNS hostnames, and have powerful routing capabilities to send incoming HTTP requests to different destinations based on characteristics like the hostname, or even the path of the request\. This means you can use a single Application Load Balancer as the front facing ingress for many different internal services, or even microservices on different paths of a REST API\.
+ gRPC support and websockets — Application Load Balancers handle more than just HTTP\. They can also load balance gRPC and websocket based services, with HTTP/2 support\.
+ Security — Application Load Balancers help protect your application from bad traffic\. They include features such as HTTP desync mitigations, and it integrates with AWS Web Application Firewall \(AWS WAF\) which can further filter out bad traffic that includes common attack patterns like SQL injection or cross\-site scripting\.

## Network Load Balancer<a name="networking-nlb"></a>

A Network Load Balancer functions at the fourth layer of the Open Systems Interconnection \(OSI\) model\. It is ideal for non\-HTTP protocols or situations where end to end encryption is necessary\. It doesn’t have the deep HTTP specific features that an Application Load Balancer does, but for applications that don’t use HTTP, a Network Load Balancer is better\. For more information, see [What is a Network Load Balancer?](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html) in the *User Guide for Network Load Balancers*\.

![\[Diagram showing architecture of a network using an Network Load Balancer.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/nlbingress.png)

A Network Load Balancer as an ingress is very similar to an Application Load Balancer in that it is created in a public subnet, so that it can have a public IP address that is reachable from the Internet\. The Network Load Balancer then opens a connection to the private IP address of the host running your container, and sends the packets from the public side to the private side\.

Because the Network Load Balancer operates at a lower level of the networking stack it can’t do many of the things that the higher level Application Load Balancer does, however it does have the following important features\.
+ End\-to\-end encryption — Because a Network Load Balancer operates at the fourth layer of the OSI model, it doesn't need to inspect the contents of packets, like an Application Load Balancer does\. This makes it ideal for load balancing communications that need to stay end to end encrypted all the way to your application\.
+ TLS encryption — Although you can use a Network Load Balancer for full end to end encryption you can also choose to have the Network Load Balancer terminate TLS connections so that your backend applications don’t have to implement their own TLS\.
+ UDP support — Because a Network Load Balancer operates at the fourth layer of the OSI model, it is ideal for non HTTP workloads and protocols other than TCP\.

## Amazon API Gateway HTTP API<a name="networking-apigateway"></a>

Amazon API Gateway HTTP API is a serverless ingress that is ideal for HTTP applications with wide ranging, bursty request volumes, or very low request volumes\. For more information, see [What is Amazon API Gateway?](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html) in the *API Gateway Developer Guide*\.

![\[Diagram showing architecture of a network using API Gateway.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/apigateway-ingress.png)

The pricing model for Application Load Balancers and Network Load Balancers include an hourly price to keep the load balancers available for accepting incoming connections at all times\. In contrast, API Gateway charges per request\. So if no requests come in, there are no charges\. Under high traffic loads, an Application Load Balancer or Network Load Balancer can handle a greater volume of requests at a cheaper per request price than API Gateway but if you have a low number of requests overall or periods of extremely low traffic then the cumulative price for using the API Gateway would be cheaper than paying a constant hourly charge to maintain a load balancer that is being underutilized\.

For example, an Application Load Balancer always costs at least $16\.20 a month because of its hourly charge\. On API Gateway HTTP API you could serve 16 million requests before you hit $16 in charges\. So low traffic use cases under 16 million requests per month would be cheaper on API Gateway HTTP API as opposed to using an Application Load Balancer\. However, an Application Load Balancer that is being utilized by clients with keep alive connections can serve hundreds of millions of requests per month for under $20\.

API Gateway functions using a VPC link that allows the AWS managed service to connect to hosts inside the private subnet of your VPC, using its private IP address\. It can discover these private IP addresses by looking at AWS Cloud Map service discovery records that are managed by Amazon ECS service discovery\.

API Gateway supports the following features\.
+ SSL/TLS termination
+ Routing different HTTP paths to different backend microservices

It also supports the usage of custom Lambda authorizers that help you protect your API from unauthorized usage\. For more information, see [Field Notes: Serverless Container\-based APIs with Amazon ECS and Amazon API Gateway](http://aws.amazon.com/blogs/architecture/field-notes-serverless-container-based-apis-with-amazon-ecs-and-amazon-api-gateway/)\.