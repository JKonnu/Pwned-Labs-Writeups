![Intro to AWS IAM Enumeration](./Screenshots/pwnedlabs.jpg)
# Intro to AWS IAM Enumeration  [Pwned Labs](https://pwnedlabs.io/)
<br>

## Objective(s):
- Evaluate IAM security posture of Huge Logistics.
- Obtain the flag.
- Retrieve Secrets Manager secret.
<br>

## Write-up
An Access Key ID as well as a Secret Access Key were provided for enumeration of Huge Logistics infrastructure. The provided credentials were configured for authentication in the AWS CLI using the following command:

Note: AWS Access Keys and Secret Access Keys are used to authenticate requests made to AWS services through the AWS CLI. These credentials are generated for IAM users within the AWS Console.
```bash
aws configure
```
![Snip](./Screenshots/image1.png)
<br>
<br>

The credentials were entered when prompted. Once configured, the validity of the credentials was verified using the AWS equivalent of the Linux whoami command:
```bash
aws sts get-caller-identity
```
![Snip](./Screenshots/image2.png)
<br>
<br>

From the command output, the authenticated IAM user was identified successfully. This confirms that the credentials are valid and can be used for further enumeration of the AWS environment.

Tip: sts get-caller-identity is one of the first commands that should be executed after obtaining AWS credentials as it quickly confirms authentication and identifies the current IAM principal.

Further enumeration was then conducted against the identified IAM user:
```bash
aws iam list-user-policies --user-name <user>
```
![Snip](./Screenshots/image3.png)
<br>
<br>
  
From the command output we can see that the user has an inline policy attached related to S3 permissions. Inline policies are directly embedded into a user, group, or role and should always be enumerated carefully as they may contain overly permissive access or privilege escalation paths.

Additional information regarding the discovered policy was then obtained:
```bash
aws iam get-user-policy --user-name <username> --policy-name <policy_name>
```
![Snip](./Screenshots/image4.png)
<br>
<br>

We can see that we have ListBucket and GetObject permissions for the hl-dev-artifacts bucket. For the time being will conduct further enumeration by listing attached policies and come back to the S3 permissions later:
```bash
aws iam list-attached-user-policies --user-name <username>
```
![Snip](./Screenshots/image5.png)
<br>
<br>

We can see from the command output that additional managed policies are attached to the user. Managed policies should always be reviewed thoroughly as they commonly contain broader permissions than inline policies.

The discovered managed policies were then enumerated further by obtaining their policy versions:
```bash
aws iam list-policy-versions --policy-arn <policy_arn>
```
![Snip](./Screenshots/image6.png)
<br>
<br>

After identifying the available policy versions, further enumeration was conducted on the individual versions themselves:
```bash
aws iam get-policy-version --policy-arn <policy_arn> --version-id <version>
```
![Snip](./Screenshots/image7.png)
<br>
<br>

Now we can continue enumeration by checking out the dev01 policy we discovered as well:
```bash
aws iam list-policy-versions --policy-arn <policy_arn>
```
![Snip](./Screenshots/image8.png)
<br>
<br>

Now let's inspect the specific policy versions associated with the policy:
```
aws iam get-policy-version --policy-arn <policy_arn> --version-id <version>
```
![Snip](./Screenshots/image9.png)
<br>
<br>

Let's inspect another policy version to see if additional permissions or resources are exposed:
```
aws iam get-policy-version --policy-arn <policy_arn> --version-id <version>
```
![Snip](./Screenshots/image10.png)
<br>
<br>

From this policy version we discover both a role as well as another policy reference. This highlights why thorough enumeration is extremely important in AWS environments. Small findings often lead to additional permissions, attack paths, or privilege escalation opportunities.

We can now continue investigating these newly discovered resources:
```
aws iam list-policy-versions --policy-arn <policy_arn>
```
![Snip](./Screenshots/image11.png)
<br>
<br>

Now let's inspect the identified policy version:
```
aws iam get-policy-version --policy-arn <policy_arn> --version-id <version>
```
![Snip](./Screenshots/image12.png)
<br>
<br>

From the policy output we can see that permissions exist for the AWS Secrets Manager service. Specifically, the permissions allow listing and retrieving secrets from the prod/Customers resource.

Tip: Secrets Manager permissions should always be treated as high-value findings during enumeration as they may expose credentials, API keys, database passwords, or other sensitive information that can lead to further compromise.

Next, the previously discovered role was investigated further:
```bash
aws iam list-attached-role-policies --role-name <role_name>
```
![Snip](./Screenshots/image13.png)
<br>
<br>

We can see that the policy previously enumerated is attached to this role. The role details were then retrieved to determine whether the current IAM user has permission to assume it:
```bash
aws iam get-role --role-name <role_name>
```
![Snip](./Screenshots/image14.png)
<br>
<br>

From the trust relationship and policy configuration, it was determined that the provided IAM user is allowed to assume the discovered role.

The role was then assumed for further enumeration:
```bash
aws sts assume-role --role-arn <role_arn> --role-session-name <session_name>
```
![Snip](./Screenshots/image15.png)
<br>
<br>

Temporary credentials for the assumed role were successfully obtained. These credentials can now be configured within the AWS CLI for continued enumeration under the context of the higher privileged role:
```bash
aws configure
```
![Snip](./Screenshots/image16.png)
<br>
<br>

Tip: After assuming a role, always remember that the newly generated credentials are temporary credentials. Depending on the session duration configured, these credentials may expire after a set period of time.

From the earlier policy enumeration, it was already identified that this role possessed permissions to list secrets within Secrets Manager:
```bash
aws secretsmanager list-secrets
```
![Snip](./Screenshots/image17.png)
<br>
<br>

The discovered secret was then retrieved successfully:
```bash
aws secretsmanager get-secret-value --secret-id <secret_id>
```
![Snip](./Screenshots/image18.png)
<br>
<br>

Further enumeration also revealed ListBucket and GetObject permissions against an S3 bucket within the AWS account.

Since we have both the required permissions and the bucket name, the contents of the bucket can now be enumerated:
```bash
aws s3 ls s3://<s3_bucket>
```
![Snip](./Screenshots/image19.png)
<br>
<br>

From the command output we can see that the bucket contains the flag file. Since GetObject permissions are available, the file can be copied locally to the attack host:
```bash
aws s3 cp s3://<s3_bucket>/<flag> .
```
![Snip](./Screenshots/image20.png)
<br>

## Reference(s)
- [Pwned Labs Official Lab](https://pwnedlabs.io/labs/intro-to-aws-iam-enumeration)
- [AWS IAM Documentation](https://docs.aws.amazon.com/iam/)
- [AWS CLI Documentation](https://docs.aws.amazon.com/cli/)
- [AWS STS Documentation](https://docs.aws.amazon.com/STS/latest/APIReference/welcome.html)
- [IAM Policy Versioning](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-versioning.html)
- [AWS S3 Documentation](https://docs.aws.amazon.com/s3/)









