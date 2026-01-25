# Chapter 2 — EC2 Web Server (Hands-on, Full Debugging Walkthrough)

## Purpose of This Chapter
The purpose of Chapter 2 is to launch a real Amazon EC2 instance inside a custom VPC, connect to it securely using SSH, install a web server, and expose it to the internet over HTTP.

This chapter intentionally includes **real-world troubleshooting** to demonstrate how AWS networking actually behaves in practice, not just ideal scenarios.

By the end of this chapter, we prove that:
- The VPC created in Chapter 1 works correctly
- An EC2 instance can be launched and accessed securely
- HTTP traffic can flow end-to-end from the public internet to the instance
- AWS networking issues can be debugged layer by layer

---

## Architecture Flow (End-to-End)

Client Browser  
→ Internet  
→ Internet Gateway  
→ Route Table (0.0.0.0/0 → IGW)  
→ Public Subnet  
→ Network ACL (Inbound + Outbound Allowed)  
→ Security Group (HTTP + SSH Allowed)  
→ EC2 Instance  
→ nginx Web Server (Port 80)

---

## EC2 Instance Configuration

- Instance Name: `ch2-web-server`
- Instance Type: `t2.micro` (AWS Free Tier)
- Operating System: Amazon Linux
- VPC: `project-vpc`
- Subnet: `public-subnet`
- Auto-assign Public IPv4: Enabled
- Security Group: `web-sg`
- SSH User: `ec2-user`

---

## Key Pair Creation (SSH Access)

To connect securely to the EC2 instance, a key pair was created.

- Key Name: `aws-major-key`
- Key Type: RSA
- File Format: `.pem`

The key file was downloaded locally and stored securely.

Example local path:
```bash
~/Downloads/aws-major-key.pem
Permissions were restricted to prevent unauthorized access:

bash
Copy code
chmod 400 ~/Downloads/aws-major-key.pem
Connecting to the EC2 Instance via SSH
Command used from the local machine:

bash
Copy code
ssh -i ~/Downloads/aws-major-key.pem ec2-user@<PUBLIC_IP>
Example:

bash
Copy code
ssh -i ~/Downloads/aws-major-key.pem ec2-user@98.92.235.155
Successful connection is confirmed when the terminal prompt changes to:

bash
Copy code
[ec2-user@ip-10-0-1-28 ~]$
Installing nginx Web Server
After connecting to the instance, nginx was installed and configured.

Install nginx:

bash
Copy code
sudo dnf install -y nginx
Enable nginx at startup:

bash
Copy code
sudo systemctl enable nginx
Start nginx:

bash
Copy code
sudo systemctl start nginx
Verify nginx is running:

bash
Copy code
sudo systemctl status nginx
Expected output:

text
Copy code
Active: active (running)
Deploying a Test Web Page
A simple HTML page was created to confirm HTTP functionality.

bash
Copy code
echo "<h1>Chapter 2 success – EC2 is working</h1>" | sudo tee /usr/share/nginx/html/index.html
Local Verification (Inside EC2)
Before testing externally, HTTP was tested locally:

bash
Copy code
curl http://localhost
Expected output:

html
Copy code
<h1>Chapter 2 success – EC2 is working</h1>
This confirms:

nginx is running

Port 80 is open locally

The correct file is being served

Initial Problem Encountered (Browser Timeout)
When accessing the public IP from a browser:

text
Copy code
http://98.92.235.155
The request initially timed out.

This confirmed:

EC2 was reachable via SSH

HTTP traffic was failing somewhere in the networking stack

Debugging Process (Step-by-Step)
1. Security Group Verification
Checked inbound rules for web-sg:

SSH (22) → allowed from personal IP

HTTP (80) → allowed from 0.0.0.0/0

Result: Security Group was correct.

2. nginx Verification
Confirmed nginx was running and listening:

bash
Copy code
sudo systemctl status nginx
sudo ss -tulpn | grep :80
Result:

nginx running

listening on 0.0.0.0:80

3. OS Firewall Check
Checked for firewalld:

bash
Copy code
sudo systemctl status firewalld
Result:

firewalld not installed

OS firewall not blocking traffic

4. Public IP and DNS Check
Confirmed EC2 instance had:

Public IPv4 address assigned

VPC DNS hostnames enabled

Instance was rebooted after enabling DNS hostnames.

5. Network ACL (NACL) Investigation
Reviewed Network ACL associated with public-subnet.

Inbound rules:

Rule 100 → Allow ALL traffic from 0.0.0.0/0

Outbound rules:

Rule 100 → Allow ALL traffic to 0.0.0.0/0

Key realization:

Network ACLs are stateless

Both inbound and outbound rules must allow traffic

Result:

NACL configuration was correct

6. Client-Side Network Testing
Tested from local machine:

bash
Copy code
curl http://98.92.235.155
The request succeeded, confirming:

AWS networking was correct

The browser/network was the final blocker

After retrying with correct protocol and network, the page loaded successfully.

Final Verification (Success)
Browser output:

text
Copy code
Chapter 2 success – EC2 is working
This confirms:

Internet → AWS → EC2 → nginx → response path works end-to-end

Exiting the EC2 Instance
To exit the EC2 SSH session safely:

bash
Copy code
exit
This:

Closes the SSH session

Does NOT stop or terminate the EC2 instance

Leaves nginx running

Alternative:

text
Copy code
Ctrl + D
Key Learnings from Chapter 2
EC2 is a virtual server in AWS

SSH provides secure remote access

Security Groups are stateful

Network ACLs are stateless

Both inbound and outbound rules matter

OS-level services must be running for ports to respond

AWS networking issues must be debugged layer by layer

Browser/network issues can mimic AWS failures