# STS
`AssumeRole` and `GetSessionToken` can also be called without MFA information

# MFA
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
## MFA for API access
(https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_configure-api-require.html)

- To establish MFA protection for APIs, you add MFA conditions to policies. If a policy doesn't include the condition for MFAs, the policy does not enforce the use of MFA. If a policy includes an MFA condition, a request is denied if users have not been MFA authenticated, or if they provide an invalid MFA device identifier or invalid TOTP.
- MFA protection is available **only with temporary security credentials**, which must be obtained with `AssumeRole` or `GetSessionToken`. **long-term credentials** (IAM user access keys and root user access keys) cannot be used with MFA-protected API access because they don't expire.
- No MFA-protected API for `root` user.
- When the user is authenticated by an external provider (`AssumeRoleWithWebIdentity` and `AssumeRoleWithSAML`) AWS cannot determine whether that provider required MFA. So **federated users** cannot be assigned an MFA device for use with AWS services, so they cannot access AWS resources controlled by MFA. 
- For **cross-account** delegation, if the role's trust policy doesn’t include an MFA condition then there is no MFA protection for the API calls that are made with the role's temporary security credentials. When you allow another AWS account to access resources in your account, even when you require multi-factor authentication, the security of your resources depends on the configuration of the trusted account aka the other account (not yours). Any identity in the trusted account that has permission to create virtual MFA devices can construct an MFA claim to satisfy that part of your role's trust policy. Before you allow another account's access to your AWS resources that require multi-factor authentication, you should ensure that the trusted account's owner follows security best practices and restricts access to sensitive APIs—such as MFA device-management APIs—to specific, trusted identities.

# IAM policies
## User-based policy
Active, "what can I do to X?". The `who` is the user that gets a policy attached to so the `Principal` is not specified in the polic.

## Resource-based policy (or ACL)
Passive, "who can do what to me?" (I am the resource). The `principal` specified the **who**.

An S3 bucket policy that allows an IAM user named bob in AWS account 777788889999 to put objects into the bucket called example-bucket:
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
