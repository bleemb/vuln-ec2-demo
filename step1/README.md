## Deploy Infrastructure

#### 1. Find your IP address

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

#### 2. Configure terminal for deployment of cloudformation template

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

####  3. Deploy the cloudformation template


<b> What we will provision </b>
- A t2.micro ec2 instance, vulnerable by default (Credit to SethSec for the vulnerable application: https://github.com/sethsec/Nodejs-SSRF-App)
- Instance profile to be attached to the ec2 instance.  
- Security Group attached to the ec2 instance, with port 80 from your Ip addressed inbound allowed.  
- An s3 bucket

<b> Required variables </b>
- {Subnet-Value} - the subnet id with the ability for port 80 to ingress  
- {VPC-Value} - the VPC Id that this subnet is in.

Paste the following commands, replacing `{Subnet-Value}` and `{VPC-Value}`.  

You can find these in the AWS console, [by navigating to the VPC service, and selecting the ID of a public subnet.](https://ap-southeast-2.console.aws.amazon.com/vpc/home?region=ap-southeast-2)

*Note:* Subnet is considered public if it has direct access to internet via internet gateway. If you
are operating within default VPC created by AWS on your behalf, during account creation, all subnets
will be public by default. VPC ID can also be discovered from subnet's console, in "VPC" column. 

<img alt="public subnet img" src="https://user-images.githubusercontent.com/1170273/142021113-b45706a5-6585-477e-9e70-68ef4b89719b.png" />


``` aws
aws cloudformation create-stack --stack-name ec2-metadata-ssrf \
    --template-body file://vuln-ec2-template.yaml \
    --parameters \
     ParameterKey="WebServerIngressLocation",ParameterValue="$IP_ADDRESS/32" \
     ParameterKey="Subnet",ParameterValue="{Subnet-Value}" \
     ParameterKey="DeploymentVPC",ParameterValue="{VPC-Value}" \
     --capabilities CAPABILITY_NAMED_IAM
```



Open up [cloudformation in the console](https://ap-southeast-2.console.aws.amazon.com/cloudformation/home?region=ap-southeast-2#/stacks?filteringStatus=active&filteringText=&viewNested=true&hideStacks=false) and validate that the deployment has succeeded.

<br />
<br />

#### 4.  Fill the bucket


The CloudFormation stack named ec2-metadata-ssrf we deployed previously has created an S3 Object Store bucket.
In this step, we will discover the name of this bucket, and upload the text file to the recently created bucket. This will serve to demonstrate web application access to potentially sensitive and confidential information. 

<b> Bash </b>

``` bash
    # discover bucket name and put it in variable
    BUCKET_NAME=$(aws cloudformation describe-stack-resource --stack-name ec2-metadata-ssrf --logical-resource-id DataBucket --query StackResourceDetail.PhysicalResourceId --output text)
    
    # copy the PII data to the bucket. Application now has access to this data
    aws s3 cp secret-file.txt "s3://${BUCKET_NAME}"
```

<b> Powershell </b>
``` Powershell
    Set-Variable -Name "BUCKET_NAME" -Value (aws cloudformation describe-stack-resource --stack-name ec2-metadata-ssrf --logical-resource-id DataBucket --query StackResourceDetail.PhysicalResourceId)
 
    aws s3 cp .\secret-file.txt "s3://$BUCKET_NAME"
```

You can now move to [step 2](../step2/README.md)