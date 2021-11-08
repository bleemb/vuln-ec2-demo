## Patch your infrastructure & Clean Up

1.  Switch back to your development profile.  Run this command to ensure you are using the right profile.

``` script 
    aws sts get-caller-identity
```
<br />
<br />
2.  Run a CLI command to force the ec2 instance to require a http token to be present when issuing a command


<b> Bash </b>
``` script
    INSTANCE_ID=$(aws cloudformation describe-stack-resource --stack-name ec2-metadata-ssrf --logical-resource-id vulnWebServer --query StackResourceDetail.PhysicalResourceId --output text)

    aws ec2 modify-instance-metadata-options --instance-id "$INSTANCE_ID" --http-tokens required --http-endpoint enabled 
```

<b> Powershell </b>
``` script
    Set-Variable -Name "INSTANCE_ID" -Value (aws cloudformation describe-stack-resource --stack-name ec2-metadata-ssrf --logical-resource-id vulnWebServer --query StackResourceDetail.PhysicalResourceId --output text)

    aws ec2 modify-instance-metadata-options --instance-id "$INSTANCE_ID" --http-tokens required --http-endpoint enabled 
```

Our ec2 is still running, meaning that the response will return the `State` as `Pending`.

<br />
<br />
3. Test our mitigations.

Run the same exploits against the web server - has our patch worked successfully?

```script
 http://<public_ip>/?url=http://169.254.169.254/latest/meta-data/iam/

 http://<public_ip>/?url=http://169.254.169.254/latest/meta-data/iam/security-credentials

 http://<public_ip>/?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/{name-of-your-instance-profile}
```