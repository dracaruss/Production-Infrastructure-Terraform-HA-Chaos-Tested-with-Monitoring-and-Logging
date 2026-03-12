# High Availability and Secure Chaos-Tested Production Infrastructure with Monitoring and Logging

##

# **Security & Resilience Architecture**  
> [!IMPORTANT]
> This was a recent job I did for a client, implementing a Defense-in-Depth strategy.  
> The following elements were used, to ensure that the infrastructure was not only highly available, but also hardened.  


## **Zero-Trust Network Design**  
*Isolated Database Tier:*  
>Database instances are placed in dedicated subnets with no Route Table entry to the Internet Gateway, preventing "phone-home" malware communication.

*Elimination of Port 22:*  
>By leveraging AWS Systems Manager (SSM) and Interface VPC Endpoints, the infrastructure allows for full administration without the need for SSH keys or public-facing Bastion hosts.

*Security Group Chaining:*  
>Implements a strict "Kill Chain" where the Database only accepts traffic from the App Tier, and the App Tier only accepts traffic from the ALB, preventing lateral movement.  


## Active Threat Mitigation  
*Intelligent WAF Filtering:*  
>An AWS WAF is deployed in front of the Application Load Balancer to inspect Layer 7 traffic, automatically rate-limiting IPs and blocking SQL Injection (SQLi) and Cross-Site Scripting (XSS) attempts.

*GuardDuty AI Monitoring:*  
>Continuous monitoring of VPC Flow Logs and CloudTrail events via Amazon GuardDuty to detect anomalous behavior, such as unauthorized API calls or compromised instances.

*Encryption in Transit:*  
>Implements End-to-End TLS (HTTPS). Traffic is encrypted from the client to the ALB and remains encrypted from the ALB to the EC2 instances on port 8080.

## Automated Reliability & "Chaos" Validation
*Multi-AZ Self-Healing:*  
>The infrastructure is distributed across two Availability Zones. In the event of a localized data center failure, the Auto Scaling Group (ASG) automatically redistributes capacity to maintain 100% uptime.

*Targeted Stress Testing:*  
>Validated the scaling policy using Grafana k6 to simulate high-computational load (SHA-256 hashing) and concurrent traffic spikes, confirming the system's ability to scale from 2 to 6 instances dynamically.

*Observability Dashboard:*  
>A custom CloudWatch Dashboard provides real-time visibility into CPU utilization, Request Count, and Healthy Host counts for rapid incident response.

##

# The overview of the project  
I accepted a job from a client, who was requesting a particular AWS cloud configuration, which needed to be handled in a secure and professional manner. 

# **The high level job description by the client**  
> [!IMPORTANT]
> "I’m ready to launch a production-grade web application on AWS and need the underlying infrastructure designed and built with security and high availability at >its core. The workload will run on EC2, and general-purpose instances are fine; the focus is on a rock-solid architecture, not exotic hardware profiles.
>
>Here’s what I have in mind: a VPC with public and private subnets spread across at least two Availability Zones, an Application Load Balancer in front, Auto Scaling groups to keep capacity healthy, and tight security groups plus IAM roles that follow least-privilege rules. Logging to S3, monitoring and alerts through CloudWatch, and encrypted traffic everywhere (ACM certificates, HTTPS, TLS-enabled internal hops) are all must-haves. Wherever possible, I’d like the stack expressed as infrastructure-as-code—Terraform or CloudFormation—so I can version-control and reproduce everything.

>**Deliverables**  
>* Architecture diagram (high-level and subnet-level)  
>* IaC templates or modules with clear variables and README  
>* Step-by-step deployment guide and rollback instructions  
>* Brief security checklist showing how OWASP/AWS best practices are met  
>* Final validation session in my account to confirm multi-AZ failover and basic load test success"  

##

# The solution architecture I constructed  
> [!NOTE]
> Create dedicated DB Subnets with no internet route, I made it impossible for the database to "call home" to a hacker's server, even if the app layer is compromised.

