

## Deploy Infrastructure

1.  Find IP address

From the command line, run the following command

``` script Bash
    IP_ADDRESS = $(curl ifconfig.me)

```

2.  Configure cli for deployment

Configure the AWS profile that will be used for deployment.

<i> Linux or macOS </i>
``` script Bash
    export AWS_PROFILE=${name_of your_profile}
```

<i> Windows </i>
``` script 
    set AWS_PROFILE {name_of_your_profile}
```