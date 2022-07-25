# Secrets management<a name="security-secrets-management"></a>

Secrets, such as API keys and database credentials, are frequently used by applications to gain access to other systems\. They often consist of a username and password, a certificate, or API key\. Access to these secrets should be restricted to specific IAM principals that are using IAM and injected into containers at runtime\.

Secrets can be seamlessly injected into containers from AWS Secrets Manager and Amazon EC2 Systems Manager Parameter Store\. These secrets can be referenced in your task as any of the following\.

1. They're referenced as environment variables that use the `secrets` container definition parameter\.

1. They're referenced as `secretOptions` if your logging platform requires authentication\. For more information, see [logging configuration options](https://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_LogConfiguration.html#API_LogConfiguration_Contents)\.

1. They're referenced as secrets pulled by images that use the `repositoryCredentials` container definition parameter if the registry where the container is being pulled from requires authentication\. Use this method when pulling images from Amazon ECR Public Gallery\. For more information, see [Private registry authentication for tasks](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/private-auth.html)\.

## Recommendations<a name="security-secrets-management-recommendations"></a>

We recommend that yo do the following when setting up secrets management\.

### Use AWS Secrets Manager or Amazon EC2 Systems Manager Parameter Store for storing secret materials<a name="security-secrets-management-recommendations-storing-secret-materials"></a>

You should securely store API keys, database credentials, and other secret materials in AWS Secrets Manager or as an encrypted parameter in Amazon EC2 Systems Manager Parameter Store\. These services are similar because they're both managed key\-value stores that use AWS KMS to encrypt sensitive data\. AWS Secrets Manager, however, also includes the ability to automatically rotate secrets, generate random secrets, and share secrets across AWS accounts\. If you deem these important features, use AWS Secrets Manager otherwise use encrypted parameters\.

**Note**  
Tasks that reference a secret from AWS Secrets Manager or Amazon EC2 Systems Manager Parameter Store require a **Task Execution Role** with a policy that grants the Amazon ECS access to the desired secret and, if applicable, the AWS KMS key used to encrypt and decrypt that secret\. 

**Important**  
Secrets that are referenced in tasks aren't rotated automatically\. If your secret changes, you must force a new deployment or launch a new task to retrieve the latest secret value\. For more information, see the following topics:  
[AWS Secrets Manager: Injecting data as environment variables](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/specifying-sensitive-data-secrets.html#secrets-envvar)
[Amazon EC2 Systems Manager Parameter Store: Injecting data as environment variables](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/specifying-sensitive-data-secrets.html#secrets-logconfig)

### Retrieving data from an encrypted Amazon S3 bucket<a name="security-secrets-management-recommendations-encrypted-s3-buckets"></a>

Because the value of environment variables can inadvertently leak in logs and are revealed when running `docker inspect`, you should store secrets in an encrypted Amazon S3 bucket and use task roles to restrict access to those secrets\. When you do this, your application must be written to read the secret from the Amazon S3 bucket\. For instructions, see [Setting default server\-side encryption behavior for Amazon S3 buckets](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucket-encryption.html)\.

### Mount the secret to a volume using a sidecar container<a name="security-secrets-management-recommendations-mount-secret-volumes"></a>

Because there's an elevated risk of data leakage with environment variables, you should run a sidecar container that reads your secrets from AWS Secrets Manager and write them to a shared volume\. This container can run and exit before the application container by using [Amazon ECS container ordering](https://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_ContainerDependency.html)\. When you do this, the application container subsequently mounts the volume where the secret was written\. Like the Amazon S3 bucket method, your application must be written to read the secret from the shared volume\. Because the volume is scoped to the task, the volume is automatically deleted after the task stops\. For an example of a sidecar container, see the [aws\-secret\-sidecar\-injector](https://github.com/aws-samples/aws-secret-sidecar-injector/blob/master/ecs-task-def/task-def.json) project\.

**Note**  
On Amazon EC2, the volume that the secret is written to can be encrypted with a AWS KMS customer managed key\. On AWS Fargate, volume storage is automatically encrypted using a service managed key\. 

## Additional resources<a name="security-secrets-management-resources"></a>
+ [Passing secrets to containers in an Amazon ECS task](https://aws.amazon.com/premiumsupport/knowledge-center/ecs-data-security-container-task/)
+ [Chamber](https://github.com/segmentio/chamber) is a wrapper for storing secrets in Amazon EC2 Systems Manager Parameter Store
