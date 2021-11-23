# AWS Identity and Access Management<a name="security-iam"></a>

You can use AWS Identity and Access Management \(IAM\) to manage and control access to your AWS services and resources through rule\-based policies for authentication and authorization purposes\. More specifically, through this service, you control access to your AWS resources by using policies that are applied to IAM users, groups, or roles\. Among these three, IAM users are accounts that can have access to your resources\. And, an IAM role is a set of permissions that can be assumed by an authenticated identity, which isn't associated with a particular identity outside of IAM\. For more information, see [Policies and permissions in IAM?](IAM/latest/UserGuide/access_policies.html)\.

## Managing access to Amazon ECS<a name="security-iam-managing"></a>

You can control access to Amazon ECS by creating and applying IAM policies\. These policies are composed of a set of actions that apply to a specific set of resources\. The action of a policy defines the list of operations \(such as Amazon ECS APIs\) that are allowed or denied, whereas the resource controls what are the Amazon ECS objects that the action applies to\. Conditions can be added to a policy to narrow its scope\. For example, a policy can be written to only allow an action to be performed against tasks with a particular set of tags\. For more information, see [How Amazon ECS works with IAM](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/security_iam_service-with-iam.html) in the *Amazon Elastic Container Service Developer Guide*\.

## Recommendations<a name="security-iam-recommendations"></a>

We recommend that you do the following when setting up your IAM roles and policies\.

### Follow the policy of least privileged access<a name="security-iam-recommendations-leastpriv"></a>

Create policies that are scoped to allow users to perform their prescribed jobs\. For example, if a developer needs to periodically stop a task, create a policy that only permits that particular action\. The following example only allows a user to stop a task that belongs to a particular `task_family` on a cluster with a specific Amazon Resource Name \(ARN\)\. Refering to an ARN in a condition is also an example of using resource\-level permissions\. You can use resource\-level permissionsto specify the resource that you want an action to apply to\.

**Note**  
When referencing an ARN in a policy, use the new longer ARN format\. For more information, see [Amazon Resource Names \(ARNs\) and IDs](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-account-settings.html#ecs-resource-ids) in the *Amazon Elastic Container Service Developer Guide*\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecs:StopTask"
      ],
      "Condition": {
        "ArnEquals": {
          "ecs:cluster": "arn:aws:ecs:region:account_id:cluster/cluster_name"
        }
      },
      "Resource": [
        "arn:aws:ecs:region:account_id:task-definition/task_family:*"
      ]
    }
  ]
}
```

### Let the cluster resource serve as the administrative boundary<a name="security-iam-recommendations-clusterboundary"></a>

Policies that are too narrowly scoped can cause a proliferation of roles and increase administrative overhead\. Rather than creating roles that are scoped to particular tasks or services only, create roles that are scoped to clusters and use the cluster as your primary administrative boundary\.

### Isolate end\-users from the Amazon ECS API by creating automated pipelines<a name="security-iam-recommendations-usingpipelines"></a>

You can limit the actions that users can use by creating pipelines that automatically package and deploy applications onto Amazon ECS clusters\. This effectively delegates the job of creating, updating, and deleting tasks to the pipeline\. For more information, see [Tutorial: Amazon ECS standard deployment with CodePipeline](https://docs.aws.amazon.com/codepipeline/latest/userguide/ecs-cd-pipeline.html) in the *AWS CodePipeline User Guide*\.

### Use policy conditions for an added layer of security<a name="security-iam-recommendations-policyconditions"></a>

When you need an added layer of security, add a condition to your policy\. This can be useful if you're performing a privileged operation or when you need to restrict the set of actions that can be performed against particular resources\. The following example policy requires multi\-factor authorization when deleting a cluster\.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecs:DeleteCluster"
      ],
      "Condition": {
        "Bool": {
          "aws:MultiFactorAuthPresent": "true"
        }
      },
    "Resource": ["*"]
    }
  ]
}
```

Tags applied to services are propagated to all the tasks that are part of that service\. Because of this, you can create roles that are scoped to Amazon ECS resources with specific tags\. In the following policy, an IAM principals starts and stops all tasks with a tag\-key of `Department` and a tag\-value of `Accounting`\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecs:StartTask",
                "ecs:StopTask",
                "ecs:RunTask"
            ],
            "Resource": "arn:aws:ecs:*",
            "Condition": {
                "StringEquals": {"ecs:ResourceTag/Department": "Accounting"}
            }
        }
    ]
}
```

### Periodically audit access to the Amazon ECS APIs<a name="security-iam-recommendations-audit"></a>

A user might change roles\. After they change roles, the permissions that were previously granted to them might no longer apply\. Make sure that you audit who has access to the Amazon ECS APIs and whether that access is still warranted\. Consider integrating IAM with a user lifecycle management solution that automatically revokes access when a user leaves the organization\. For more information, see [Amazon ECS security audit guidelines](https://docs.aws.amazon.com/general/latest/gr/aws-security-audit-guide.html) in the *Amazon Web Services General Reference*\.