
# READ ABOUT:
- Athena
- Inspector. How to find CVEs and deviations from benchmarks
- EC2 System Manager

`canonical user ID` is a 64-digit-alphanumeric-value obfuscated form of the AWS account ID.

# STS

## API methods
### `AssumeRole` call. 
You must call this API using existing IAM user credentials. 

**Request** must contain:
- `ARN` of the role that the app should assume.
- `Duration`: optional, how long the temporary security credentials are valid (15 minutes to 1 hour (default)
- `Role session name` use to identify the session in CloudTrail.
- (Optional) `additional policy` to further restrict permissions.
- (Optional) `MFA identifier` and OTP
- (Optional) `ExternalID`.

**Responses** return:
- Access Key:
  - Access Key ID
  - Secret Access Key
- Session Token


### `GetSessionToken` API call.
- The primary reason for calling `GetSessionToken` is that a user needs to perform actions that are allowed only when the user is authenticated with multi-factor authentication (MFA).
- temporary security credentials for an IAM user are valid for a maximum of 12 hours, but you can request a duration as short as 15 minutes or as long as 36 hours. A token for an AWS account root user is restricted to a duration of one hour.
- If `GetSessionToken` is called with the credentials of an IAM user, the temporary security credentials have the same permissions as the IAM user.
- Pro: You can use the credentials to access the AWS Management Console.
- COns: You cannot use the credentials to call IAM or STS APIs but only to call some You can use them to call APIs for other AWS services.

### Other API calls:
- `AssumeRoleWithWebIdentity`, `AssumeRoleWithSAML`

## AssumeRole vs GetSessionToken
| AWS STS API | Who can call | Credential lifetime (min/max/default) | MFA support* | Passed policy support |Restrictions on resulting temporary credentials |
|-|-|-|-|-|-|
`AssumeRole` | IAM user or user with existing temporary security credentials | 15m/1hr/1hr | Yes | Yes | Cannot call GetFederationToken or GetSessionToken. |
`GetSessionToken` | IAM user or root user | IAM user: 15m/36hr/12hr. Root user: 15m/1hr/1hr | Yes | No | Cannot call IAM APIs unless MFA information is included with the request.Cannot call AWS STS APIs except AssumeRole or GetCallerIdentity. Single sign-on (SSO) to console is not allowed, but any user with a password (root or IAM user) can sign into the console.*
|


## Endpoints
STS is a single global (https://sts.amazonaws.com) physically located in us-east-1 i.e. the US East (N. Virginia) region, but you can choose an endpoint closer to you to reduce latency (https://sts.us-west-2.amazonaws.com). If you activate regional STS endpoints you must also turn on CloudTrail logging in those regions to record any API calls. 

## Disabling temporary security credentials 
They are valid until they expire, and they **cannot be revoked**. However, because permissions are evaluated each time an AWS request is made using the credentials, you can achieve the effect of revoking the credentials by changing the permissions for the credentials even after they have been issued.
- Note: You cannot change the permissions for an AWS account root user. Likewise, you cannot change the permissions for the temporary security credentials that were created by calling GetFederationToken or GetSessionToken while signed in as the root user. 

# MFA
## MFA Delete
S3 offers **MFA Delete** for `root` only. You can enable it when you setup versioning. MFA Delete cannot be applied to an IAM user and is managed independently from MFA-protected API access. An IAM user with permission to delete a bucket cannot delete a bucket with Amazon S3 MFA Delete enabled.

## MFA for normal access
This `User-based policy` grants users permission to call the EC2 `StopInstances` and `TerminateInstances` actions only if the user has authenticated using MFA.
```
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "ec2:StopInstances",
      "ec2:TerminateInstances"
    ],
    "Resource": ["*"],
    "Condition": {"Bool": {"aws:MultiFactorAuthPresent": "true"}}
  }]
}
```

### MFA Protection for Resources That Have Resource-based Policies
Imagine that you are in account A and you create an S3 bucket. You want to grant access to this bucket to users who are in several different AWS accounts, but only if those users are authenticated with MFA. Also this is an example of providing cross-account MFA protection without requiring users to assume a role first:
```
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"AWS": ["ACCOUNT-A-ID", "ACCOUNT-B-ID", "ACCOUNT-C-ID"]},
    "Action": [
      "s3:PutObject",
      "s3:DeleteObject"
    ],
    "Resource": ["arn:aws:s3:::ACCOUNT-A-BUCKET-NAME/*"],
    "Condition": {"Bool": {"aws:MultiFactorAuthPresent": "true"}}
  }]
}
```

### User-base policy that denies access to specific APIs without recent valid MFA Authentication 
grants access to the entire Amazon EC2 API, but denies access to StopInstances and TerminateInstances unless the user authenticated with MFA within the last hour. 
- The first statement (containing `"Sid": "AllowAllActionsForEC2"`) allows all Amazon EC2 actions. 
- The second statement (containing `"Sid": "DenyStopAndTerminateWhenMFAIsNotPresent"`) denies the `StopInstances` and `TerminateInstances` actions when the MFA authentication context is missing (meaning MFA was not used). The condition check for `MultiFactorAuthPresent` in the `Deny` statement should not be a `{"Bool":{"aws:MultiFactorAuthPresent":false}}` because that key is not present and cannot be evaluated when MFA is not used. So instead, use the `BoolIfExists` check to see if the key is present before checking the value.
- The third statement in the following example (containing `"Sid": "DenyStopAndTerminateWhenMFAIsOlderThanOneHour"`) contains an additional condition that denies the `StopInstances` and `TerminateInstances` actions when MFA authentication is present but occurred more than one hour prior to the request. The condition `aws:MultiFactorAuthAge` is only present when MFA context is present in the request. So statement 2 covers the case when MFA is not present at all, and statement 3 covers the case when MFA is present and evaluates whether it occurred in the proper time window. Again, when the key is not present, the ...IfExists causes the test to return true, the statement matches, and the user is denied access to those APIs.
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowAllActionsForEC2",
      "Effect": "Allow",
      "Action": "ec2:*",
      "Resource": "*"
    },
    {
      "Sid": "DenyStopAndTerminateWhenMFAIsNotPresent",
      "Effect": "Deny",
      "Action": [
        "ec2:StopInstances",
        "ec2:TerminateInstances"
      ],
      "Resource": "*",
      "Condition": {"BoolIfExists": {"aws:MultiFactorAuthPresent": false}}
    },
    {
      "Sid": "DenyStopAndTerminateWhenMFAIsOlderThanOneHour",
      "Effect": "Deny",
      "Action": [
        "ec2:StopInstances",
        "ec2:TerminateInstances"
      ],
      "Resource": "*",
      "Condition": {"NumericGreaterThanIfExists": {"aws:MultiFactorAuthAge": "3600"}}
    }
  ]
}
```
## MFA for API access
(https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_configure-api-require.html)

- To establish MFA protection for APIs, you add MFA conditions to policies. If a policy doesn't include the condition for MFAs, the policy does not enforce the use of MFA. If a policy includes an MFA condition, a request is denied if users have not been MFA authenticated, or if they provide an invalid MFA device identifier or invalid TOTP.
- MFA protection is available **only with temporary security credentials**, which must be obtained with `AssumeRole` or `GetSessionToken`. **long-term credentials** (IAM user access keys and root user access keys) cannot be used with MFA-protected API access because they don't expire.
- No MFA-protected API for `root` user.
- When the user is authenticated by an external provider (`AssumeRoleWithWebIdentity` and `AssumeRoleWithSAML`) AWS cannot determine whether that provider required MFA. So **federated users** cannot be assigned an MFA device for use with AWS services, so they cannot access AWS resources controlled by MFA. 
- For **cross-account** delegation, if the role's trust policy doesn’t include an MFA condition then there is no MFA protection for the API calls that are made with the role's temporary security credentials. When you allow another AWS account to access resources in your account, even when you require multi-factor authentication, the security of your resources depends on the configuration of the trusted account aka the other account (not yours). Any identity in the trusted account that has permission to create virtual MFA devices can construct an MFA claim to satisfy that part of your role's trust policy. Before you allow another account's access to your AWS resources that require multi-factor authentication, you should ensure that the trusted account's owner follows security best practices and restricts access to sensitive APIs—such as MFA device-management APIs—to specific, trusted identities.

# IAM
## Groups
- User *--* Group
- can't be nested (contain only users not other groups)
- If you **change a group's name** or path: policies attached to the group stay with the group, group retains all its users, unique ID remains the same. But IAM does not automatically update policies that refer to the group as a resource to use the new name.
- If you **delete a group** in the Console, it removes all group members, detaches all attached managed policies, and deletes all inline policies. The command-line doesn't, so you must remove everything before removing the group from the command line.

## Roles
- Does not have standard long-term credentials (password or access keys). When a user assumes a role, temporary security credentials are created dynamically and provided to the user.

**Service-linked role**: predefined by the service and include all the permissions that the service requires to call other AWS services on your behalf. A service might automatically create or delete the role

## Policies
[AWS Docs](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html)

The evaluation logic:
![](https://docs.aws.amazon.com/IAM/latest/UserGuide/images/AccessPolicyLanguage_Evaluation_Flow.diagram.png)
1. By default, all requests are denied.
2. An explicit allow overrides this default.
3. An explicit deny overrides any allows.

The order in which the policies are evaluated has no effect: all policies are evaluated.

## Identity-Based Policies
attach to a principal (or identity), such as an IAM user, role, or group.
Active, "what can I do to X?". The `who` is the user that gets a policy attached to so the `Principal` is not specified in the policy. The permissions that the role grants to the user do not add to the permissions already granted to the user. When a user **switches to a role**, the user **temporarily gives up his or her original permissions** in exchange for those granted by the role. When the user exits the role, then the original user permissions are automatically restored.

- **Managed**. A new version of the managed policy (up to 5) is created whenever you or AWS updates a Managed policy (no versioning for inline policies).
  - AWS Managed
  - Customer managed
- **Inline**. Embedded directly into a single user, group, or role. Useful if you want to maintain a strict one-to-one relationship between a policy and the principal entity that it's applied to. For example, you want to be sure that the permissions in a policy are not inadvertently assigned to a principal entity other than the one they're intended for. If you delete the Principal via the Console, the inline policies are deleted too.

## Resource-based policies (or ACLs)
[Roles vs. Resource-based Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_compare-resource-policies.html).
- To be attached to a resource such as an Amazon S3 bucket.
- Are **inline** policies only and not managed.
- Passive, "who can do what to me?" (I am the resource). The `principal` specified the **who**. For 
- Useful for **cross-account** access. You can attach a policy directly to the resource that you want to share, instead of using a role as a proxy. The resource that you want to share must support resource-based policies. Unlike a user-based policy, a resource-based policy specifies who (in the form of a list of AWS account ID numbers) can access that resource. Advantage over a role: with a resource-based policy, **the user still works in the trusted account and does not have to give up his or her user permissions in place of the role permissions** (as instead has to do if using a proxy role).
- Not all services support resource-based policies: only S3 buckets, Glacier vaults, SNS topics, and SQS queues.
### An S3 bucket policy that allows an IAM user named bob in AWS account 777788889999 to put objects into the bucket called example-bucket:
```
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Principal": {"AWS": "arn:aws:iam::777788889999:user/bob"},
    "Action": [
      "s3:PutObject",
      "s3:PutObjectAcl"
    ],
    "Resource": "arn:aws:s3:::example-bucket/*"
  }
}
```

Trust policy of an IAM role that includes an MFA condition to test for the existence of MFA authentication. With this policy, users from the AWS account specified in the `rincipal`element can assume the role that this policy is attached to, but only if the user is MFA authenticated:
```
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Principal": {"AWS": "ACCOUNT-B-ID"},
    "Action": "sts:AssumeRole",
    "Condition": {"Bool": {"aws:MultiFactorAuthPresent": "true"}}
  }
}
```

# Key-policy
CMK key policy (a special case of resource policy)for EBS
```
{
    "Sid": "Allow for use of this Key",
    "Effect": "Allow",
    "Principal": {
        "AWS": "arn:aws:iam::111122223333:role/UserRole"
    },
    "Action": [
        "kms:GenerateDataKeyWithoutPlaintext",
        "kms:Decrypt"
    ],
    "Resource": "*"
},
{
    "Sid": "Allow for EC2 Use",
    "Effect": "Allow",
    "Principal": {
        "AWS": "arn:aws:iam::111122223333:role/UserRole"
    },
    "Action": [
        "kms:CreateGrant",
        "kms:ListGrants",
        "kms:RevokeGrant"
    ],
    "Resource": "*",
    "Condition": {
        "StringEquals": {
            "kms:ViaService": "ec2.us-west-2.amazonaws.com"
        }
    }
}
```
- The first statement provides a specified IAM principal the ability to generate a data key and decrypt that data key from the CMK when necessary. These two APIs are necessary to encrypt the EBS volume while it’s attached to an Amazon Elastic Compute Cloud (EC2) instance.
- The second statement in this policy provides the specified IAM principal the ability to create, list, and revoke grants for Amazon EC2.  In this case, the condition policy explicitly ensures that only Amazon EC2 can use the grants. Amazon EC2 will use them to re-attach an encrypted EBS volume back to an instance if the volume gets detached due to a planned or unplanned outage

- When you use `NotPrincipal` in the same policy statement as `"Effect: Deny"`, the permissions specified in the policy statement are explicitly denied to all principals except for the ones specified.


### Key-policies vs Grants
- **Key policies** work best for relatively **static assignments** of permissions. To enable more granular permissions management, you can use grants. 
- **Grants** are useful when you want to define scoped-down, **temporary permissions** for other principals to **use your CMK on your behalf** in the absence of a direct API call from you.

### What does this do?
- `"Effect": "Deny"`
- `"NotAction"`
- `"Condition": { "BoolIfExists": { "aws:MultiFactorAuthPresent": "false" } }`
```
        {
            "Sid": "BlockMostAccessUnlessSignedInWithMFA",
            "Effect": "Deny",
            "NotAction": [
                "iam:CreateVirtualMFADevice",
                "iam:DeleteVirtualMFADevice",
                "iam:ListVirtualMFADevices",
                "iam:EnableMFADevice",
                "iam:ResyncMFADevice",
                "iam:ListAccountAliases",
                "iam:ListUsers",
                "iam:ListSSHPublicKeys",
                "iam:ListAccessKeys",
                "iam:ListServiceSpecificCredentials",
                "iam:ListMFADevices",
                "iam:GetAccountSummary",
                "sts:GetSessionToken"
            ],
            "Resource": "*",
            "Condition": {
                "BoolIfExists": {
                    "aws:MultiFactorAuthPresent": "false"
                }
            }
        }
