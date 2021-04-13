# Optimizing and troubleshooting<a name="networking-troubleshooting"></a>

The following services and features can help you to gain insights about your network and service configurations\. You can use this information to troubleshoot networking issues and better optimize your servcies\.

## CloudWatch Container Insights<a name="networking-troubleshooting-containerinsights"></a>

CloudWatch Container Insights collects, aggregates, and summarizes metrics and logs from your containerized applications and microservices\. Metrics include the utilization of resources such as CPU, memory, disk, and network\. They're available in CloudWatch automatic dashboards\. For more information, see [Setting up Container Insights on Amazon ECS](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/deploy-container-insights-ECS.html) in the *Amazon CloudWatch User Guide*\.

## AWS X\-Ray<a name="networking-troubleshooting-xray"></a>

AWS X\-Ray is a tracing service that you can use to collect information about the network requests that your application makes\. You can use the SDK to instrument your application and capture timings and response codes of traffic between your services, and between your services and AWS service endpoints\. For more information, see [What is AWS X\-Ray](https://docs.aws.amazon.com/xray/latest/devguide/aws-xray.html) in the *AWS X\-Ray Developer Guide*\.

![\[Diagram showing architecture of X-Ray.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/xray.png)

You can also explore AWS X\-Ray graphs of how your services network with each other\. Or, use them to explore aggregate statistics about how each service\-to\-service link is performing\. Last, you can dive deeper into any specific transaction to see how segments representing network calls are associated with that particular transaction\.

![\[Diagram showing architecture of an X-Ray timeline.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/xray-timeline.png)

You can use these features to identify if there's a networking bottleneck or if a specific service within your network isn't performing as expected\.

## VPC Flow Logs<a name="networking-troubleshooting-vpcflowlogs"></a>

You can use Amazon VPC flow logs to analyze network performance and debug connectivity issues\. With VPC flow logs enabled, you can capture a log of all the connections in your VPC\. These include connections to networking interfaces that are associated with Elastic Load Balancing, Amazon RDS, NAT gateways, and other key AWS services that you might be using\. For more information, see [VPC Flow Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html) in the *Amazon VPC User Guide*\.

## Network tuning tips<a name="networking-troubleshooting-tuning"></a>

There are a few settings that you can fine tune in order to improve your networking\.

### nofile ulimit<a name="networking-troubleshooting-tuning-nofile"></a>

If you expect your application to have high traffic and handle many concurrent connections, you should take into account the system quota for the number of files allowed\. When there are a lot of network sockets open, each one must be represented by a file descriptor\. If your file descriptor quota is too low, it will limit your network sockets,\. This results in failed connections or errors\. You can update the container specific quota for the number of files in the Amazon ECS task definition\. If you're running on Amazon EC2 \(instead of AWS Fargate\), then you might also need to adjust these quotas on your underlying Amazon EC2 instance\.

### sysctl net<a name="networking-troubleshooting-tuning-sysctl"></a>

Another category of tunable setting is the `sysctl` net settings\. You should refer to the specific settings for your Linux distribution of choice\. Many of these settings adjust the size of read and write buffers\. This can help in some situations when running large\-scale Amazon EC2 instances that have a lot of containers on them\.