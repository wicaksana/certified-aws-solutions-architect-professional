# AWS Organizations

- Standard AWS account: it is an account which is not in an AWS Organization
- We create an AWS Organization from a standard AWS account
- The organization is not created in this account, we just use the account to create the organization. The standard account then becomes the **Management Account** (used to be called *Master Account*)
- Using the Management Account we can invite other accounts into the organization
- When a standard account joins an organization, it will change to **Member Account** of that organization
- Organizations have 1 Management Account and 0 or more Member Accounts
- We can create a structure of AWS accounts in an organization. We can group accounts by things such as business units, function or development stage, etc.
- This structure is hierarchical, it is an inverted tree
- At the top of this tree is the root container of the organization (just a container within the organization, NOT to be confused with the root user)
- This root container can contain other containers, this containers are known as **Organizational Units (OU)**
- OUs can contains accounts (Management/Member accounts) or other OUs

## Consolidated Billing

- It is an important feature of AWS Organizations
- The individual billing method of each account from the organization is removed, the member accounts pass their billing through the Management Account (**Payer Account**)
- Using consolidated billing we get a single monthly bill. This covers the Management Account and all the Member Accounts of the Organization
- When using organization reservation benefits and discounts are pooled, meaning the organization can benefit as a whole for the spending of each AWS account within the org

## Best Practices

- Centrally manage user access accross multiple AWS accounts using AWS IAM Identity Center rather than manually pooling IAM users in a single central account:
    * Eliminate standard IAM users & their long-term access key entirely
    * IAM Identity Center integrates natively with AWS Organizations to centrally access control seamlessly:
        - External federation (idP)
        - Permission sets -- define reusable access templates (permission Sets) instead of manually writing and provisioning cross-account IAM roles
        - Temporary credentials -- users authenticate once via SSO and are granted temporary, short-lived session credentials
     
- For reference, the legacy approach (without using IAM Identity Center) is the manual Hub and Spoke identity pattern:
    * single user pool: create all IAM users in one dedicated "security" or "identity" AWS account. These users are granted zero direct permissions to create or modify infrastructure in this account.
    * Cross-account trust: you create IAM roles in the destination "Resource" accounts (e.g. Development, Production) and configure their trust policies to allow the Identity account's users to assume them.
    * Assume Role (`sts:AssumeRole`) -- users log into the identity account and explicitly assume the target role to obtain temporary credentials to work in the Resource account.
- Security guardrails for centralized IAM roles:
    * enforce MFA on role assumption
    * apply the principle of least privilege
    * use SCPs to establish maximum permission guardrails (SCPs apply to all IAM roles and cannot be overriden by local account permissions)
    * Set strict session durations -- restrict the maximum session duration for assumed roles.
    * Require external IDs for 3Ps 

## `OrganizationAccountAccessRole`

- This is a default IAM role automatically created by AWS Organizations when you provision a new member account within your organization.
- This role has to be created manually in the member account if the account was invited into the organization.
- Upon creation, the role is automatically granted the AWS-managed `AdministratorAccess` policy, giving it complete control over all resources in the new account.
- The role's trust relationship is hardcoded to trust the Organization's management account --> only identities residing in the Management account can attempt to use it.
- To use this role, a user or service in the Management account must be granted the `sts:AssumeRole` permission to obtain temporary administrative credentials for the member account.
- Use cases:
   * Account bootstrapping -- eliminates the need to log in with the new account's root email and password. Usually being used to programmatically deploy security baselines, establish networking, or integrate the account with IAM Identity Center immediately after creation.
   * emergency break-glass access -- if standard identity federation or SSO fails, this role provides a guaranteed administrative entry point from the Management account.
- Best practices:
   * Strict access control -- only your most privileged admins or automated account-vending pipelines should have an IAM policy allowing `sts:AssumeRole` on `arn:aws:iam::*:role/OrganizationAccountAccessRole`
   * Audit & monitor: ensure AWS CloudTrail is active and monitor logs for `AssumeRole` API calls targeting this role.

# Service Control Policies (SCP)

- They are a feature of AWS Organizations used to restrict AWS accounts
- They are JSON documents
- They can be attached to the root of the organization, to one or more OUs or to individual AWS accounts
- SCPs inherit down through the organization tree
- The Management Account is special: even if it has SCPs attached (directly or through an OU) it wont be affected by the SCP
- SCPs are account permission boundaries:
    - They limit what the account (including the root user of the account) can do
    - We can never restrict a root user from an account, but we can restrict the account itself, hence these restrictions will apply to the root user as well
- **SCPs don't grant any permissions!** This are just a boundary to limit what is and is not allowed in an account
- SCPs can be used in two ways:
    - Deny list (default): allow by default and block access to certain services
        - `FullAWSAccess`: policy applied by default to the org an all OUs when we enable SCPs. This policy means tha by default nothing is restricted
        - SCPs don't grant permissions, but when they are enabled, there is a default deny for everything. This is why the `FullAWSAccess` policy is needed
        - SCP priority rules:
            1. Explicit Deny
            2. Allow
            3. Default (implicit) deny
        - Benefits of deny lists is that as AWS is extends the list of service offerings, new services will be available for accounts (low admin overhead)
    - Allow list: block by default and allow certain services
        - To implement allow lists:
            1. Remove the `FullAWSAccess` policy
            2. Add any services which should be allowed in a new policy
        - Allow lists are more secure, but they require more admin overhead
