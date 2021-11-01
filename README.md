# ec2 Metadata Exploitation Workshop

<i> This is a repo designed to walk users through the deployment, and exploitation of a vulnerable ec2 instance via the AWS IMDSv1 SSRF exploit.  Every attempt is made to minimise the risks associated, however this should not be deployed in a production environment. </i>

## Learning Objectives
- Discuss the AWS metadata service, and the role it plays.   
- Discuss Server Side Request Forgery (SSRF).  
- Deploy a vulnerable ec2 instance, and use this technique to extract AWS access keys.  
- Enumerate permissions and exfiltrate data from the account.  
- Patch the vulnerable infrastructure, demonstrating that the previous exploit is no longer possible.  


## Requirements
### Package Dependencies
- awscli
- curl

### Environmental Dependencies
- AWS account
- AWS IAM role or user that can be used to deploy infrastructure and interact with the resources. 
- This repo cloned locally


## How to use this repo
This workshop is broken down into a series of steps.  Each folder contains a README with all of the required steps and information.

In some cases, the code snippets provided are operating system dependent.  Some code snippets will also require users to provide some form of input, and this will be clearly called out in each case. 


## Important Contextual Information 

### What is the AWS instance metadata service?
The instance metadata service (IMDS) is an API used by resources in AWS to access information about themselves.  It can include data such as User data scripts, IDs, Network information.  Critically however, it is the mechanism that ec2 instances use to retrieve credentials related to their instance profile - which affect what API actions they are authorised to carry out.  There are two versions of the IMDS API, v1 and v2.  

### What is SSRF?
SSRF, or Server Side Request Forgery, is a member of the OWASP top 10; and one of the most common web exploit families.  Essentially, it involves making a request that tricks a web server into making a HTTP request and returning the result to the attacker.  As it is the web server making the request, the attacker has the ability to access information that they otherwise would not have access to.

### So what?
As you may have guessed, where SSRF vulnerabilities are present in an application, the IMDSv1 service can be vulnerable to Server Side Request Forgery attacks.  This attack vector can be used to steal AWS credentials, which in turn can be used to access the privileges that the vulnerable ec2 instance had access to.

### How do we mitigate this risk?
- IMDSv2 Service  
Users can choose to restrict ec2 instances to only use the IMDSv2 service, which applies an appropriate mitigation to this vulnerability via a Token methodology.  
- Secure Web Development Practices  
Embracing good secure coding practices such as never trusting user input will reduce the risk of being vulnerable to these types of vulnerabilities.
- Least Privilege Principles  
Whilst this will not completely mitigate this exploit, ensuring that IAM policies are designed with least privilege principles in mind will lower an attackers ability to move laterally in the event that an ec2 instance is compromised.