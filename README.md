# Application-Load-Balancer
Highly Available Node.js Application on AWS using ALB + 2 EC2 Instances
Architecture Overview
User → Application Load Balancer → EC2 (AZ-1)
                              → EC2 (AZ-2)


If one EC2 instance fails, the ALB automatically routes traffic to the other instance, ensuring high availability.

1. Create VPC

Go to VPC → Create VPC

Name: ha-vpc

IPv4 CIDR: 10.0.0.0/16

Tenancy: Default

Click Create

2. Create Public Subnets (Across 2 AZs)

Public Subnet 1

Name: public-subnet-1

AZ: ap-south-1a

CIDR: 10.0.1.0/24

Public Subnet 2

Name: public-subnet-2

AZ: ap-south-1b

CIDR: 10.0.2.0/24

✅ Both subnets are required for an ALB to distribute traffic across AZs.

3. Create Internet Gateway (IGW)

Navigate to VPC → Internet Gateways → Create

Name: ha-igw

Attach to ha-vpc

4. Create Public Route Table

Go to VPC → Route Tables → Create

Name: public-rt

VPC: ha-vpc

Edit Routes:

Destination: 0.0.0.0/0

Target: ha-igw

Subnet Association:

Associate with public-subnet-1 and public-subnet-2

✅ Both subnets are now public.

5. Create Security Groups
ALB Security Group (alb-sg)

Inbound: HTTP, Port 80, Source 0.0.0.0/0

Outbound: Allow all

EC2 Security Group (ec2-sg)

Inbound:

HTTP, Port 3000, Source: alb-sg

SSH, Port 22, Source: Your IP

Outbound: Allow all

⭐ EC2 instances only accept HTTP traffic from the ALB for security.

6. Launch 2 EC2 Instances (Node.js)

AMI: Amazon Linux 2

Instance Type: t2.micro

VPC: ha-vpc

Subnets:

EC2-1 → public-subnet-1

EC2-2 → public-subnet-2

Security Group: ec2-sg

Key Pair: Your key

7. Deploy Node.js Applications
EC2-1 (app.js)
const http = require('http');

http.createServer((req, res) => {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello from EC2 Instance 1 🚀');
}).listen(3000);

console.log("Server running on port 3000");

EC2-2 (app.js)
const http = require('http');

http.createServer((req, res) => {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello from EC2 Instance 2 ⚡');
}).listen(3000);

console.log("Server running on port 3000");


Install Node.js & Run

sudo yum update -y
curl -sL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo yum install nodejs -y
node app.js


Test locally:
http://EC2_PUBLIC_IP:3000

8. Create Target Group

EC2 → Target Groups → Create

Type: Instances

Protocol: HTTP

Port: 3000

VPC: ha-vpc

Health Check Path: /

Register both EC2 instances

✅ Both instances should show Healthy.

9. Create Application Load Balancer (ALB)

EC2 → Load Balancers → Create

Type: Application Load Balancer

Name: ha-alb

Scheme: Internet-facing

VPC: ha-vpc

Subnets: public-subnet-1 and public-subnet-2

Security Group: alb-sg

Listener: HTTP → Forward to target group

10. Test High Availability

Open the ALB DNS:
http://ha-alb-xxxxxxxx.ap-south-1.elb.amazonaws.com

Refresh multiple times → Should alternate responses between EC2-1 and EC2-2

Stop EC2-1

Refresh ALB URL → Only see EC2-2 response

Confirms failover works