**Elimination of Public Exposure (VPC Endpoints):**  
>*By using Interface Endpoints for SSM, I completely removed the need for SSH (Port 22) or even a Bastion Host. Instead the client will be managing servers over AWS's private internal fiber.*
>
**Defense in Depth (Security Group Chaining):**  
>*The DB only talks to the App SG, and the App SG only talks to the ALB SG. This creates a "kill chain"—an attacker has to break three distinct layers to reach the data.*
>
**Visibility & Accountability:**  
>*Adding VPC Flow Logs and CloudTrail means the setup will have a "black box recorder" for both network traffic and human API calls. This is a requirement for SOC 2 and HIPAA compliance.*
>
**AWS WAF (Web Application Firewall):**  
>A Load Balancer only checks if a packet is coming to the right port. AWS WAF looks inside the packet to see if it contains a SQL injection attack or a Cross-Site Scripting (XSS) script.
>
**Amazon GuardDuty (The AI Watchdog):**  
>I have logs (Flow Logs, CloudTrail), but who is reading them? GuardDuty uses machine learning to analyze those logs 24/7. It will alert if an EC2 instance starts behaving like a crypto-miner or if a login is coming from an unusual country.
>
**S3 Bucket Hardening:**  
>Since the code is sending all the sensitive logs to S3, if that bucket is accidentally made public, the client's entire security audit trail is exposed. So hardening the S3 bucket is essential.
>
**Multi-AZ Failover Test (The "Chaos" Test):**  
>Create tests that verify the auto-scaling and load balancing is working as intended.
>* Use the k6 tool to send traffic to the ALB's URL.  
>* Enable the CloudWatch Dashboard for monitoring.  
>* As the traffic climbs, show the CPU utilization rising, and show the Auto Scaling Group adding another server to handle the load.  

##  

# **This is the flow chart of the infrastructure I intended to design:**  
<img width="512" height="732" alt="arch" src="https://github.com/user-attachments/assets/062fa782-9520-4211-b541-21c00c2ffb22" />

##
  
