# Disk encryption with a custom KMS Key<a name="tutorials_04_encrypted_kms_fs"></a>

AWS ParallelCluster supports the configuration options `ebs_kms_key_id` and `fsx_kms_key_id`\. These options allow you to provide a custom AWS KMS key for Amazon EBS Disk encryption or FSx for Lustre\. To use them, you specify an `ec2_iam_role`\.

In order for the cluster to create, the AWS KMS key must know the name of the cluster's role\. This prevents you from using the role created on cluster create, requiring a custom `ec2_iam_role`\.

**Prerequisites**
+ AWS ParallelCluster [is installed](install.md)\.
+ The AWS CLI [is installed and configured\.](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
+ You have an [EC2 key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)\.
+ You have an IAM role with the [permissions](iam.md#example-parallelcluser-policies) required to run the [`pcluster`](pcluster.md) CLI\.

## Creating the role<a name="creating-the-role"></a>

First you create a policy:

1. Go to the IAM Console: [https://console\.aws\.amazon\.com/iam/home](https://console.aws.amazon.com/iam/home)\.

1. Under **Policies**, **Create policy**, click the **JSON** tab\.

1. As the policy's body, paste in the [Instance Policy](iam.md)\. Make sure to replace all occurrences of `<AWS ACCOUNT ID>` and `<REGION>`\.

1. Name the policy `ParallelClusterInstancePolicy`, and then click **Create Policy**\.

Next create a role:

1. Under **Roles**, create a role\.

1. Click `EC2` as the trusted entity\.

1. Under **Permissions**, search for the `ParallelClusterInstancePolicy` role that you just created, and attach it\.

1. Name the role `ParallelClusterInstanceRole`, and then click **Create Role**\.

## Give your key permissions<a name="give-your-key-permissions"></a>

In the AWS KMS Console > **Customer managed keys** > click your key's **Alias** or **Key ID**\.

Click the **Add** button in the **Key users** box, underneath the **Key policy** tab, and search for the *ParallelClusterInstanceRole* you just created\. Attach it\.

## Creating the cluster<a name="creating-the-cluster"></a>

Now create a cluster\. The following is an example of a cluster with encrypted `Raid 0` drives:

```
[cluster default]
...
raid_settings = rs
ec2_iam_role = ParallelClusterInstanceRole

[raid rs]
shared_dir = raid
raid_type = 0
num_of_raid_volumes = 2
volume_size = 100
encrypted = true
ebs_kms_key_id = xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

The following is an example with the FSx for Lustre file system:

```
[cluster default]
...
fsx_settings = fs
ec2_iam_role = ParallelClusterInstanceRole

[fsx fs]
shared_dir = /fsx
storage_capacity = 3600
imported_file_chunk_size = 1024
export_path = s3://bucket/folder
import_path = s3://bucket
weekly_maintenance_start_time = 1:00:00
fsx_kms_key_id = xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

Similar configurations apply to Amazon EBS and Amazon FSx based file systems\.