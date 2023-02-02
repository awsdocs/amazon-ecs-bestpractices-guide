# Compliance<a name="security-compliance"></a>

Your compliance responsibility when using Amazon ECS is determined by the sensitivity of your data, and the compliance objectives of your company, and applicable laws and regulations\. 

AWS provides the following resources to help with compliance:
+ [Security and compliance quick start guides](http://aws.amazon.com/quickstart/?awsf.quickstart-homepage-filter=categories%23security-identity-compliance): These deployment guides discuss architectural considerations and provide steps for deploying security and compliance\-focused baseline environments on AWS\.
+ [Architecting for HIPAA Security and Compliance Whitepaper](https://docs.aws.amazon.com/whitepapers/latest/architecting-hipaa-security-and-compliance-on-aws/architecting-hipaa-security-and-compliance-on-aws.html): This whitepaper describes how companies can use AWS to create HIPAA\-compliant applications\.
+ [AWS Services in Scope by Compliance Program](http://aws.amazon.com/compliance/services-in-scope/): This list contains the AWS services in scope of specific compliance programs\. For more information, see [AWS Compliance Programs](http://aws.amazon.com/compliance/programs/)\.

## Payment Card Industry Data Security Standards \(PCI DSS\)<a name="security-compliance-pci-dss"></a>

It's important that you understand the complete flow of cardholder data \(CHD\) within the environment when adhering to PCI DSS\. The CHD flow determines the applicability of the PCI DSS, defines the boundaries and components of a cardholder data environment \(CDE\), and therefore the scope of a PCI DSS assessment\. Accurate determination of the PCI DSS scope is key to defining the security posture and ultimately a successful assessment\. Customers must have a procedure for scope determination that assures its completeness and detects changes or deviations from the scope\.

The temporary nature of containerized applications provides additional complexities when auditing configurations\. As a result, customers need to maintain an awareness of all container configuration parameters to ensure compliance requirements are addressed throughout all phases of a container lifecycle\.

For additional information on achieving PCI DSS compliance on Amazon ECS, refer to the following whitepapers\.
+ [Architecting on Amazon ECS for PCI DSS compliance](                         https://d1.awsstatic.com/whitepapers/compliance/architecting-on-amazon-ecs-for-pci-dss-compliance.pdf)
+ [Architecting for PCI DSS Scoping and Segmentation on AWS](https://d1.awsstatic.com/whitepapers/pci-dss-scoping-on-aws.pdf)

## HIPAA \(U\.S\. Health Insurance Portability and Accountability Act\)<a name="security-compliance-hipaa"></a>

Using Amazon ECS with workloads that process protected health information \(PHI\) requires no additional configuration\. Amazon ECS acts as an orchestration service that coordinates the launch of containers on Amazon EC2\. It doesn't operate with or upon data within the workload being orchestrated\. Consistent with HIPAA regulations and the AWS Business Associate Addendum, PHI should be encrypted in transit and at\-rest when accessed by containers launched with Amazon ECS\.

Various mechanisms for encrypting at\-rest are available with each AWS storage option, such as Amazon S3, Amazon EBS, and AWS KMS\. You may deploy an overlay network \(such as VNS3 or Weave Net\) to ensure complete encryption of PHI transferred between containers or to provide a redundant layer of encryption\. Complete logging should also be enabled and all container logs should be directed to Amazon CloudWatch\. For more information, see [Architecting for HIPAA Security and Compliance](https://d0.awsstatic.com/whitepapers/compliance/AWS_HIPAA_Compliance_Whitepaper.pdf)\.

## Recommendations<a name="security-compliance-recommendations"></a>

You should engage the compliance program owners within your business early and use the [AWS shared responsibility model](https://aws.amazon.com/compliance/shared-responsibility-model/) to identify compliance control ownership for success with the relevant compliance programs\.