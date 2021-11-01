

## Deploy Infrastructure

1.  Find IP address

From the command line, run the following command

``` script Bash
    IP_ADDRESS = $(curl ifconfig.me)

```

2.  Configure terminal for deployment of cloudformation template

Set the AWS profile that will be used for deployment.  Execute an aws command to ensure the profile is correctly configured.

This role will require the following permissions: 
- Create/Modify/Delete permissions for ec2
- Create/Modify/Delete permissions for s3

<i> Linux or macOS </i>
``` script Bash
    aws sts get-caller-identity
```

<i> Powershell </i>
``` script 
    aws sts get-caller-identity
```

3.  Deploy the cloudformation template

<i> What we will provision </i>
- t2.micro ec2 instance, vulnerable by default (Credit to SethSec for the vulnerable application: https://github.com/sethsec/Nodejs-SSRF-App)
- instance profile to be attached to the ec2 instance.  
- security group attached to the ec2 instance, with port 80 from your Ip addressed inbound allowed.  
- an s3 bucket

<b> Required variables </b>
- {Subnet-Value} - the subnet id with the ability for port 80 to ingress  
- {VPC-Value} - the VPC that this subnet is in.

``` script
    aws cloudformation create-stack --stack-name ec2-metadata-ssrf --template-body file://vuln-ec2-template.yaml --parameters ParameterKey="WebServerIngressLocation",ParameterValue="$(IP_ADDRESS)/32" ParameterKey="Subnet",ParameterValue="{Subnet-Value}" ParameterKey="DeploymentVPC",ParameterValue="{VPC-Value}" --capabilities CAPABILITY_NAMED_IAM

```

``` Powershell
aws cloudformation create-stack --stack-name ec2-metadata-ssrf --template-body file://vuln-ec2-template.yaml --parameters ParameterKey="WebServerIngressLocation",ParameterValue="$IP_ADDRESS/32" ParameterKey="Subnet",ParameterValue="{Subnet-Value}" ParameterKey="DeploymentVPC",ParameterValue="{VPC-Value}" --capabilities CAPABILITY_NAMED_IAM
```

4.  Fill the bucket

Find the name of the bucket, and upload the text file to the recently created bucket.

``` script
    bucketname = aws cloudformation describe-stack-resource --stack-name ec2-metadata-ssrf --logical-resource-id DataBucket --query StackResourceDetail.PhysicalResourceId
    aws s3 cp secret-file.txt "s3://${bucketname}"
```

``` Powershell
 Set-Variable -Name "BUCKET_NAME" -Value (aws cloudformation describe-stack-resource --stack-name ec2-metadata-ssrf --logical-resource-id DataBucket --query StackResourceDetail.PhysicalResourceId)
 aws s3 cp .\secret-file.txt "s3://$BUCKET_NAME"
```