```
The above policy uses a combination of "Deny" and "NotAction" to deny all actions for all other AWS services if the user is not signed-in with MFA. If the user is signed-in with MFA, then the "Condition" test fails and the final "deny" statement has no effect and other permissions granted to the user can take effect. This last statement ensures that when the user is not signed-in with MFA that they can perform only the IAM actions allowed in the earlier statements. The ...IfExists version of the Bool operator ensures that if the aws:MultiFactorAuthPresent key is missing, the condition returns true This means that a user accessing an API with long-term credentials, such as an access key, is denied access to the non-IAM API operations (https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_users-self-manage-mfa-and-creds.html)

## Role policies:
Any role contains two policies:
- **trust policy** Defines who is allowed to assume the role (the trusted entity or principal). You cannot specify a wildcard (\*) as a `Principal`.
- **permission policy** defines what actions and resources the role can use/do.

Example. The policy specifies that the user can only switch to a role in that account if the role name begins with the letters "Test" followed by any other combination of characters.
```
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Action": "sts:AssumeRole",
    "Resource": "arn:aws:iam::ACCOUNT-ID-WITHOUT-HYPHENS:role/Test*"
  }
}
```

## Permissions
An IAM user might be granted access to create a resource, but the user's permissions, even for that resource, are limited to what's been explicitly granted. This means that just because you create a resource, such as an IAM role, you do not automatically have permission to edit or delete that role

## Delegation

You can only delegate permissions equivalent to, or less than, the permissions granted to your account by the resource owning account.

![Resource-based delegation](https://docs.aws.amazon.com/IAM/latest/UserGuide/images/Delegation.diagram.png "Delegation diagram")

1. Account A gives account B full access to account A's S3 bucket by naming account B as a principal in the policy. As a result, account B is authorized to perform any action on account A's bucket, and the account B administrator can delegate access to its users in account B.
2. The account B administrator grants user 1 read-only access to account A's S3 bucket. User 1 can view the objects in account A's bucket. The level of access account B can delegate is equivalent to, or less than, the access the account has. In this case, the full access granted to account B is filtered to read only for user 1.
3. The account B administrator does not give access to user 2. Because users by default do not have any permissions except those that are explicitly granted, user 2 does not have access to account A's Amazon S3 bucket.

## Example Separate Development and Production Accounts
[AWS Docs - Separate Development and Production Accounts](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_common-scenarios_aws-accounts-example.html)
![Separate Development and Production Accounts](https://docs.aws.amazon.com/IAM/latest/UserGuide/images/roles-usingroletodelegate.png "Separate Development and Production Accounts")
1. The admin of *production account*:
- creates the `UpdateAPP` and defines a `trust policy` that specifies the  *development account* as a `Principal`: authorized users from the development account can use the UpdateAPP role. 
- Creates a `permissions policy` for the role that specifies that users of the role have read and write permissions to the bucket named *productionapp*.
- give the account number and name of the role (for AWS console users) or the Amazon Resource Name (ARN) (for AWS CLI, Tools for Windows PowerShell, or AWS API access) of the role with anyone who needs to assume the role. The role ARN might look like `arn:aws:iam::123456789012:role/UpdateAPP`.
2. The admin of the *Development account* grants the developers permission to call STS `AssumeRole` API for the `UpdateAPP` role.

## Providing Access to AWS Accounts Owned by Third Parties
[AWS - Docs: Using an External ID for Third-Party Access](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-user_externalid.html)

You can use roles to delegate access. The third parties must provide A) its own AWS *account ID* and B) a secret *external ID*, both need to be specified in the `trust policy` for the role.
```
"Principal": {"AWS": "Example Corp's AWS Account ID"},
"Condition": {"StringEquals": {"sts:ExternalId": "Unique ID Assigned by Example Corp"}}
```
When Example Corp needs to access your AWS resources, someone from the company calls the AWS `sts:AssumeRole` API. The call includes the ARN of the role to assume and the `ExternalID` parameter that corresponds to your customer ID. In other words, when a role policy includes an `external ID`, anyone who wants to assume the role must be a principal in the role and must include the correct `external ID` which solves the "confused deputy problem". You are an AWS account owner and you have configured a role for a third party that accesses other AWS accounts in addition to yours. You should ask the third party for an external ID that it includes when it assumes your role. Then you check for that external ID in your role's trust policy. Doing so ensures that the external party can assume your role only when it is acting on your behalf.
![](https://docs.aws.amazon.com/IAM/latest/UserGuide/images/confuseddeputymitigation2.png)

## Custom identity broker application to federate user access to AWS
[AWS Docs - identity broker application to federate user access to AWS](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_common-scenarios_federated-users.html)
![identity broker application](https://docs.aws.amazon.com/IAM/latest/UserGuide/images/enterprise-authentication-with-identity-broker-application.diagram.png "Identity Broker Application")

2. The identity broker application is able to verify that employees are authenticated within the existing authentication system.

3. Is doing a `AssumeRole` or `GetFederationToken` API call to STS which returns the credentials.

4. Users are able to get a temporary URL that gives them access to the AWS Management Console (which is referred to as single sign-on).

### Using SAML-Based Federation for API Access to AWS
Dropbox like desktop app that copies data from computer to a backup folder in S3:
![](https://docs.aws.amazon.com/IAM/latest/UserGuide/images/saml-based-federation.diagram.png)

### SAML-enabled single sign-on
Inside your organization's network, you configure your identity store (such as Windows Active Directory) to work with a SAML-based identity provider (IdP) like Windows Active Directory Federation Services, Shibboleth
![](https://docs.aws.amazon.com/IAM/latest/UserGuide/images/saml-based-sso-to-console.diagram.png)

# Enable LDAPS
- https://aws.amazon.com/blogs/security/how-to-enable-ldaps-for-your-aws-microsoft-ad-directory/

## 1) Delegate permissions to CAAdmin
![](https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2017/09/22/VS-Diagram1-0917-a.png)

## 2) Add a subordinate Microsoft Enterprise CA to the domain
![](https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2017/09/22/VS-Diagram2-0917-a.png)
2.a) Create an Amazon EC2 Windows Server to use as a `subordinate CA` and join it to the domain.

2.b) Log in to `RootCA` as `RootAdmin` and upload the `RootCA’s public certificate` and `Certificate Revocation List` to an S3 bucket.

2.c) Log in to `SubordinateCA` as the `CAAdmin`. Download `RootCA’s public certificate` and `CRL` from the S3 bucket and adde them to the local store of `SubordinateCA`. Finally, publish the `RootCA’s public certificate` to the domain.

2.d) Make the `SubordinateCA` a real Microsoft **Enterprise** CA using the AD Certificate Services Configuration tool. This generates a Certificate Request; upload it to S3. (LDAPS with AWS Microsoft AD does not support certificates that are issued by a **Standalone** CA but only by an **Enterprise** CA)
 
2.e) Log in to `RootCA` as `RootAdmin`, download the `Certificate Request` from the S3 and approve it (f). This generated a `subordinateCA.crt`; upload it to S3.

2.g) Log in to `SubordinateCA` as `CAAdmin`, download the `SubordinateCA.crt` from S3 bucket and install it (h).

- Delete everything in S3, shut down the `RootCA`.

## 3) Create a certificate template 
With **Server Authentication** and **Autoenrollment** enabled on `subordinateCA`. Once done, the Domain Controllers can obtain a certificate through autoenrollment to enable LDAPS. This certificate lets the LDAP service on the domain controllers listen for and automatically accept SSL connections from LDAP clients.

## 4) Configure AWS security group rules
For the Domain Controllers to request a certificate from the `subordinateCA`:
4.1) accept incoming traffic directed to the `subordinateCA` from the Domain Controllers
4.2) allow outbound traffic from the Domain Controllers to the `subordinateCA`

## 5) Auto-enabled LDAPS configuration
AWS Microsoft AD Domain Controllers will automatically see the published template and they will request a certificate from the `subordinateCA`. The `subordinateCA` will take up to 180 minutes to issue the certificate to the DCs. Once this happens, the DC will receive the cert, import it and enable LDAPS.

You can test the LDAPS connection to the AWS Microsoft AD directory using the `LDP tool`. The LDP tool comes with the Active Directory Administrative Tools. 
# Cognito
Preferred way to use web identity federation, it acts as an identity broker and does much of the federation work for you. With web identity federation, you don't need to create custom sign-in code or manage your own user identities. Instead, users of your app can sign in using a well-known identity provider (IdP) — Amazon, Facebook, Google, or any other OpenID Connect (OIDC) - compatible IdP, *receive an authentication token, and then exchange that token for temporary security credentials in AWS that map to an IAM role with permissions to use the resources in your AWS account*. Using an IdP helps you keep your AWS account secure, because you don't have to embed and distribute long-term security credentials with your application.

## Example - Game App for phone with Cognito as IdP
[AWS Docs - Cognito as IdP](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_oidc_cognito.html)

User data such as scores and profiles is stored in S3 and DynamoDB.
![Cognito](https://docs.aws.amazon.com/IAM/latest/UserGuide/images/mobile-app-web-identity-federation.diagram.png "Cognito")

# Web Identity Federation APIs for Mobile Apps
The alternative to Cognito is to create an app that manually does an unsigned call to `AssumeRoleWithWebIdentity` API passing the web identity federation token anf the ARN for the IAM role that you created for that IdP. Once the app has obtained the temporary security credentials, it makes signed requests to AWS APIs. The app should cache the temporary security credentials so that you do not have to get new ones each time the app needs to make a request to AWS. By default, the credentials are good for one hour.

## Allow access to resources using Web Identity Federation
- https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_oidc_user-id.html

To write policies that allow exclusive access to resources for individual users, you can match the complete folder name, including the app name and provider name, if you're using that. The following example shows a permission policy that grants access to a bucket in Amazon S3 only if the prefix for the bucket matches the string: `myBucket/Amazon/mynumbersgame/user1`. The example assumes that the user is signed in using Login with Amazon, and that the user is using an app called `mynumbersgame`. The user's unique ID is presented as an attribute called user_id. 
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": ["arn:aws:s3:::myBucket"],
      "Condition": {"StringLike": {"s3:prefix": ["Amazon/mynumbersgame/${www.amazon.com:user_id}/*"]}}
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": [
        "arn:aws:s3:::myBucket/amazon/mynumbersgame/${www.amazon.com:user_id}",
        "arn:aws:s3:::myBucket/amazon/mynumbersgame/${www.amazon.com:user_id}/*"
      ]
    }
  ]
}
```

