# Load balancer connection draining<a name="load-balancer-connection-draining"></a>

To allow for optimization, clients maintain a keep alive connection to the container service\. This is so that subsequent requests from that client can reuse the existing connection\. When you want to stop traffic to a container, you notify the load balancer\. 

The following diagram describes the load balancer connection draining process\. When you tell the load balancer to stop traffic to the container, it periodically checks to see if the client closed the keep alive connection\. The Amazon ECS agent monitors the load balancer and waits the load balancer to report that the keep alive connection is closed\.

![\[Diagram showing the load balancer container draining.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/load-balancer-connection-drainig.PNG)

The amount of time that the load balancer waits is the deregistraion delay\. You can configure the following load balancer setting to speed up your deployments\.
+ `deregistration_delay.timeout_seconds`: 300 \(default\)

When you have a service with a response time that's under one second, set the parameter to the following value to have the load balancer only wait five seconds before it breaks the connection between the client and the backend service: 
+ `deregistration_delay.timeout_seconds`: 5 

**Note**  
Do not set the value to 5 seconds when you have a service with long\-lived requests, such as slow file uploads or streaming connections\.

## SIGTERM responsiveness<a name="sigterm"></a>

The following diagram shows how Amazon ECS terminates a task\. Amazon ECS first sends a SIGTERM signal to the task to notify the application needs to finish and shut down, and then the sends a SIGKILL message\. When applications ignore the SIGTERM, the Amazon ECS service must wait to send the SIGKILL signal to terminate the process\.

![\[Diagram showing the Amazon ECS service sending the SIGTERM and SIGKILL messages.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/ecs-sigterm.PNG)

The amount of time that Amazon ECS waits is determined by the following Amazon ECS agent option:
+ `ECS_CONTAINER_STOP_TIMEOUT`: 30 \(default\)

  For more information about the container agent parameter, see [Container agent configuration](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-agent-config.html) in the *Amazon Elastic Container Service Developer Guide*\.

To speed up the waiting period, set the Amazon ECS agent option to the following value:

**Note**  
If your application takes more than one second, multiply the value by two and use that number as the value\.
+ `ECS_CONTAINER_STOP_TIMEOUT`: 2

In this case, the Amazon ECS waits two seconds for the container to shut down, and then Amazon ECS sends a SIGKILL message when the application didn't stop\.

You can also modify the application code to trap the SIGTERM signal and react to it\. The following is example in JavaScript: 

```
process.on('SIGTERM', function() { 
  server.close(); 
})
```

This code causes the HTTP server to stop listening for any new requests, finish answering any in\-flight requests, and then the Node\.js process terminates\. This is because its event loop has nothing left to do\. Given this, if it takes the process only 500 ms to finish its in\-flight requests, it terminates early without having to wait out the stop timeout and get sent a SIGKILL\. 