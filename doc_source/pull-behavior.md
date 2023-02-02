# Container image pull behavior<a name="pull-behavior"></a>

## Container image pull behavior for Fargate launch types<a name="fargate-pull-behavior"></a>

Fargate does not cache images, and therefore the whole image is pulled from the registry when a task runs\. The following are our recommendations for images used for Fargate tasks:
+ Use a larger task size with additional vCPUs\. The larger task size can help reduce the time that is required to extract the image when a task launches\.
+ Use a smaller base image\.
+ Have the repository that stores the image in the same Region as the task\.

## Container image pull behavior for Windows AMIs on Fargate launch types<a name="fargate-windows-behavior"></a>

Fargate windows AMIs cache the most recent base image \(for example, [https://mcr.microsoft.com/windows/servercore:ltsc2019](https://mcr.microsoft.com/windows/servercore:ltsc2019)\) provided by Microsoft\. When a task runs, the only parts of the image that are pulled are the custom parts built on top of the base layers\. On a monthly basis, AWS updates the base layers with the latest Microsoft\-supplied patches\.

**Note**  
If the image is based on an older version of the servercore than what is provided in the AMI, the appropriate layers are pulled at runtime\.

## Container image pull behavior for Amazon EC2 launch types<a name="ec2-pull-behavior"></a>

When the Amazon ECS agent starts a task, it pulls the Docker image from its remote registry, and then caches a local copy\. When you use a new image tag for each release of your application, this behavior is unnecessary\. 

![\[Diagram showing the container image pull behavior.\]](http://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/images/ecs-cache.png)

The following Amazon ECS agent parameter determines the image pull behavior:
+ `ECS_IMAGE_PULL_BEHAVIOR`: default

  For more information about the container agent parameter, see [Container agent configuration](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-agent-config.html) in the *Amazon Elastic Container Service Developer Guide*\.

To speed up deployment, set the Amazon ECS agent parameter to one of the following values: 
+ `ECS_IMAGE_PULL_BEHAVIOR`: `once`

  The image is pulled remotely only if it wasn't pulled by a previous task on the same container instance or if the cached image was removed by the automated image cleanup process\. Otherwise, the cached image on the instance is used\. This ensures that no unnecessary image pulls are attempted\. 
+ `ECS_IMAGE_PULL_BEHAVIOR`: `prefer-cached`

  The image is pulled remotely if there is no cached image\. Otherwise, the cached image on the instance is used\. Automated image cleanup is disabled for the container to ensure that the cached image isn't removed\. 

Setting the parameter to either of the preceding values can save time because the Amazon ECS agent uses the existing downloaded image\. For larger Docker images, the download time might take 10\-20 seconds to pull over the network\.