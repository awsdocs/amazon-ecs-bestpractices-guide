# Runtime security<a name="security-runtime"></a>

Runtime security provides active protection for your containers while they're running\. The idea is to detect and prevent malicious activity from occurring on your containers\. Runtime security configuration differs between Windows and Linux containers\.

To secure a Microsoft Windows container, see [Secure Windows containers](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/container-security)\.

To secure a Linux container, you can add or drop Linux kernel capabilities using the `linuxParameters` and apply SELinux `labels`, or an AppArmor profile using the `dockerSecurityOptions`, both per container within a task definition\. SELinux or AppArmor have to be configured on the container instance before they can be used\. SELinux and AppArmor are not available in AWS Fargate\. For more information, see [https://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_ContainerDefinition.html#ECS-Type-ContainerDefinition-dockerSecurityOptions](https://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_ContainerDefinition.html#ECS-Type-ContainerDefinition-dockerSecurityOptions) in the Amazon Elastic Container Service API Reference, and [Security configuration](https://docs.docker.com/engine/reference/run/#security-configuration) in the *Docker run reference*\.

AppArmor is a Linux security module that restricts a container's capabilities including accessing parts of the file system\. It can be run in either `enforcement` or `complain` mode\. Because building AppArmor profiles can be challenging, we recommend that you use a tool like [bane](https://github.com/genuinetools/bane)\. For more information about AppArmor, see the official [AppArmor](https://www.apparmor.net/) page\.

**Important**  
AppArmor is only available for Ubuntu and Debian distributions of Linux\. 

## Recommendations<a name="security-runtime-recommendations"></a>

We recommend that you take the following actions when setting up your runtime security\.

### Use a third\-party solution for runtime defense<a name="security-runtime-recommendations-3rd-part-runtime-defense"></a>

Use a third\-party solution for runtime defense\. If you're familiar with how Linux security works, create and manage AppArmor profiles\. Both are open\-source projects\. Otherwise, consider using a different third\-party service instead\. Most use machine learning to block or alert on suspicious activity\. For a list of available third\-party solutions, see [AWS Marketplace for Containers](https://aws.amazon.com/marketplace/features/containers)\.