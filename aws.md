# STS

## API methods
- `AssumeRole`. You must call this API using existing IAM user credentials.
- `GetSessionToken` 

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
??? Duration (1-36 hours)

## Endpoints
STS is global, but you can choose an endpoint closer to you to reduce latency.

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

### Role policies:
Any role contains two policies
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

## Policies 

### User-based policy
Active, "what can I do to X?". The `who` is the user that gets a policy attached to so the `Principal` is not specified in the polic.

The permissions that the role grants to the user do not add to the permissions already granted to the user. When a user **switches to a role**, the user **temporarily gives up his or her original permissions** in exchange for those granted by the role. When the user exits the role, then the original user permissions are automatically restored.

### Resource-based policy (or ACL)
[Roles vs. Resource-based Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_compare-resource-policies.html).
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

# S3
## Bucket Policies vs ACLs
### Bucket policy
Account A's S3 bucket `mybucket` can be accessed by account B (account number 111122223333)
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
### ACLs
**Pros**: 
- Useful when a bucket owner allows other AWS accounts to upload objects, permissions to these objects can only be managed using object ACL by the AWS account that owns the object.

**Cons**:
- you can grant permissions only to **other AWS accounts** not to users in your account. 
- No conditional permissions, 
- No explicit deny

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
