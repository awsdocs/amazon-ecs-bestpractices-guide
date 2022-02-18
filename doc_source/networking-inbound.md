# Receiving inbound connections from the internet<a name="networking-inbound"></a>

If you run a public service, you must accept inbound traffic from the internet\. For example, your public website must accept inbound HTTP requests from browsers\. In such case, other hosts on the internet must also initiate an inbound connection to the host of your application\.

One approach to this problem is to launch your containers on hosts that are in a public subnet with a public IP address\. However, we don't recommend this for large\-scale applications\. For these, a better approach is to have a scalable input layer that sits between the internet and your application\. For this approach, you can use any of the AWS services listed in this section as an input\. 

**Topics**
+ [Application Load Balancer](#networking-alb)
+ [Network Load Balancer](#networking-nlb)
+ [Amazon API Gateway HTTP API](#networking-apigateway)

## Application Load Balancer<a name="networking-alb"></a>

An Application Load Balancer functions at the application layer\. It's the seventh layer of the Open Systems Interconnection \(OSI\) model\. This makes an Application Load Balancer suitable for public HTTP services\. If you have a website or an HTTP REST API, then an Application Load Balancer is a suitable load balancer for this workload\. For more information, see [What is an Application Load Balancer?](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) in the *User Guide for Application Load Balancers*\.

![\[Diagram showing architecture of a network using an Application Load Balancer.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/alb-ingress.png)

With this architecture, you create an Application Load Balancer in a public subnet so that it has a public IP address and can receive inbound connections from the internet\. When the Application Load Balancer receives an inbound connection, or more specifically an HTTP request, it opens a connection to the application using its private IP address\. Then, it forwards the request over the internal connection\.

An Application Load Balancer has the following advantages\.
+ SSL/TLS termination — An Application Load Balancer can sustain secure HTTPS communication and certificates for communications with clients\. It can optionally terminate the SSL connection at the load balancer level so that you don’t have to handle certificates in your own application\.
+ Advanced routing — An Application Load Balancer can have multiple DNS hostnames\. It also has advanced routing capabilities to send incoming HTTP requests to different destinations based on metrics such as the hostname or the path of the request\. This means that you can use a single Application Load Balancer as the input for many different internal services, or even microservices on different paths of a REST API\.
+ gRPC support and websockets — An Application Load Balancer can handle more than just HTTP\. It can also load balance gRPC and websocket based services, with HTTP/2 support\.
+ Security — An Application Load Balancer helps protect your application from malicious traffic\. It includes features such as HTTP de sync mitigations, and is integrated with AWS Web Application Firewall \(AWS WAF\)\. AWS WAF can further filter out malicious traffic that might contain attack patterns, such as SQL injection or cross\-site scripting\.

## Network Load Balancer<a name="networking-nlb"></a>

A Network Load Balancer functions at the fourth layer of the Open Systems Interconnection \(OSI\) model\. It's suitable for non\-HTTP protocols or scenarios where end\-to\-end encryption is necessary, but doesn’t have the same HTTP\-specific features of an Application Load Balancer\. Therefore, a Network Load Balancer is best suited for applications that don’t use HTTP\. For more information, see [What is a Network Load Balancer?](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html) in the *User Guide for Network Load Balancers*\.

![\[Diagram showing architecture of a network using an Network Load Balancer.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/nlbingress.png)

When a Network Load Balancer is used as an input, it functions similarly to an Application Load Balancer\. This is because it's created in a public subnet and has a public IP address that can be accessed on the internet\. The Network Load Balancer then opens a connection to the private IP address of the host running your container, and sends the packets from the public side to the private side\.

**Network Load Balancer features**  
Because the Network Load Balancer operates at a lower level of the networking stack, it doesn't have the same set of features that Application Load Balancer does\. However, it does have the following important features\.
+ End\-to\-end encryption — Because a Network Load Balancer operates at the fourth layer of the OSI model, it doesn't read the contents of packets\. This makes it suitable for load balancing communications that need end\-to\-end encryption\.
+ TLS encryption — In addition to end\-to\-end encryption, Network Load Balancer can also terminate TLS connections\. This way, your backend applications don’t have to implement their own TLS\.
+ UDP support — Because a Network Load Balancer operates at the fourth layer of the OSI model, it's suitable for non HTTP workloads and protocols other than TCP\.

**Closing connections**  
Because the Network Load Balancer does not observe the application protocol at the higher layers of the OSI model, it cannot send closure messages to the clients in those protocols\. Unlike the Application Load Balancer, those connections need to be closed by the application or you can configure the Network Load Balancer to close the fourth layer connections when a task is stopped or replaced\. See the connection termination setting for Network Load Balancer target groups in the [Network Load Balancer documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-target-groups.html#deregistration-delay)\.

Letting the Network Load Balancer close connections at the fourth layer can cause clients to display undesired error messages, if the client does not handle them\. See the Builders Library for more information on recommended client configuration [here](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter)\.

The methods to close connections will vary by application, however one way is to ensure that the Network Load Balancer target deregistration delay is longer than client connection timeout\. The client would timeout first and reconnect gracefully through the Network Load Balancer to the next task while the old task slowly drains all of its clients\. For more information about the Network Load Balancer target deregistration delay, see the [Network Load Balancer documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-target-groups.html#deregistration-delay)\. 

## Amazon API Gateway HTTP API<a name="networking-apigateway"></a>

Amazon API Gateway HTTP API is a serverless ingress that's suitable for HTTP applications with sudden bursts in request volumes or low request volumes\. For more information, see [What is Amazon API Gateway?](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html) in the *API Gateway Developer Guide*\.

![\[Diagram showing architecture of a network using API Gateway.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/apigateway-ingress.png)

The pricing model for both Application Load Balancer and Network Load Balancer include an hourly price to keep the load balancers available for accepting incoming connections at all times\. In contrast, API Gateway charges for each request separately\. This has the effect that, if no requests come in, there are no charges\. Under high traffic loads, an Application Load Balancer or Network Load Balancer can handle a greater volume of requests at a cheaper per\-request price than API Gateway\. However, if you have a low number of requests overall or have periods of low traffic, then the cumulative price for using the API Gateway should be more cost effective than paying a hourly charge to maintain a load balancer that's being underutilized\. The API Gateway can also cache API responses, which might result in lower backend request rates\.

API Gateway functions which use a VPC link that allows the AWS managed service to connect to hosts inside the private subnet of your VPC, using its private IP address\. It can detect these private IP addresses by looking at AWS Cloud Map service discovery records that are managed by Amazon ECS service discovery\.

API Gateway supports the following features\.
+ The API Gateway operatation is similar to a load balancer, but has additional capabilities unique to API management
+ The API Gateway provides additional capabilities around client authorization, usage tiers, and request/response modification\. For more information, see [Amazon API Gateway features](http://aws.amazon.com/api-gateway/features/)\.
+ The API Gateway can support edge, regional, and private API gateway endpoints\. Edge endpoints are available through a managed CloudFront distribution\. Regional and private endpoints are both local to a Region\.
+ SSL/TLS termination
+ Routing different HTTP paths to different backend microservices

Besides the preceding features, API Gateway also supports using custom Lambda authorizers that you can use to protect your API from unauthorized usage\. For more information, see [Field Notes: Serverless Container\-based APIs with Amazon ECS and Amazon API Gateway](http://aws.amazon.com/blogs/architecture/field-notes-serverless-container-based-apis-with-amazon-ecs-and-amazon-api-gateway/)\.