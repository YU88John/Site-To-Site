# Highly Available and Dynamic Site-to-site VPN stimulated on AWS 
This project was tested during my AWS SOA-C02 learning journey. The backbone of the project is heavily referenced to Adrian Cantrill's idea during the course. The course is available at: https://learn.cantrill.io/p/aws-certified-sysops-administrator-associate

### What is AWS Site-to-Site VPN and why?
Scenario: You're a cloud architect of a company which hosts its applications and services hybrid. Your task is to ensure there is a seamless and secure connection between resources in both environments. <br>
Certainly, this can be easily achieved with something like EC2 ssh-login. Nevertheless, this method possesses a potential vulnerability as anyone with ssh key can access the server. <br>
For such situation, we can utilize **AWS Site-to-Site VPN**. By setting up S2S VPN, we have benefits such as secure and private data transfer, hybrid deployments, scalability, and so on. You can read more about S2S VPN at: https://aws.amazon.com/vpn/site-to-site-vpn/

### Create the stack
Since there is no on-premises readily available, we are stimulating both environments on AWS. <br>
The Cloud Formation template is readily provided in this repository. `Download the template > Go to CFN console > Upload it`. It will take 10-15 minutes for the stack to complete its creation. You will see the public and private IPs of on-premises routers under the Output section of Cloud Formation. <br> <br>
This stack creates: two on-premises servers, two on-premises routers, and two AWS EC2 instances. <br>
You can ensure there is no connectivity yet between servers by SSHing(`Session Manager`) into one of EC2 instances and pinging the private IP of an on-premises server.

### Setup Customer Gateway
Since we are designing a Highly Available VPN connection, we will setup two Customer Gateway(CGW) for two on-premises routers <br>
In the VPC console: <br>
`Virtual Private Network > Customer Gateways > Create Customer Gateway` <br>
Name: `ROUTER-1`, BGP: `65016`, IP: `<PUBLIC_IP_OF_ROUTER1>` <br>
Repeat the same steps for `ROUTER-2`.  

### Create Transit Gateway attachments 
CGW is for on-premises routing. To route the incoming network from on-premises to AWS VPCs, we need to create Transit Gateway(TGW) attachments. <br>
In the VPC console: <br>
`Transit Gateway Attachments > Create` <br>
TGW ID: `A4LTGW` <br>
Attachment: `VPN` <br>
Choose your existing customer gateway(`ROUTER-1`), `Dynamic(Requires BGP)` routing, Enable acceleration <br>
Repeat the same steps for `ROUTER-2`




