# Logging and monitoring<a name="security-logging-and-monitoring"></a>

Logging and monitoring are an important aspect of maintaining the reliability, availability, and performance of Amazon ECS and your AWS solutions\. AWS provides several tools for monitoring your Amazon ECS resources and responding to potential incidents:
+ [Amazon CloudWatch Alarms](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cloudwatch-metrics.html)
+ [Amazon CloudWatch Logs](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/using_awslogs.html) 
+ [Amazon CloudWatch Events](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cloudwatch_event_stream.html) 
+ [AWS CloudTrail Logs](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/logging-using-cloudtrail.html)

You can configure the containers in your tasks to send log information to Amazon CloudWatch Logs\. If you're using the AWS Fargate launch type for your tasks, you can view the logs from your containers\. If you're using the Amazon EC2 launch type, you can view different logs from your containers in one convenient location\. This also prevents your container logs from taking up disk space on your container instances\. 

For more information about Amazon CloudWatch Logs, see **Monitor Logs from Amazon EC2 Instances** in the [Amazon CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/WhatIsCloudWatchLogs.html)\. For instruction on sending container logs from your tasks to Amazon CloudWatch Logs, see [Using the `awslogs` log driver](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/using_awslogs.html)\.

## Container logging with Fluent Bit<a name="security-logging-and-monitoring-fluent-bit"></a>

AWS provides a Fluent Bit image with plugins for both Amazon CloudWatch Logs and Amazon Kinesis Data Firehose\. This image provides the capability to route logs to Amazon CloudWatch and Amazon Kinesis Data Firehose destinations \(which include Amazon S3, Amazon OpenSearch Service, and Amazon Redshift\)\. We recommend using Fluent Bit as your log router because it has a lower resource utilization rate than Fluentd\. For more information, see [Amazon CloudWatch Logs for Fluent Bit](https://github.com/aws/amazon-cloudwatch-logs-for-fluent-bit) and [Amazon Kinesis Data Firehose for Fluent Bit](https://github.com/aws/amazon-kinesis-firehose-for-fluent-bit)\.

The AWS for Fluent Bit image is available on:
+ [Amazon ECR on Amazon ECR Public Gallery](https://gallery.ecr.aws/aws-observability/aws-for-fluent-bit)
+ [Amazon ECR repository](https://github.com/aws/aws-for-fluent-bit#amazon-ecr) \(in most Regions of high availability\)

The following shows the syntax to use for the Docker CLI\.

```
docker pull public.ecr.aws/aws-observability/aws-for-fluent-bit:tag
```

For example, you can pull the latest AWS for Fluent Bit image using this Docker CLI command:

```
docker pull public.ecr.aws/aws-observability/aws-for-fluent-bit:latest
```

Also refer to the following blog posts for more information on Fluent Bit and related features:
+ [Fluent Bit for Amazon EKS on AWS Fargate](http://aws.amazon.com/blogs/containers/fluent-bit-for-amazon-eks-on-aws-fargate-is-here/)
+ [Centralized Container Logging with Fluent Bit](http://aws.amazon.com/blogs/opensource/centralized-container-logging-fluent-bit/)
+ [Building a scalable log solution aggregator with AWS Fargate, Fluentd, and Amazon Kinesis Data Firehose](http://aws.amazon.com/blogs/compute/building-a-scalable-log-solution-aggregator-with-aws-fargate-fluentd-and-amazon-kinesis-data-firehose/)

## Custom log routing \- FireLens for Amazon ECS<a name="security-logging-and-monitoring-firelens"></a>

With FireLens for Amazon ECS, you can use task definition parameters to route logs to an AWS service or AWS Partner Network \(APN\) destination for log storage and analytics\. FireLens works with [Fluentd](https://www.fluentd.org/) and [https://fluentbit.io/](https://fluentbit.io/)\. We provide the AWS for Fluent Bit image\. Or, you can alternatively use your own Fluentd or Fluent Bit image\.

You should consider the following conditions and considerations when using FireLens for Amazon ECS:
+ FireLens for Amazon ECS is supported for tasks that are hosted both on AWS Fargate and Amazon EC2\.
+ FireLens for Amazon ECS is supported in AWS CloudFormation templates\. For more information, see [AWS::ECS::TaskDefinition FirelensConfiguration](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ecs-taskdefinition-firelensconfiguration.html) in the **AWS CloudFormation User Guide**\.
+ For tasks that use the `bridge` network mode, containers with the FireLens configuration must start before any of the application containers that rely on it start\. To control the order that your containers start in, use dependency conditions in your task definition\. For more information, see [Container dependency](https://docs.aws.amazon.com/AmazonECS/latest/userguide/task_definition_parameters.html#container_definition_dependson)\.