### Configure permissions in AWS for your federated users
When you create the **trust policy** that indicates who can assume the role, you specify the SAML provider that you created earlier in IAM along with one or more SAML attributes that a user must match to be allowed to assume the role. For example, you can specify that only users whose SAML `eduPersonOrgDN` value is `ExampleOrg` are allowed to sign in. The role wizard automatically adds a condition to test the `saml:aud` attribute to make sure that the role is assumed only for sign-in to the AWS Management Console. 
```
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"Federated": "arn:aws:iam::ACCOUNT-ID-WITHOUT-HYPHENS:saml-provider/ExampleOrgSSOProvider"},
    "Action": "sts:AssumeRoleWithSAML",
    "Condition": {"StringEquals": {
      "saml:edupersonorgdn": "ExampleOrg",
      "saml:aud": "https://signin.aws.amazon.com/saml"
    }}
  }]
}
```

# S3 Access Management
- **User-based**
-  **Resource-based** policies: Bucket policies and Bucket ACLs, and Object ACLs.
- If you create an IAM user, your AWS account is the parent owner. If the IAM user uploads an object, the parent account, to which the user belongs, owns the object.
- A bucket owner (who pays the bill) can explicitly deny access to objects in the bucket even if he does not own the object himself. The bucket owner can also delete any object in the bucket, even if he does not own those objects. A bucket owner can enable other AWS accounts to upload objects. These objects are owned by the accounts that created them. The bucket owner does not own objects that were not created by the bucket owner. Therefore, for the bucket owner to grant access to these objects, the object owner must first grant permission to the bucket owner using an **object ACL**. The bucket owner can then delegate those permissions via a **bucket policy**.
- You can require that your users access your Amazon S3 content by using CloudFront URLs (instead of Amazon S3 URLs). To do this, create a CloudFront origin access identity (OAI), and then change the permissions either on your bucket or on the objects in your bucket.
- **Conditions**: key names are preceded by the prefix s3:. For example, `s3:x-amz-acl`.

