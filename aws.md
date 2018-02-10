# Enable LDAPS
- https://aws.amazon.com/blogs/security/how-to-enable-ldaps-for-your-aws-microsoft-ad-directory/

## 1) Delegate permissions to CAAdmin
![](https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2017/09/22/VS-Diagram1-0917-a.png)

## 2) Add a subordinate Microsoft Enterprise CA to the domain
![](https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2017/09/22/VS-Diagram2-0917-a.png)
2.1) Create an Amazon EC2 Windows Server to use as a `subordinate CA` and join it to the domain.

2.2) Log in to `RootCA` as `RootAdmin` and upload the `RootCA’s public certificate` and `Certificate Revocation List` to an S3 bucket.

2.3) Log in to `SubordinateCA` as the `CAAdmin`. Download `RootCA’s public certificate` and `CRL` from the S3 bucket and adde them to the local store of `SubordinateCA`. Finally, publish the `RootCA’s public certificate` to the domain.

2.4) Make the `SubordinateCA` a real Microsoft **Enterprise** CA using the AD Certificate Services Configuration tool. This generates a Certificate Request; upload it to S3.
 
2.5) Log in to `RootCA` as `RootAdmin`, download the `Certificate Request` from the S3 and approve it. This generated a `subordinateCA.crt`; upload it to S3.

2.6) Log in to `SubordinateCA` as `CAAdmin`, download the `SubordinateCA.crt` from S3 bucket and install it.

2.7) Delete everything in S3, shut down the `RootCA`.

