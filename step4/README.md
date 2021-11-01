## Patch your infrastructure & Clean Up

1.  Switch back to your development profile

``` script 
    aws sts get-caller-identity
```

2.  Run a CLI command to force the ec2 instance to require a http token to be present when issuing a command

<b> variables required </b>
- {Instance_id} - The instance ID of our web server.  You can find this on the AWS console, an ID beginning with 'i-...'

``` script
    aws ec2 modify-instance-metadata-options --instance-id {Instance_id} --http-tokens required --http-endpoint enabled 
```

Our ec2 is still running, meaning that the response will return the `State` as `Pending`.

3. Test our mitigations, 

Run the same exploits against the web server - has our patch worked successfully?

```script
 http://<public_ip>/?url=http://169.254.169.254/latest/meta-data/iam/
 http://<public_ip>/?url=http://169.254.169.254/latest/meta-data/iam/security-credentials
 http://<public_ip>/?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/{name-of-your-instance-profile}
```