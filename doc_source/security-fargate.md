# AWS Fargate security<a name="security-fargate"></a>

We recommend that you take into account the following best practices when you use AWS Fargate\. For additional guidance, see [Security overview of AWS Fargate](https://d1.awsstatic.com/whitepapers/AWS_Fargate_Security_Overview_Whitepaper.pdf)\.

## Use AWS KMS to encrypt ephemeral storage<a name="security-fargate-ephemeral-storage-encryption"></a>

You should have your ephemeral storage encrypted by AWS KMS\. For Amazon ECS tasks that are hosted on AWS Fargate using platform version `1.4.0` or later, each task receives 20 GiB of ephemeral storage\. You can increase the total amount of ephemeral storage, up to a maximum of 200 GiB, by specifying the `ephemeralStorage` parameter in your task definition\. For such tasks that were launched on May 28, 2020 or later, the ephemeral storage is encrypted with an AES\-256 encryption algorithm using an encryption key managed by AWS Fargate\.

For more information, see [Using data volumes in tasks ](https://docs.aws.amazon.com/AmazonECS/latest/userguide/using_data_volumes.html)\.

**Example: Launching an Amazon ECS task on AWS Fargate platform version 1\.4\.0 with ephemeral storage encryption**

The following command will launch an Amazon ECS task on AWS Fargate platform version 1\.4\. Because this task is launched as part of the Amazon ECS cluster, it uses the 20 GiB of ephemeral storage that's automatically encrypted\.

```
aws ecs run-task --cluster clustername \
  --task-definition taskdefinition:version \
  --count 1
  --launch-type "FARGATE" \
  --platform-version 1.4.0 \
  --network-configuration "awsvpcConfiguration={subnets=[subnetid],securityGroups=[securitygroupid]}" \ 
  --region region
```

## SYS\_PTRACE capability for kernel syscall tracing<a name="security-fargate-syscall-tracing"></a>

The default configuration of Linux capabilities that are added or removed from your container are provided by Docker\. For more information about the available capabilities, see [Runtime privilege and Linux capabilities](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) in the **Docker run** documentation\.

Tasks that are launched on AWS Fargate only support adding the `SYS_PTRACE` kernel capability\.

Refer to the tutorial video below that shows how to use this feature through the Sysdig [Falco](https://github.com/falcosecurity/falco) project\.

[![AWS Videos](http://img.youtube.com/vi/https://www.youtube.com/embed/OYGKjmFeLqI/0.jpg)](http://www.youtube.com/watch?v=https://www.youtube.com/embed/OYGKjmFeLqI)

The code discussed in the previous video can be found on GitHub [here](https://github.com/paavan98pm/ecs-fargate-pv1.4-falco)\.