*I improved the look for the client with Google's Gemini Nano Banana 2:*  
![Gemini_Generated_Image_4f6y24f6y24f6y24 (1)](https://github.com/user-attachments/assets/f406aded-fb04-4162-a689-4440b6919709)

##  

*The folder structure is:*  
>/client-project-name  
>├── providers.tf&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;# The "Who": AWS & Terraform version settings  
>├── main.tf&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;# The "What": VPC, ALB, and Network resources  
>├── compute.tf&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;# The "Logic": EC2 Instances and Auto Scaling  
>├── security.tf&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;# The "Guardrails": IAM Roles, WAF, Security Groups  
>├── variables.tf&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;# The "Inputs": Configurable settings (Regions, CIDRs)  
>├── outputs.tf&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;# The "Results": ALB URL, Instance IDs  
>├── data.tf&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;# The data  
>├── monitoring.tf&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;# The monitoring resources  
>├── terraform.tfvars&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;# The "Values": Actual data for the variables  
>└── README.md&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;# The "Manual": How the client deploys it  

##

# A walkthrough of the terraform configuration process 
### Identity Center vs IAM (Stock OEM)  
>To not have to use any long term or potentially risky access keys, I decided to setup IAM Identity Center instead of normal IAM:
<img width="1087" height="255" alt="iam" src="https://github.com/user-attachments/assets/3e475bd8-0f91-4a9b-8c13-77c97c3756fe" />

##

First I connected to AWS from my Kali console for a 2 hour session limit:  
```
$ aws configure sso
```
<img width="732" height="233" alt="sign in" src="https://github.com/user-attachments/assets/437485c0-d34e-458b-b775-fcf45d3d1501" />

##  

Ok since my files are ready, I can start my terraform procedures to create my cloud infrastructure:
```
$ terraform init
```
<img width="859" height="363" alt="init" src="https://github.com/user-attachments/assets/d7188d67-f8e6-412e-8250-d7eaf4befadc" />

##

Ok i'm in business. Terraform fmt returned no issues. So I moved on to terraform validate:
```
$ terraform fmt
$ terraform validate  
```
<img width="508" height="77" alt="validate" src="https://github.com/user-attachments/assets/1cfa0777-0b08-46e5-b1ae-90dd4c701fba" />

##

Ok also no issues so lets get going with the plan:
```
$ terraform plan
```
<img width="839" height="588" alt="plan" src="https://github.com/user-attachments/assets/c84bd84f-6e5f-4dcc-bf88-a2f5a34d7f72" />

##

Time to test the auto scaling group, to see if it replenishes the instances if one fails the health check. 
Right now both are running:
<img width="1075" height="205" alt="autoscaling" src="https://github.com/user-attachments/assets/8ad56cc3-25ea-47fd-b404-6e98f376f309" />

##

Let me manually terminate one and see if a new one is started:
<img width="1260" height="260" alt="terminate" src="https://github.com/user-attachments/assets/bb12cd61-804e-4a83-8244-efbf7e1e50c4" />

##

Now it's shutting down, so lets see if it triggers a new one:
<img width="808" height="282" alt="shut down" src="https://github.com/user-attachments/assets/9be2d3cc-c7a6-446e-8b03-daae8c0485bd" />

##

Ok great after the deletion one more is auto starting:
<img width="1078" height="277" alt="initializing" src="https://github.com/user-attachments/assets/2c7c1955-4746-4a28-ba19-bb5265b07633" />

##

Ok now to demonstrate to the client that the WAF is working, by simulating a Brute Force or DOS attack. I will share my screen with the client, and run a stress test using Grafana K6 (A stress test tool), to display that the WAF is working right to rate limit any IP sending too much traffic.  
##  
The WAF chipped in and blocked my IP:
<img width="1048" height="679" alt="waf chipped in" src="https://github.com/user-attachments/assets/b5d66751-4bbd-4164-85ee-baeac86edcd8" />

##

Cloudwatch showed the huge rise in requests:
<img width="1327" height="403" alt="cloudwatch" src="https://github.com/user-attachments/assets/ba36e7bb-67cc-4042-a5e0-8d7a3523904d" />

##

Since I know the WAF is working and rate limited me during the last stress test, let me disassociate it from the ALB and run the stress test again to test the Auto scaling:
<img width="577" height="182" alt="no associated resources" src="https://github.com/user-attachments/assets/ffb63b61-a198-43d1-85d7-2c6429b6306a" />

##

I setup a Python HTTPS server on both instances using the SSM console. The instance must speak HTTPS because that's what the target group protocol dictates — the ALB initiates an HTTPS connection to port 8080 on the EC2 instances, so there has to be a TLS server listening there.
```
cd /tmp

openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 1 -nodes -subj '/CN=localhost'
python3 -c "
import ssl, http.server
class H(http.server.SimpleHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b'ok')
ctx = ssl.SSLContext(ssl.PROTOCOL_TLS_SERVER)
ctx.load_cert_chain('cert.pem', 'key.pem')
server = http.server.HTTPServer(('0.0.0.0', 8080), H)
server.socket = ctx.wrap_socket(server.socket, server_side=True)
server.serve_forever()"
```

##

Instance 1 shows its 'healthy' and ready:
<img width="817" height="820" alt="healthy and ready" src="https://github.com/user-attachments/assets/4c672a1a-b265-448b-9b27-618757e3be78" />

##

Instance 2 in the second AZ also:
<img width="778" height="185" alt="insatnce2" src="https://github.com/user-attachments/assets/976e7f6e-e664-4eed-a0d9-d2078775b11b" />

##

Ok great health checks are being passed with 200s, now it's time to run a stress test again without the WAF, to see what happens with the ASG:
<img width="760" height="312" alt="ok great" src="https://github.com/user-attachments/assets/8ad1d433-1807-460c-b295-1cb5bf5734cb" />

##

I bombarded the EC2 mini's with requests again using Grafana K6 from Kali:
```
$ k6 run -e TARGET_URL=http://myapp-prod-alb-1305166777.us-east-2.elb.amazonaws.com stress_test.js
```

##

The test kicked off:
<img width="1191" height="775" alt="k6-1" src="https://github.com/user-attachments/assets/24e7cf8c-09a3-4277-9ca1-026d5039bc2a" />

##

The results were no WAF blocking and 92% success with 200s:
<img width="554" height="233" alt="no waf" src="https://github.com/user-attachments/assets/72cbb98c-ffb7-4bb9-ada5-8280318e0b22" />

##

The EC2's stood up like champs initially, but collapsed eventually and triggered the autoscale:
<img width="978" height="595" alt="the ec2s" src="https://github.com/user-attachments/assets/5a047421-0b8b-4f6d-aed7-837cacf66c5c" />

##

Now I had my max limit of 6 EC2s running:
<img width="1425" height="367" alt="now i had" src="https://github.com/user-attachments/assets/ca09590e-5c2e-4998-94c7-1e761eaf6b3f" />

##

Cloudwatch showed the dashboard was working as planned and showed the increase in instance count to the maximum of 6:
<img width="941" height="378" alt="cloudwatch showed" src="https://github.com/user-attachments/assets/1fc1f35a-58be-4aea-843f-a998a4cd139a" />

##

As things cooled down, I saw the instances properly terminate one by one, resetting the count back down to just 2:
<img width="1408" height="371" alt="as things cooled" src="https://github.com/user-attachments/assets/a0ae4c2b-6c9d-4fb5-8030-892c72b82621" />

##

And the downscale was also captured on Cloudwatch:
<img width="940" height="362" alt="and the downscale" src="https://github.com/user-attachments/assets/4daa5d50-b446-4203-bd7f-787bdb64791b" />

##


Ok for a second stress test, I decided to just overwhelm the CPUs themself. It's a more realistic simulation of what happens in production when an app is under heavy load.  
<br>
First I needed to generate new HTTPS certs, so the ALB can talk to the EC2s, since these instances are new ones generated by the ASG:
```
$ cd /tmp
$ openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 1 -nodes -subj '/CN=localhost'
```






> [!NOTE]
> This is a blue box for general info.

> [!TIP]
> This is a green box for tips/successes.

> [!IMPORTANT]
> This is a purple box for key security details.

> [!WARNING]
> This is an orange box for warnings.

> [!CAUTION]
> This is a red box for critical "danger" info.
