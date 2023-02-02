# FSx for Windows File Server<a name="storage-fsx"></a>

FSx for Windows File Server provides fully managed, highly reliable, and scalable file storage that is accessible over the industry\-standard Server Message Block \(SMB\) protocol\. It's built on Windows Server, delivering a wide range of administrative features such as user quotas, end\-user file restore, and Microsoft Active Directory \(AD\) integration\. It offers single\-AZ and multi\-AZ deployment options, fully managed backups, and encryption of data at rest and in transit\.

Amazon ECS supports the use of FSx for Windows File Server in Amazon ECS Windows task definitions enabling persistent storage as a mount point through SMBv3 protocol using a SMB feature called GlobalMappings\.

To setup the FSx for Windows File Server and Amazon ECS integration, the Windows container instance must be a domain member on an Active Directory Domain Service \(AD DS\), hosted by an AWS Directory Service for Microsoft Active Directory, on\-premises Active Directory or self\-hosted Active Directory on Amazon EC2\. AWS Secrets Manager is used to store sensitive data like the username and password of an Active Directory credential which is used to map the share on the Windows container instance\.

To use FSx for Windows File Server file system volumes for your containers, you must specify the volume and mount point configurations in your task definition\. The following is a snippet of a task definition that uses FSx for Windows File Server for container storage\.

```
{
	"containerDefinitions": [{
		"name": "container-using-fsx",
		"image": "iis:2",
		"entryPoint": [
			"powershell",
			"-command"
		],
		"mountPoints": [{
			"sourceVolume": "myFsxVolume",
			"containerPath": "\\mount\\fsx",
			"readOnly": false
		}]
	}],
	"volumes": [{
		"fsxWindowsFileServerVolumeConfiguration": {
			"fileSystemId": "fs-ID",
			"authorizationConfig": {
				"domain": "ADDOMAIN.local",
				"credentialsParameter": "arn:aws:secretsmanager:us-east-1:111122223333:secret:SecretName"
			},
			"rootDirectory": "share"
		}
	}]
}
```

For more information, see [Amazon FSx for Windows File Server volumes](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/wfsx-volumes.html) in the *Amazon Elastic Container Service Developer Guide*\.

## Security and access controls<a name="storage-fsx-security"></a>

FSx for Windows File Server offers the following access control features that you can use to ensure that the data stored in an FSx for Windows File Server file system is secure and accessible only from applications that need it\.

### Data encryption<a name="storage-fsx-security-encryption"></a>

FSx for Windows File Server supports two forms of encryption for file systems\. They are encryption of data in transit and encryption at rest\. Encryption of data in transit is supported on file shares that are mapped on a container instance that supports SMB protocol 3\.0 or newer\. Encryption of data at rest is automatically enabled when creating an Amazon FSx file system\. Amazon FSx automatically encrypts data in transit using SMB encryption as you access your file system without the need for you to modify your applications\. For more information, see [Data encryption in Amazon FSx](https://docs.aws.amazon.com/fsx/latest/WindowsGuide/encryption.html) in the *Amazon FSx for Windows File Server User Guide*\.

### Folder level access control using Windows ACLs<a name="storage-fsx-security-access"></a>

The Windows Amazon EC2 instance access Amazon FSx file shares using Active Directory credentials\. It uses standard Windows access control lists \(ACLs\) for fine\-grained file\- and folder\-level access control\. You can create multiple credentials, each one for a specific folder within the share which maps to a specific task\.

In the following example, the task has access to the folder `App01` using a credential saved in Secrets Manager\. Its Amazon Resource Name \(ARN\) is `1234`\.

```
"rootDirectory": "\\path\\to\\my\\data\App01",
"credentialsParameter": "arn-1234",
"domain": "corp.fullyqualified.com",
```

In another example, a task has access to the folder `App02` using a credential saved in the Secrets Manager\. Its ARN is `6789`\.

```
"rootDirectory": "\\path\\to\\my\\data\App02",
"credentialsParameter": "arn-6789",
"domain": "corp.fullyqualified.com",
```

## Use cases<a name="storage-fsx-usecase"></a>

Containers aren't designed to persist data\. However, some containerized \.NET applications might require local folders as persistent storage to save application outputs\. FSx for Windows File Server offers a local folder in the container\. This allows for multiple containers to read\-write on the same file system that's backed by a SMB Share\.