## Evaluation Logic
User context > Bucket Context > Object Context
![](https://docs.aws.amazon.com/AmazonS3/latest/dev/images/example50-policy-eval-logic.png)

## User Policy
- You cannot grant anonymous permissions in an IAM user policy, because the policy is attached to a user. 

## Bucket policy
- You must use a bucket policy for **cross-account permissions** to other AWS accounts or users in another account.
- bucket policies are limited to 20 KB in size.

Example - Account A's S3 bucket `mybucket` can be accessed by account B (account number 111122223333)
```
{
  "Version": "2012-10-17",
  "Statement": {
    "Sid": "AccountBAccess1",
    "Effect": "Allow",
    "Principal": {"AWS": "111122223333"},
    "Action": "s3:*",
    "Resource": [
      "arn:aws:s3:::mybucket",
      "arn:aws:s3:::mybucket/*"
    ]
  }
}
```

Example - grants anonymous (i.e. `"Principal": "*"`)read permission on all objects in a bucket:
```
{
    "Version":"2012-10-17",
    "Statement": [
        {
            "Effect":"Allow",
            "Principal": "*",
            "Action":["s3:GetObject"],
            "Resource":["arn:aws:s3:::examplebucket/*"]
        }
    ]
}
```
## ACLs
- ACLs use an Amazon S3–specific **XML schema**.
- Each bucket and object has an ACL associated with it
**Pros**: 
- Useful when a bucket owner allows other AWS accounts to upload objects, permissions to these objects can only be managed using object ACL by the AWS account that owns the object.
- **An object ACL is the only way to manage access to objects not owned by the bucket owner**. A bucket owner cannot grant permissions on objects it does not own. An AWS account that owns the bucket can grant another AWS account permission to upload objects. The bucket owner does not own these objects. The AWS account that created the object must grant permissions using object ACLs.
- **object ACL** is also limited to a maximum of 100 grants
- The **only recommended use case for the bucket ACL** is to grant write permission to the **S3 Log Delivery** group to write access log objects to your bucket .


**Cons**:
- you can grant permissions only to **other AWS accounts** not to users in your account. 
- No conditional permissions, 
- No explicit deny

Example -  a bucket owner as having full control permission:
```
<?xml version="1.0" encoding="UTF-8"?>
<AccessControlPolicy xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
  <Owner>
    <ID>*** Owner-Canonical-User-ID ***</ID>
    <DisplayName>owner-display-name</DisplayName>
  </Owner>
  <AccessControlList>
    <Grant>
      <Grantee xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
               xsi:type="Canonical User">
        <ID>*** Owner-Canonical-User-ID ***</ID>
        <DisplayName>display-name</DisplayName>
      </Grantee>
      <Permission>FULL_CONTROL</Permission>
    </Grant>
  </AccessControlList>
</AccessControlPolicy> 
```

### Explicit deny and overrides
[AWS - Docs](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_policy-examples.html)

Account A writes a bucket policy on account A's S3 bucket that **explicitly denies** account B access to account A's bucket:
```
{
  "Version": "2012-10-17",
  "Statement": {
    "Sid": "AccountBDeny",
    "Effect": "Deny",
    "Principal": {"AWS": "111122223333"},
    "Action": "s3:*",
    "Resource": "arn:aws:s3:::mybucket/*"
  }
}
```
Now even if account B writes an IAM user policy that grants a user in account B access to account A's bucket, the explicit deny applied to account A's S3 bucket propagates to the users in account B and overrides the IAM user policy granting access to the user in account B ==> Users in account B won't be able to access A's S3 bucket

### Example - Bucket Owner Granting Cross-account Permission to Objects It Does Not Own:
![](https://docs.aws.amazon.com/AmazonS3/latest/dev/images/access-policy-ex4.png)
1. Account A administrator user attaches a bucket policy granting Account B conditional permission to upload objects.
2. Account A administrator creates an IAM role, establishing trust with Account C, so users in that account can access Account A. The access policy attached to the role limits what user in Account C can do when the user accesses Account A.
3. Account B administrator uploads an object to the bucket owned by Account A, granting full-control permission to the bucket owner.
4. Account C administrator creates a user and attaches a user policy that allows the user to assume the role.
5. User in Account C first assumes the role, which returns the user temporary security credentials. Using those temporary credentials, the user then accesses objects in the bucket.

# EC2
## Instance profile
[AWS Docs - Using Roles for Applications on Amazon EC2](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html)
- Contains a role and provides the role's temporal credentials so an app running on an EC2 instance can assume the role and use the instance privileges for accessing other AWS resources via these temp credentials, for example to query RDS database. REmove the need to hardcoded long-term creds(i.e. username and pw) into applications.
- Only one role can be assigned to an EC2 instance at a time, and all applications on the instance share the same role and permissions.

### Example - App that needs access to S3:
![](https://docs.aws.amazon.com/IAM/latest/UserGuide/images/roles-usingrole-ec2roleinstance.png)
- The developer runs an application on an EC2 instance that requires access to the S3 bucket named photos. 
- An administrator creates the Get-pics role. The role includes policies that grant read permissions for the bucket and that allow the developer to launch the role with an EC2 instance. 
- When the application runs on the instance, it can use the role's temporary credentials to access the photos bucket. 
- The administrator doesn't have to grant the developer permission to access the photos bucket
- The developer never has to share or manage credentials.

### Example policy that grants a user permission to launch an EC2 instance with a specific role:
The policy grants the user the permission to pass only the Get-pics role. If the user tries to specify a different role when launching an instance, the action fails.
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:RunInstances",
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "iam:PassRole",
      "Resource": "arn:aws:iam::ACCOUNT-ID-WITHOUT-HYPHENS:role/Get-pics"
    }
  ]
}
```

## Permissions for Federated User
![](https://docs.aws.amazon.com/IAM/latest/UserGuide/images/getfederationtoken-permissions.diagram.png)
- The most common way to ensure that the federated user is assigned appropriate permission is to **pass a policy as a parameter** of the `GetFederationToken` API call. 
  - The effective permissions for the federated user consist of only those permissions that are granted in both the IAM user policy **and** the passed policy:
- **Resource-based Policies** provide another mechanism to grant permissions directly to a federated user. You assign permissions directly to a federated user by specifying the ARN of the federated user in the `Principal` element of the resource-based policy.
  - A federated user is granted permissions only when those permissions are explicitly granted to both the IAM user **and** the federated user.

# CloudTrail
AWS does not log the entered user name text when the sign-in failure is caused by an incorrect user name. The user name text is masked by the value `HIDDEN_DUE_TO_SECURITY_REASONS` (accidentally type your password in the user name field?)

# AWS Organizations
service that enables you to group together and centrally manage the AWS accounts that your business owns
## Service Control Policies SCPs
serve as "filters" or "guardrails" that limit what services and actions can be accessed by the IAM users, groups, and roles in those accounts. If an SCP attached to an account denies access to a service, such as S3, then no user in that account can access any S3 API, even if the user has administrator permissions in the account. Even the AWS account root user is denied access to S3 APIs.
