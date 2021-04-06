# Optimizing and troubleshooting<a name="networking-troubleshooting"></a>

When developing an architecture that has significant networking needs it can be hard to troubleshoot networking issues, and figure out what to optimize\. Let’s look at some of the tools that are available for understanding your networking and then some configurations and general tips for optimizing your networking\.

## CloudWatch Container Insights<a name="networking-troubleshooting-containerinsights"></a>

CloudWatch Container Insights collects, aggregates, and summarizes metrics and logs from your containerized applications and microservices\. The metrics include utilization for resources such as CPU, memory, disk, and network\. The metrics are available in CloudWatch automatic dashboards\. For more information, see [Setting up Container Insights on Amazon ECS](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/deploy-container-insights-ECS.html) in the *Amazon CloudWatch User Guide*\.

## AWS X\-Ray<a name="networking-troubleshooting-xray"></a>

AWS X\-Ray is a tracing service that helps you collect information about the network requests that your application makes\. You can use the SDK to instrument your application and capture timings and response codes of traffic between your own services, and between your services and AWS service endpoints\. For more information, see [What is AWS X\-Ray](https://docs.aws.amazon.com/xray/latest/devguide/aws-xray.html) in the *AWS X\-Ray Developer Guide*\.

![\[Diagram showing architecture of X-Ray.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/xray.png)

You can also explore graphs of how your services network with each other, and explore aggregate stats about how each service to service link is performing\. You can also dive deeper into a specific transaction to see segments representing network calls associated with that particular transaction\.

![\[Diagram showing architecture of an X-Ray timeline.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/xray-timeline.png)

These tools can make it much easier to identify when there is a networking bottleneck, or when a specific service within your network is not performing as expected\.

## VPC Flow Logs<a name="networking-troubleshooting-vpcflowlogs"></a>

Amazon VPC flow logs are another powerful tool that helps you to analyze your network performance and debug connectivity issues\. With VPC flow logs enabled, you can capture a log of all the connections in your VPC, including connections to networking interfaces associated with Elastic Load Balancing, Amazon RDS, NAT gateways, and other key AWS services you might be using\. For more information, see [VPC Flow Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html) in the *Amazon VPC User Guide*\.

## Network tuning tips<a name="networking-troubleshooting-tuning"></a>

There are a few settings that you can fine tune in order to improve your networking\.

### nofile ulimit<a name="networking-troubleshooting-tuning-nofile"></a>

If you expect your application to have high traffic and handle many concurrent connections, then you will need to consider the system limit on number of files\. When there are a lot of network sockets open each one needs to be represented by a file descriptor\. If your file descriptor limit is too low then it will have the unintended effect of limiting your network sockets, resulting in failed connections or errors\. You can update the container specific ulimit on number of files in the Amazon ECS task definition\. If running on Amazon EC2 instead of AWS Fargate then you may also need to adjust these limits on your underlying Amazon EC2 instance as well\.

### sysctl net<a name="networking-troubleshooting-tuning-sysctl"></a>

Another category of tunable setting is the `sysctl` net settings\. You should refer to the specific settings for your Linux distribution of choice, but many of these settings adjust the size of read and write buffers which can help in some situations when running really large Amazon EC2 instances that have a lot of containers on them\.