# Handling throttling issues<a name="operating-at-scale-dealing-with-throttles"></a>

This section provides an in\-depth overview of some strategies that you can use to monitor and troubleshoot API throttling errors\. Throttling errors fall into two major categories: *synchronous* throttling and *asynchronous* throttling\.

## Synchronous throttling<a name="synchronous-throttling"></a>

When synchronous throttling occurs, you immediately receive an error response from Amazon ECS\. This category of throttling typically occurs when you call Amazon ECS APIs while running tasks or creating services\. For more information about the throttling involved and the relevant throttle limits, see [Request throttling for the Amazon ECS API](https://docs.aws.amazon.com/AmazonECS/latest/APIReference/request-throttling.html)\.

When your application initiates API requests, for example, by using the AWS CLI or an AWS SDK, you can remediate API throttling\. You can do this by either architecting your application to handle the errors or by implementing an exponential backoff and jitter strategy with retry logic for the API calls\. For more information, see [Timeouts, retries, and backoff with jitter](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/)\.

If you use an AWS SDK, the automatic retry logic is already built\-in and configurable\.

## Asynchronous throttling<a name="asynchronous-throttling"></a>

Asynchronous throttling occurs because of asynchronous workflows where Amazon ECS or AWS CloudFormation might be calling APIs on your behalf to provision resources\. It's important to know which AWS APIs that Amazon ECS invokes on your behalf\. For example, the `CreateNetworkInterface` API is invoked for tasks that use the `awsvpc` network mode, and the `DescribeTargetHealth` API is invoked when performing health checks for tasks registered to a load balancer\.

When your workloads reach a considerable scale, these API operations might be throttled\. That is, they might be throttled enough to breach the limits enforced by Amazon ECS or the AWS service that is being called\. For example, if you deploy hundreds of services, each having hundreds of tasks concurrently that use the `awsvpc` network mode, Amazon ECS invokes Amazon EC2 API operations such as `CreateNetworkInterface` and Elastic Load Balancing API operations such as `RegisterTarget` or `DescribeTargetHealth` to register the elastic network interface and load balancer, respectively\. These API calls can exceed the API limits, resulting in throttling errors\. The following is an example of an Elastic Load Balancing throttling error that's included in the service event message\.

```
{
   "userIdentity":{
      "arn":"arn:aws:sts::111122223333:assumed-role/AWSServiceRoleForECS/ecs-service-scheduler",
      "eventTime":"2022-03-21T08:11:24Z",
      "eventSource":"elasticloadbalancing.amazonaws.com",
      "eventName":" DescribeTargetHealth ",
      "awsRegion":"us-east-1",
      "sourceIPAddress":"ecs.amazonaws.com",
      "userAgent":"ecs.amazonaws.com",
      "errorCode":"ThrottlingException",
      "errorMessage":"Rate exceeded",
      "eventID":"0aeb38fc-229b-4912-8b0d-2e8315193e9c"
   }
}
```

When these API calls share limits with other API traffic in your account, they might be difficult monitor even though they're emitted as service events\.

## Monitoring throttling<a name="monitoring-throttling"></a>

To monitor throttling, it's important to identify which API requests are throttled and who issues these requests\. You can use AWS CloudTrail to do it\. This service monitors throttling, and can be integrated with CloudWatch, Amazon Athena, and Amazon EventBridge\. You can configure CloudTrail to send specific events to CloudWatch Logs\. These events are then parsed and analyzed using CloudWatch Logs log insights\. This identifies details in throttling events such as the user or IAM role that made the call and the number of API calls that were made\. For more information, see [Monitoring CloudTrail log files with CloudWatch Logs](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/monitor-cloudtrail-log-files-with-cloudwatch-logs.html)\.

For more information about CloudWatch Logs insights and instructions on how to query log files, see [Analyzing log data with CloudWatch Logs Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AnalyzingLogData.html)\.

With Amazon Athena, you can create queries and analyze data using standard SQL\. For example, you can create an Athena table to parse CloudTrail events\. For more information, see [Using the CloudTrail console to create an Athena table for CloudTrail logs](https://docs.aws.amazon.com/athena/latest/ug/cloudtrail-logs.html#create-cloudtrail-table-ct)\.

After creating an Athena table, you can use simple SQL queries such as the following one to investigate `ThrottlingException` errors\.

```
select eventname, errorcode,eventsource,awsregion, useragent,COUNT(*) count
FROM cloudtrail-table-name
where errorcode = 'ThrottlingException'
AND eventtime between '2022-01-14T03:00:08Z' and '2022-01-23T07:15:08Z'
group by errorcode, awsregion, eventsource, username, eventname
order by count desc;
```

Amazon ECS also emits event notifications to Amazon EventBridge\. There are resource state change events and service action events\. They include API throttling events such as `ECS_OPERATION_THROTTLED` and `SERVICE_DISCOVERY_OPERATION_THROTTLED`\. For more information, see [Amazon ECS service action events](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_cwe_events.html#ecs_service_events)\.

These events can be consumed by a service such as AWS Lambda to perform actions in response\. For more information, see [Handling events](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_cwet_handling.html)\.

If you run standalone tasks, some API operations such as `RunTask` are asynchronous, and retry operations aren't automatically performed\. In such cases, you can use services such as AWS Step Functions with EventBridge integration to retry throttled or failed operations\. For more information, see [Manage a container task \(Amazon ECS, Amazon SNS\)](https://docs.aws.amazon.com/step-functions/latest/dg/sample-project-container-task-notification.html)\.

## Using CloudWatch to monitor throttling<a name="monitoring-throttling-cw"></a>

CloudWatch offers API usage monitoring on the `Usage` namespace under **By AWS Resource**\. These metrics are logged with type **API** and metric name **CallCount**\. You can create alarms to start whenever these metrics reach a certain threshold\. For more information, see [Visualizing your service quotas and setting alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Quotas-Visualize-Alarms.html)\.

CloudWatch also offers anomaly detection\. This feature uses machine learning to analyze and establish baselines based on the particular behavior of the metric that you enabled it on\. If there's unusual API activity, you can use this feature together with CloudWatch alarms\. For more information, see [Using CloudWatch anomaly detection](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Anomaly_Detection.html)\.

By proactively monitoring throttling errors, you can contact AWS Support to increase the relevant throttling limits and also receive guidance for your unique application needs\.