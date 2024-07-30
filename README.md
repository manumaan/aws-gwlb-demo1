# aws-gwlb-demo1
Demo of AWS GWLB with one hand architecture

Architecture is shown below: 
![glbwdemo diagram](https://github.com/user-attachments/assets/a52ed13d-5bae-425e-85eb-f61ecd1f708c)


Once the resources are created, you can ssh into the app-ec2 (Application Ec2) and fw-ec2 (Firewall appliance EC2) using EC2 Instance Connect Endpoints. 
Check status of the GWLBTUN tunnel on the fw-ec2: 

sudo systemctl status gwlbtun

This is a piece of software that takes the GENEVE traffic, decapsulates to TCP, then encapsulates back to GENEVE and sends it back. This allows us to see the GWLB in action. 
GWLBTUN Repo: https://github.com/aws-samples/aws-gateway-load-balancer-tunnel-handler

Now from the app-ec2 access internet by ping

ping 8.8.8.8 

All traffic from app-ec2 not destined for local resources are sent to the GWLB Endpoint, which forwards it to GWLB, which forwards it to the GWLBTUN appliance. From there it comes back the same way into GWLB Endpoint and then goes to the actual destination (In our case, NAT Gateway -> Internet Gateway -> Internet) 

To verify the GENEVE traffic coming into the appliance you can run tcpdump in the fw-ec2. 

sudo tcpdump -n -i enX0 port 6081

Output will show, right to left - original TCP data the GWLB recieved, the GENEVE encapsulation on top of it, and then TCP/UDP encapsualtion by the GWLBTUN on top of it. 

