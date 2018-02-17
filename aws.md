# IAM policies
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
## What does this do?
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
