# vuln-ec2-demo

<i> This is a repo designed to walk users through the deployment, and exploitation of a vulnerable ec2 instance via the metadata API SSRF exploit.  Every attempt is made to minimise the risks associated, however this should not be deployed in a production environment. </i>


### Learning Objectives
- Discuss the AWS metadata service, and the role it plays.   
- Discuss Server Side Request Forgery (SSRF).  
- Deploy a vulnerable ec2 instance, and use this technique to extract AWS access keys.  
- Enumerate permissions and find s3 bucket.  
- Patch the vulnerable infrastructure, demonstrating that the previous exploit is no longer possible.  


### Requirements
#### Package Dependencies
- awscli
- curl

#### Environmental Dependencies
- AWS account & Configured AWS profile
- This repo cloned locally