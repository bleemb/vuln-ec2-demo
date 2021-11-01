## Clean-Up

1.  Remove the secret file from your s3 bucket

s3 buckets are not able to be deleted unless they are empty.  We need to remove the secret-file from earlier.

<b> Variables Required </b>
- {bucket-name} - The name of the ec2-metadata bucket we provisioned.

``` script
    aws s3 rm s3://{bucket-name}/secret-file.txt
```

2.  Delete the stack

```
    aws cloudformation delete-stack --stack-name ec2-metadata-ssrf
```

Monitor the progress, to ensure all resources are deleted as expected.