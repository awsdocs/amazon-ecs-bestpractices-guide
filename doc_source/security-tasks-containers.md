# Task and container security<a name="security-tasks-containers"></a>

You should consider the container image as your first line of defense against an attack\. An insecure, poorly constructed image can allow an attacker to escape the bounds of the container and gain access to the host\. You should do the following to mitigate the risk of this happening\.

## Recommendations<a name="security-tasks-containers-recommendations"></a>

We recommend that you do the following when setting up your tasks and containers\.

### Create minimal or use distroless images<a name="security-tasks-containers-recommendations-images"></a>

Start by removing all extraneous binaries from the container image\. If you’re using an unfamiliar image from Amazon ECR Public Gallery, inspect the image to refer to the contents of each of the container's layers\. You can use an application such as [Dive](https://github.com/wagoodman/dive) to do this\.

Alternatively, you can use **distroless** images that only include your application and its runtime dependencies\. They don't contain package managers or shells\. Distroless images improve the "signal to noise of scanners and reduces the burden of establishing provenance to just what you need\." For more information, see the GitHub documentation on [distroless](https://github.com/GoogleContainerTools/distroless)\.

Docker has a mechanism for creating images from a reserved, minimal image known as **scratch**\. Formore information, see [Creating a simple parent image using **scratch**](https://docs.docker.com/develop/develop-images/baseimages/#create-a-simple-parent-image-using-scratch) in the Docker documentation\. With langages like Go, you can create a static linked binary and reference it in your Dockerfile\. The following example shows how you can accomplish this\.

```
############################
# STEP 1 build executable binary
############################
FROM golang:alpine AS builder
# Install git.
# Git is required for fetching the dependencies.
RUN apk update && apk add --no-cache git
WORKDIR $GOPATH/src/mypackage/myapp/
COPY . .
# Fetch dependencies.
# Using go get.
RUN go get -d -v
# Build the binary.
RUN go build -o /go/bin/hello
############################
# STEP 2 build a small image
############################
FROM scratch
# Copy our static executable.
COPY --from=builder /go/bin/hello /go/bin/hello
# Run the hello binary.
ENTRYPOINT ["/go/bin/hello"]
This creates a container image that consists of your application and nothing else, making it extremely secure.
```

The previous example is also an example of a multi\-stage build\. These types of builds are attractive from a security standpoint because you can use them to minimize the size of the final image pushed to your container registry\. Container images devoid of build tools and other extraneous binaries improves your security posture by reducing the attack surface of the image\. For more information about multi\-stage builds, see [creating multi\-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/)\.

### Scan your images for vulnerabilities<a name="security-tasks-containers-recommendations-vulnerability-scanning"></a>

Similar to their virtual machine counterparts, container images can contain binaries and application libraries with vulnerabilities or develop vulnerabilities over time\. The best way to safeguard against exploits is by regularly scanning your images with an image scanner\. Images that are stored in Amazon ECR can be scanned on push or on\-demand \(once every 24 hours\)\. Amazon ECR currently uses [Clair](https://github.com/quay/clair), an open\-source image scanning solution\. After an image is scanned, the results are logged to the Amazon ECR event stream in Amazon EventBridge\. You can also see the results of a scan from within the Amazon ECR console or by calling the [DescribeImageScanFindings](https://docs.aws.amazon.com/AmazonECR/latest/APIReference/API_DescribeImageScanFindings.html) API\. Images with a `HIGH` or `CRITICAL` vulnerability should be deleted or rebuilt\. If an image that has been deployed develops a vulnerability, it should be replaced as soon as possible\.

[Docker Desktop Edge version 2\.3\.6\.0](https://www.docker.com/products/docker-desktop) or later can [scan](https://docs.docker.com/engine/scan/) local images\. The scans are powered by [Snyk](snyk.io), an application security service\. When vulnerabilities are discovered, Snyk identifies the layers and dependencies with the vulnerability in the Dockerfile\. It also recommends safe alternatives like using a slimmer base image with fewer vulnerabilities or upgrading a particular package to a newer version\. By using Docker scan, developers can resolve potential security issues before pushing their images to the registry\.
+ [Automating image compliance using Amazon ECR and AWS Security Hub](http://aws.amazon.com/blogs/containers/automating-image-compliance-for-amazon-eks-using-amazon-elastic-container-registry-and-aws-security-hub/) explains how to surface vulnerability information from Amazon ECR in AWS Security Hub and automate remediation by blocking access to vulnerable images\.

### Remove special permissions from your images<a name="security-tasks-containers-recommendations-defang-images"></a>

The access rights flags `setuid` and `setgid` allow running an executable with the permissions of the owner or group of the executable\. Remove all binaries with these access rights from your image as these binaries can be used to escalate privileges\. Consider removing all shells and utilities like `nc` and `curl` that can be used for malicious purposes\. You can find the files with `setuid` and `setgid` access rights by using the following command\.

```
find / -perm /6000 -type f -exec ls -ld {} \;
```

To remove these special permissions from these files, add the following directive to your container image\.

```
RUN find / -xdev -perm /6000 -type f -exec chmod a-s {} \; || true
```

### Create a set of curated images<a name="security-tasks-containers-recommendations-curated-images"></a>

Rather than allowing developers to create their own images, create a set of vetted images for the different application stacks in your organization\. By doing so, developers can forego learning how to compose Dockerfiles and concentrate on writing code\. As changes are merged into your codebase, a CI/CD pipeline can automatically compile the asset and then store it in an artifact repository\. And, last, copy the artifact into the appropriate image before pushing it to a Docker registry such as Amazon ECR\. At the very least you should create a set of base images hat developers can create their own Dockerfiles from\. You should avoid pulling images from Docker Hub\. You don't always know what is in the image and about a fifth of the top 1000 images have vulnerabilties\. A list of those images and their vulnerabilities can be found at [https://vulnerablecontainers\.org/](https://vulnerablecontainers.org/)\.

### Scan application packages and libraries for vulnerabilities<a name="security-tasks-containers-recommendations-vulnerability-scanning"></a>

Use of open source libraries is now common\. As with operating systems and OS packages, these libraries can have vulnerabilities \. As part of the development lifecycle these libraries should be scanned and updated when critical vulnerabilities are found\.

Docker Desktop performs local scans using Snyk\. It can also be used to find vulnerabilities and potential licensing issues in open source libraries\. It can be integrated directly into developer workflows giving you the ability to mitigate risks posed by open source libraries\. For more information, see the following topics:
+ [Open Source Application Security Tools](https://owasp.org/www-community/Free_for_Open_Source_Application_Security_Tools) includes a list of tools for detecting vulnerabilities in applications\. 
+ [Docker scanning cheatsheet](https://snyk.io/blog/docker-security-scanning-cheatsheet-2021/) 

### Perform static code analysis<a name="security-tasks-containers-recommendations-static-code-analysis"></a>

You should perform static code analysis before building a container image\. It's performed against your source code and is used to identify coding errors and code that could be exploited by a malicious actor, such as fault injections\. [SonarQube](https://www.sonarqube.org/features/security/) is a popular option for static application security testing \(SAST\), with support for a variety of different programming languages\.

### Run containers as a non\-root user<a name="security-tasks-containers-recommendations-run-non-root-users"></a>

You should run containers as a non\-root user\. By default, containers run as the `root` user unless the `USER` directive is included in your Dockerfile\. The default Linux capabilities that are assigned by Docker restrict the actions that can be run as `root`, but only marginally\. For example, a container running as `root` is still not allowed to access devices\.

As part of your CI/CD pipeline you should lint Dockerfiles to look for the `USER` directive and fail the build if it's missing\. For more information, see the following topics:
+ [Dockerfile\-lint](https://github.com/projectatomic/dockerfile_lint) is an open\-source tool from RedHat that can be used to check if the file conforms to best practices\. 
+ [Hadolint](https://github.com/hadolint/hadolint) is another tool for building Docker images that conform to best practices\. 

### Use a read\-only root file system<a name="security-tasks-containers-recommendations-read-only-file-system"></a>

You should use a read\-only root file system\. A container's root file system is writable by default\. When you configure a container with a `RO` \(read\-only\) root file system it forces you to explicitly define where data can be persisted\. This reduces your attack surface because the container's file system can't be written to unless permissions are specifically granted\.

**Note**  
Having a read\-only root file system can cause issues with certain OS packages that expect to be able to write to the filesystem\. If you're planning to use read\-only root file systems, thoroughly test beforehand\. 

### Configure tasks with CPU and Memory limits \(Amazon EC2\)<a name="security-tasks-containers-recommendations-configure-limits"></a>

You should configure tasks with CPU and memory limits to minimize the following risk\. A task's resource limits set an upper bound for the amount of CPU and memory that can be reserved by all the containers within a task\. If no limits are set, tasks have access to the host's CPU and memory\. This can cause issues where tasks deployed on a shared host can starve other tasks of system resources\.

**Note**  
Amazon ECS on AWS Fargate tasks require you to specify CPU and memory limits because it uses these values for billing purposes\. One task hogging all of the system resources isn't an issue for Amazon ECS Fargate because each task is run on its own dedicated instance\. If you don't specify a memory limit, Amazon ECS allocates a minimum of 4MB to each container\. Similarly, if no CPU limit is set for the task, the Amazon ECS container agent assigns it a minimum of 2 CPUs\. 

### Use immutable tags with Amazon ECR<a name="security-tasks-containers-recommendations-immutable-ecr-tags"></a>

With Amazon ECR, you can and should use configure images with immutable tags\. This prevents pushing an altered or updated version of an image to your image repository with an identical tag\. This protects against an attacker pushing a compromised version of an image over your image with the same tag\. By using immutable tags, you effectively force yourself to push a new image with a different tag for each change\. 

### Avoid running containers as privileged \(Amazon EC2\)<a name="security-tasks-containers-recommendations-avoid-privileged-containers"></a>

You should avoid running containers as privileged\. For background, containers run as `privileged` are run with extended privileges on the host\. This means the container inherits all of the Linux capabilities assigned to `root` on the host\. It's use should be severely restricted or forbidden\. We advise setting the Amazon ECS container agent environment variable `ECS_DISABLE_PRIVILEGED` to `true` to prevent containers from running as `privileged` on particular hosts if `privileged` isn't needed\. Alternatively you can use AWS Lambda to scan your task definitions for the use of the `privileged` parameter\.

**Note**  
Running a container as `privileged` isn't supported on Amazon ECS on AWS Fargate\.

### Remove unnecessary Linux capabilities from the container<a name="security-tasks-containers-recommendations-remove-linux-capabilities"></a>

The following is a list of the default Linux capabilities assigned to Docker containers\. For more information about each capability, see [Overview of Linux Capabilities](http://man7.org/linux/man-pages/man7/capabilities.7.html)\.

```
CAP_CHOWN, CAP_DAC_OVERRIDE, CAP_FOWNER, CAP_FSETID, CAP_KILL,
CAP_SETGID, CAP_SETUID, CAP_SETPCAP, CAP_NET_BIND_SERVICE, 
CAP_NET_RAW, CAP_SYS_CHROOT, CAP_MKNOD, CAP_AUDIT_WRITE, 
CAP_SETFCAP
```

If a container doesn't require all of the Docker kernal capabilties listed above, consider dropping them from the container\. For more information about each Docker kernal capability, see [KernalCapabilities](https://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_KernelCapabilities.html)\. You can find out which capabilities are in use by doing the following:
+ Install the OS package [libcap\-ng](https://people.redhat.com/sgrubb/libcap-ng/) and run the `pscap` utility to list the capabilities that each process is using\.
+ You can also use [capsh](https://www.man7.org/linux/man-pages/man1/capsh.1.html) to decipher which capabilities a process is using\. 
+ Refer to [Linux Capabilities 101](https://linux-audit.com/linux-capabilities-101/) for more information\. 

### Use a customer managed key \(CMK\) to encrypt images pushed to Amazon ECR<a name="security-tasks-containers-recommendations-cmk-encryption"></a>

You should use a customer managed key \(CMK\) to encrupt images that are pushed to Amazon ECR\. Images that are pushed to Amazon ECR are automatically encrypted at rest with a AWS Key Management Service \(AWS KMS\) managed key\. If you would rather use your own key, Amazon ECR now supports AWS KMS encryption with customer managed keys \(CMK\)\. Before enabling server side encryption with a CMK, review the Considerations listed in the documentation on [encryption at rest](https://docs.aws.amazon.com/AmazonECR/latest/userguide/encryption-at-rest.html)\.