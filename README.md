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
- TGW ID: `A4LTGW` <br>
- Attachment: `VPN` <br>
Choose your existing customer gateway(`ROUTER-1`), `Dynamic(Requires BGP)` routing, Enable acceleration <br>
Repeat the same steps for `ROUTER-2`

### Download CGW Configurations 
`VPC > Site-to-Site VPN connections` <br>
There will be two connections: one for `ROUTER-1` and one for `ROUTER-2` with their respective public IPs. <br>
Click `Download Configuration` and specify the following. <br>
- Vendor: `Generic` <br>
Make sure you name both files accordingly with the routers public IPs (e.g. `Router1config.txt`). <br>

Now it's time to note down the respective values. In this repository, there's a file named `ConfigTemplate.md`, download it. <br>
In each configuration file that you download earlier, find the respective values and note it down in `ConfigTemplate.md`. The instructions for replacement is already written by Adrian in that template. 

### Configure IPSec tunnels for on-premises routers 
Go to EC2 console: <br>
For each instance (`ONPREM-ROUTER1, ONPREM-ROUTER2`), go into `Connect > Session Manager` and type the following commands: <br>
- `sudo bash`
- `cd /home/ubuntu/demo-assets`
- `vim ipsec.conf` <br> 
We are going to configure two IPSec tunnels for each CGW connections. You will see the placeholders the same as in `ConfigTemplate.md`. Replace the following placeholders: 
- `ROUTER1_PRIVATE_IP`
- `CONN1_TUNNEL1_ONPREM_OUTSIDE_IP`
- `CONN1_TUNNEL1_AWS_OUTSIDE_IP`
- `CONN1_TUNNEL1_AWS_OUTSIDE_IP` 
- `ROUTER1_PRIVATE_IP`
- `CONN1_TUNNEL2_ONPREM_OUTSIDE_IP`
- `CONN1_TUNNEL2_AWS_OUTSIDE_IP`
- `CONN1_TUNNEL2_AWS_OUTSIDE_IP` <br>
Press `Esc` and type `:wq`. Hit `Enter`. <br>

Now, we are going to configure authentication for the tunnels. <br>
- `vim ipsec.secrets`
As before, replace the following placeholders: <br>
- `CONN1_TUNNEL1_ONPREM_OUTSIDE_IP`
- `CONN1_TUNNEL1_AWS_OUTSIDE_IP`
- `CONN1_TUNNEL1_PresharedKey`
- `CONN1_TUNNEL2_ONPREM_OUTSIDE_IP`
- `CONN1_TUNNEL2_AWS_OUTSIDE_IP`
- `CONN1_TUNNEL2_PresharedKey`
Press `Esc` and type `:wq`. Hit `Enter`. <br>

Now, we are going to configure the bash script for IPSec tunnel interfaces. <br>
- `vim ipsec-vti.sh`
As before, replace the following placeholders: <br>
- `CONN1_TUNNEL1_ONPREM_INSIDE_IP (ensuring the /30 is at the end)`
- `CONN1_TUNNEL1_AWS_INSIDE_IP (ensuring the /30 is at the end)`
- `CONN1_TUNNEL2_ONPREM_INSIDE_IP (ensuring the /30 is at the end)`
- `CONN1_TUNNEL2_AWS_INSIDE_IP (ensuring the /30 is at the end)`
Press `Esc` and type `:wq`. Hit `Enter`. <br>

We have to copy these IPSec configuration files into `/etc` directory where other configurations files are located. <br>
`cp ipsec* /etc` <br>
Make the bash script executable `chmod +x /etc/ipsec-vti.sh` <br>

To bring the newly added configurations up, we need to restart our router service, which is `strongswan` in this project <br>
`systemctl restart strongswan` 

Wait for a minute or two and ensure the IPSec tunnels are up and running <br>
`ifconfig` <br>
You will see two new interfaces are configured: `vti1` and `vti2`. 













