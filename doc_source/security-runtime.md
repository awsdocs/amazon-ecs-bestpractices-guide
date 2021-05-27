# Runtime security<a name="security-runtime"></a>

Runtime security provides active protection for your containers while they're running\. The idea is to detect and prevent malicious activity from occuring your containers\. 

With secure computing \(`seccomp`\) you can prevent a containerized application from making certain syscalls to the underlying host operating system's kernel\. While the Linux operating system has a few hundred system calls, the most of them aren't necessary for running containers\. By restricting which syscalls can be made by a container, you can effectively decrease your application's attack surface\.

To get started with seccomp, you can use `strace` to generate a stack trace to see which system calls your application is making\. You can use a tool such as `syscall2seccomp` to create a seccomp profile from the data gathered from the stack trace\. For more information, see [strace](https://man7.org/linux/man-pages/man1/strace.1.html) and [syscall2seccomp](https://github.com/antitree/syscall2seccomp)\.

Unlike the SELinux security module, seccomp can't isolate containers from each other\. However, it protects the host kernel from unauthorized syscalls\. It functions by intercepting syscalls and only allowing those that have been allow\-listed to pass through\. Docker has a [default](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json) seccomp profile that is suitable for a majority of general purpose workloads\.

**Note**  
It's also possible to create your own profiles for things that require additional privileges\.

AppArmor is a Linux security module similar to seccomp, but it restricts a container's capabilities including accessing parts of the file system\. It can be run in either `enforcement` or `complain` mode\. Because building AppArmor profiles can be challenging, we recommend that you use a tool like [bane](https://github.com/genuinetools/bane)\. For more information about AppArmor, see the official [AppArmor](https://www.apparmor.net/) page\.

**Important**  
AppArmor is only available for Ubuntu and Debian distributions of Linux\. 

## Recommendations<a name="security-runtime-recommendations"></a>

We recommend that you take the folowing actions when setting up your runtime security\.

### Use a third\-party solution for runtime defense<a name="security-runtime-recommendations-3rd-part-runtime-defense"></a>

Use a third\-party solution for runtime defense\. If you're familiar with how Linux security works, create and manage seccomp and AppArmor profiles\. Both are open\-source projects\. Otherwise, consider using a different third\-party service instead\. Most use machine learning to block or alert on suspicious activity\. For a list of available third\-party solutions, see [AWS Marketplace for Containers](https://aws.amazon.com/marketplace/features/containers)\.

### Add or remove Linux capabilities using seccomp policies<a name="security-runtime-recommendations-alter-linux-capabilities"></a>

Use seccomp to have greater control over Linux capabilities and to avoid syscall check errors\. Seccomp works as a syscall filter that revokes the permission to run certain syscalls or to use specific agruments\.