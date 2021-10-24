

## Deploy Infrastructure

1.  Find IP address

From the command line, run the following command

``` script Bash
    IP_ADDRESS = $(curl ifconfig.me)

```

2.  Configure terminal for deployment of cloudformation template

Set the AWS profile that will be used for deployment.  Execute an aws command to ensure the profile is correctly configured.

<i> Linux or macOS </i>
``` script Bash
    export AWS_PROFILE=${name_of your_profile}
    aws sts get-caller-identity
```

<i> Windows </i>
``` script 
    set AWS_PROFILE {name_of_your_profile}
    aws sts get-caller-identity
```

3.  Deploy the cloudformation template

<i> What we will provision </i>
- t2.micro ec2 instance.  
- instance profile to be attached to the ec2 instance.  
- security group attached to the ec2 instance, with port 80 from your Ip addressed inbound allowed.  
- an s3 bucket

<i> required information </i>
- the subnet id with the ability for port 80 to ingress  
- the VPC that this subnet is in

``` script
    aws cloudformation create-stack --stack-name ec2-metadata-ssrf --template-body file://vuln-ec2-template.yaml --parameters ParameterKey="WebServerIngressLocation",ParameterValue=$(IP_ADDRESS) ParameterKey="Subnet",ParameterValue="" ParameterKey="DeploymentVPC",ParameterValue="" --capabilities CAPABILITY_IAM

```

4.  Fill the bucket

Find the name of the bucket, and upload the text file to the recently created bucket.

``` script
    bucketname = aws cloudformation describe-stack-resource --stack-name ec2-metadata-ssrf --logical-resource-id DataBucket --query StackResourceDetail.PhysicalResourceId
    aws s3 cp secret-file.txt $bucketname
```