# IAM: Identity and Access Management

- When accessing AWS, the root account should **never** be used. Users must be created with the proper permissions. IAM is central to AWS
- **Users**: A physical person
- **Groups**: Functions (admin, devops) Teams (engineering, design) which contain a group of users
- **Roles**: Internal usage within AWS resources
    - **Cross Account Roles**: roles used to assumed by another AWS account in order to have access to some resources in our account
- **Policies (JSON documents)**: Defines what each of the above can and cannot do. **Note**: IAM has predefined managed policies
    - There are 3 types of policies:
        - AWS Managed
        - Customer Managed
        - Inline Policies
- **Resource Based Policies**: policies attached to AWS services such as S3, SQS

## IAM Roles vs Resource Based Policies

- When we assume a role (user, application or service), we give up our original permission and take the permission assigned to the role
- When using a resource based policy, principal does not have to give up any permissions
- Example: user in account A needs to scan a DynamoDB table in account A and dump it in an S3 bucket in account B. In this case if we assume a role in account B, we wont be able to scan the table in account A
  
## Best practices

- One IAM User per person **ONLY**
- One IAM Role per Application
- IAM credentials should **NEVER** be shared
- Never write IAM credentials in your code. **EVER**
- Never use the ROOT account except for initial setup
- It's best to give users the minimal amount of permissions to perform their job

## Cross-Account permission evaluation

- In a single AWS account, an explicit `Allow` in either an identity-based policy or a Resource-based policy is usually enough to grant access.
- However, in cross-account scenarios, both the identity policy (in Account A) and the resource policy (in account B) must explicitly allow the action.
- Example: if a user in Account A is trying to read an S3 bucket in Account B, and the bucket policy allows it but the user's IAM policy lacks the `s3:GetObject` permission, the request is implicitly denied.
- KMS evaluation logic -- IAM policies alone are never enough to grant access to a KMS key. The KMS key policy must explicitly allow the IAM principal.
- Permission boundaries and delegated administration:
    * If you want developers to create IAM roles for their Lambda functions, you must ensure they cannot create an Admin role and assume it (privilege escalation). The solution is to grant developers the `iam:CreateRole` permission but attach a condition key (`iam:PermissionsBoundary`) that forces them to attach a highly restrictive permission boundary to any new role they create.
- Data perimeters
    * Securing corporate networks and preventing data exfiltration using VPC endpoints (e.g. a malicious insider attempt to upload sensitive company data to their own personal S3 bucket through corporate VPC endpoint)
    * How? apply a VPC Endpoint policy restricting access using the `aws:PrincipalOrgID` (ensuring only identities from your organization can use the endpoint) and `aws:ResourceOrgID` (ensuring the endpoint can only communicate with buckets owned by your organization) condition keys.
- S3 object ownership & cross-account uploads
    * When an IAM role from account A uploads an object to an S3 bucket in Account B
    * Solution? enable S3 Object Ownership (`BucketOwnerEnforced`) on Account B's bucket. This setting disables ACLs entirely and automatically grants Account B full control over all objects uploaded by external accounts.
- `iam:PassRole` vs `sts:AssumeRole`
    * `sts:AssumeRole` --> human/compute resource actively calls to obrtain temporary credentials
    * `iam:PassRole` --> an administrative permission required to assign an IAM role to an AWS service.
