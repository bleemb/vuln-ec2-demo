## Clean-Up

1.  Remove the secret file from your s3 bucket

s3 buckets are not able to be deleted unless they are empty.  We need to remove the secret-file from earlier.

``` bash
    BUCKET_NAME=$(aws cloudformation describe-stack-resource --stack-name ec2-metadata-ssrf --logical-resource-id DataBucket --query StackResourceDetail.PhysicalResourceId --output text)

    aws s3 rm s3://{BUCKET_NAME}/secret-file.txt
```

<b> Powershell </b>
``` Powershell
    Set-Variable -Name "BUCKET_NAME" -Value (aws cloudformation describe-stack-resource --stack-name ec2-metadata-ssrf --logical-resource-id DataBucket --query StackResourceDetail.PhysicalResourceId)
 
    aws s3 rm s3://{BUCKET_NAME}/secret-file.txt
```

<br />
<br />

2.  Delete the stack

```
    aws cloudformation delete-stack --stack-name ec2-metadata-ssrf
```

Monitor the progress, to ensure all resources are deleted as expected.