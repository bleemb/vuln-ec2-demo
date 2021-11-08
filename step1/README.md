## Deploy Infrastructure

1.  Find your IP address

From the command line, run the following command.  The output returned is your current IP address, and will be used to configure the instance security group. 

<b> Bash </b>
``` Bash
    IP_ADDRESS=$(curl ifconfig.me)
```

<b> Powershell </b>
``` Powershell
    Set-Variable -Name "IP_ADDRESS" -Value (curl ifconfig.me)
```

<br />
<br />

2.  Configure terminal for deployment of cloudformation template

Execute an aws command to ensure the profile is correctly configured.

This role will require the following permissions: 
- Create/Modify/Delete permissions for ec2
- Create/Modify/Delete permissions for s3
- Create/Delete permissions for IAM

``` script
    cd step1
    aws sts get-caller-identity
```

<br />
<br />
3.  Deploy the cloudformation template

<b> What we will provision </b>
- A t2.micro ec2 instance, vulnerable by default (Credit to SethSec for the vulnerable application: https://github.com/sethsec/Nodejs-SSRF-App)
- Instance profile to be attached to the ec2 instance.  
- Security Group attached to the ec2 instance, with port 80 from your Ip addressed inbound allowed.  
- An s3 bucket

<b> Required variables </b>
- {Subnet-Value} - the subnet id with the ability for port 80 to ingress  
- {VPC-Value} - the VPC Id that this subnet is in.

Paste the following commands, replacing `{Subnet-Value}` and `{VPC-Value}`.  

You can find these in the AWS console, by navigating to the `VPC` service, and selecting the ID of a public subnet, or one that allows port 80 inbound.  

``` aws
aws cloudformation create-stack --stack-name ec2-metadata-ssrf --template-body file://vuln-ec2-template.yaml --parameters ParameterKey="WebServerIngressLocation",ParameterValue="$IP_ADDRESS/32" ParameterKey="Subnet",ParameterValue="{Subnet-Value}" ParameterKey="DeploymentVPC",ParameterValue="{VPC-Value}" --capabilities CAPABILITY_NAMED_IAM
```

Open up cloudformation in the console and validate that the deployment has succeeded.

<br />
<br />
4.  Fill the bucket

Find the name of the bucket, and upload the text file to the recently created bucket.  This will serve as our mock customer information. 


``` bash
    BUCKET_NAME=$(aws cloudformation describe-stack-resource --stack-name ec2-metadata-ssrf --logical-resource-id DataBucket --query StackResourceDetail.PhysicalResourceId --output text)
    
    aws s3 cp secret-file.txt "s3://${BUCKET_NAME}"
```

<b> Powershell </b>
``` Powershell
    Set-Variable -Name "BUCKET_NAME" -Value (aws cloudformation describe-stack-resource --stack-name ec2-metadata-ssrf --logical-resource-id DataBucket --query StackResourceDetail.PhysicalResourceId)
 
    aws s3 cp .\secret-file.txt "s3://$BUCKET_NAME"
```