# Using IAM roles with Amazon ECS tasks<a name="security-iam-roles"></a>

We recommend that you assign a task an IAM role\. Its role can be distinguished from the role of the Amazon EC2 instance that it's running on\. Assigning each task a role aligns with the principle of least priviledged access and allows for greater granular control over actions and resources\.

When assigning IAM roles for a task, you must use the folllowing trust policy so that each of your tasks can assume an IAM role that's different from the one that your EC2 instance uses\. This way, your task doesn't inherit the role of your EC2 instance\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Service": "ecs-tasks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

When you add a task role to a task definition, the Amazon ECS container agent automatically creates a token with a unique credential ID \(for example, `12345678-90ab-cdef-1234-567890abcdef`\) for the task\. This token and the role credentials are then added to the agent's internal cache\. The agent populates the the environment variable `AWS_CONTAINER_CREDENTIALS_RELATIVE_URI` in the container with the URI of the credential ID \(for example, `/v2/credentials/12345678-90ab-cdef-1234-567890abcdef`\)\.

![\[This workflow shows the process involved when the Amazon ECS container agent caches credentials. These credentials are determined by the task role that is defined in the task definition.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/iam-roles-with-ecs-workflow.png)

You can manually retrieve the temporary role credentials from inside a container by appending the environment variable to the IP address of the Amazon ECS container agent and running the `curl` command on the the resulting string\.

```
curl 192.0.2.0$AWS_CONTAINER_CREDENTIALS_RELATIVE_URI
```

The expected output is as follows:

```
{
	"RoleArn": "arn:aws:iam::123456789012:role/SSMTaskRole-SSMFargateTaskIAMRole-DASWWSF2WGD6",
	"AccessKeyId": "AKIAIOSFODNN7EXAMPLE",
	"SecretAccessKey": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
	"Token": "IQoJb3JpZ2luX2VjEEM/Example==",
	"Expiration": "2021-01-16T00:51:53Z"
}
```

Newer versions of the AWS SDKs automatically fetch these credentials from the `AWS_CONTAINER_CREDENTIALS_RELATIVE_URI` environment varible when making AWS API calls\.

The output includes an access key\-pair consisting of a secret access key ID and a secret key which your application uses to access AWS resources\. It also includes a token that AWS uses to verify that the credentials are valid\. By default, credentials assigned to tasks using task roles are valid for six hours\. After that, they are automatically rotated by the Amazon ECS container agent\.

## Task execution role<a name="security-iam-roles-task-execution"></a>

The task execution role is used to grant the Amazon ECS container agent permission to call specific AWS API actions on your behalf\. For example, when you use AWS Fargate, Fargate needs an IAM role that allows it to pull images from Amazon ECR and write logs to CloudWatch Logs\. An IAM role is also required when a task references a secret that's stored in AWS Secrets Manager, such as an image pull secret\.

**Note**  
If you're pulling images as an authenticated user, you're less likely to be impacted by the changes that occurred to [Docker Hub's pull rate limits](https://www.docker.com/pricing/resource-consumption-updates)\. For more information see, [Private registry authentication for container instances](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/private-auth-container-instances.html)\.  
By using Amazon ECR and Amazon ECR Public, you can avoid the limits imposed by Docker\. If you pull images from Amazon ECR, this also helps shorten network pull times and reduces data transfer changes when traffic leaves your VPC\.

**Important**  
When you use Fargate, you must authenticate to a private image registry using `repositoryCredentials`\. It's not possible to set the Amazon ECS container agent environment variables `ECS_ENGINE_AUTH_TYPE` or `ECS_ENGINE_AUTH_DATA` or modify the `ecs.config` file for tasks hosted on Fargate\. For more information, see [Private registry authentication for tasks](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/private-auth.html)\.

## Amazon EC2 container instance role<a name="security-iam-roles-ec2-container-instance"></a>

The Amazon ECS container agent is a container that runs on each Amazon EC2 instance in an Amazon ECS cluster\. It's initialized outside of Amazon ECS using the `init` command that's available on the operating system\. Consequently, it can't be granted permissions through a task role\. Instead, the permissions have to be assigned to the Amazon EC2 instances that the agents run on\. The actions list in the example `AmazonEC2ContainerServiceforEC2Role` policy need to be granted to the `ecsInstanceRole`\. If you don't do this, your instances cannot join the cluster\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeTags",
                "ecs:CreateCluster",
                "ecs:DeregisterContainerInstance",
                "ecs:DiscoverPollEndpoint",
                "ecs:Poll",
                "ecs:RegisterContainerInstance",
                "ecs:StartTelemetrySession",
                "ecs:UpdateContainerInstancesState",
                "ecs:Submit*",
                "ecr:GetAuthorizationToken",
                "ecr:BatchCheckLayerAvailability",
                "ecr:GetDownloadUrlForLayer",
                "ecr:BatchGetImage",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "*"
        }
    ]
}
```

In this policy, the `ecr` and `logs` api actions allow the containers that are running on your instances to pull images from Amazon ECR and write logs to Amazon CloudWatch\. The `ecs` actions allow the agent to register and de\-register instances and to communicate with the Amazon ECS control plane\. Of these, the `ecs:CreateCluster` action is optional\.

## Service\-linked roles<a name="security-iam-roles-service-linked"></a>

You can use the service\-linked role for Amazon ECS to grant the Amazon ECS service permission to call other service APIs on your behalf\. Amazon ECS needs the permissions to create and delete network interfaces, register, and de\-register targets with a target group\. It also needs the necessary permissions to create and delete scaling policies\. These permissions are granted through the service\-linked role\. This role is created on your behalf the first time that you use the service\.

**Note**  
If you inadvertantly delete the service\-linked role, you can recreate it\. For instructions, see [Create the service\-linked role](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/using-service-linked-roles.html#create-service-linked-role)\.

## Recommendations<a name="security-iam-roles-recommendations"></a>

We recommend that you do the following when setting up your task IAM roles and policies\.

### Block access to Amazon EC2 metadata<a name="security-iam-roles-recommendations-ec2-metadata"></a>

When you run your tasks on Amazon EC2 instances, we strongly recommend that you block access to Amazon EC2 metadata to prevent your containers from inheriting the role assigned to those instances\. If your applications have to call an AWS API action, use IAM roles for tasks instead\.

To prevent tasks running in **bridge** mode from accessing Amazon EC2 metadata, run the following command or update the instance's user data\. For more instruction on updating the user data of an instance, see this [AWS Support Article](https://aws.amazon.com/premiumsupport/knowledge-center/ecs-container-ec2-metadata/)\. For more information about the task definition bridge mode, see [task definition network mode](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html#network_mode)\.

```
sudo yum install -y iptables-services; sudo iptables --insert FORWARD 1 --in-interface docker+ --destination 192.0.2.0/32 --jump DROP
```

For this change to persist after a reboot, run the following command that's specific for your Amazon Machine Image \(AMI\):
+ Amazon Linux 2

  ```
  sudo iptables-save | sudo tee /etc/sysconfig/iptables && sudo systemctl enable --now iptables
  ```
+ Amazon Linux

  ```
  sudo service iptables save
  ```

For tasks that use `awsvpc` network mode, set the environment variable `ECS_AWSVPC_BLOCK_IMDS` to `true` in the `/etc/ecs/ecs.config` file\.

You should set the `ECS_ENABLE_TASK_IAM_ROLE_NETWORK_HOST` variable to `false` in the ecs\-agent config file to prevent the containers that are running within the `host` network from accessing the Amazon EC2 metadata\.

### Use `awsvpc` network mode<a name="security-iam-roles-recommendations-awsvpc-networking-mode"></a>

Use the network `awsvpc` network mode to restrict the flow of traffic between different tasks or between your tasks and other services that run within your Amazon VPC\. This adds an additional layer of security\. The `awsvpc` network mode provides task\-level network isolation for tasks that run on Amazon EC2\. It is the default mode on AWS Fargate\. it's the only network mode that you can use to assign a security group to tasks\. 

### Use IAM Access Advisor to refine roles<a name="security-iam-roles-recommendations-iam-access-advisor-refine-roles"></a>

We recommend that you remove any actions that were never used or haven't been used for some time\. This prevents unwanted access from happening\. To do this, review the results produced by IAM Access Advisor, and then remove actions that were never used or haven't been used recently\. You can do this by following the following steps\.

Run the following command to generate a report showing the last access information for the referenced policy:

```
aws iam generate-service-last-accessed-details --arn arn:aws:iam::123456789012:policy/ExamplePolicy1
```

use the `JobId` that was in the output to run the following command\. After you do this, you can view the results of the report\.

```
aws iam get-service-last-accessed-details --job-id 98a765b4-3cde-2101-2345-example678f9
```

For more information, see [IAM Access Advisor](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_access-advisor.html)\.

### Monitor AWS CloudTrail for suspicious activity<a name="security-iam-roles-recommendations-cloudtrail-monitoring"></a>

You can monitor AWS CloudTrail for any suspicious activity\. Most AWS API calls are logged to AWS CloudTrail as events\. They are analyzed by AWS CloudTrail Insights, and you're alerted of any suspicious behavior that's associated with `write` API calls\. This might includea spike in call volume\. These alerts include such information as the time the unusual activity occurred and the top identity ARN that contributed to the APIs\. 

You can identify actions that are performed by tasks with an IAM role in AWS CloudTrail by looking at the event's `userIdentity` property\. In the following example, the `arn` includes of the name of the assumed role, `s3-write-go-bucket-role`, followed by the name of the task, `7e9894e088ad416eb5cab92afExample`\.

```
"userIdentity": {
    "type": "AssumedRole",
    "principalId": "AROA36C6WWEJ2YEXAMPLE:7e9894e088ad416eb5cab92afExample",
    "arn": "arn:aws:sts::123456789012:assumed-role/s3-write-go-bucket-role/7e9894e088ad416eb5cab92afExample",
    ...
}
```

**Note**  
When tasks that assume a role are run on Amazon EC2 container instances, a request is logged by Amazon ECS container agent to the audit log of the agent that's located at an address in the `/var/log/ecs/audit.log.YYYY-MM-DD-HH` format\. For more information, see [Task IAM Roles Log](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/logs.html#task_iam_roles-logs) and [Logging Insights Events for Trails](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-insights-events-with-cloudtrail.html)\.