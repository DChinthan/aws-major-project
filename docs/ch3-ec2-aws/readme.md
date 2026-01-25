# AWS Major Project – Chapter 3

## Chapter Title

**Deploying a Spring Boot Backend to AWS EC2 (End‑to‑End)**

---

## 1. What Chapter 3 Is About (Big Picture)

In this chapter, we take a **Spring Boot application that works on our local machine** and deploy it to **AWS EC2**, making it accessible over the internet.

By the end of this chapter, we prove that:

* Our backend is **not just code**, but a **running cloud service**
* We understand how application code, networking, Linux, and AWS infrastructure connect together

This chapter is the **bridge between local development and real cloud deployment**.

---

## 2. Starting Point (What We Already Had)

Before starting Chapter 3, we already had:

* A Spring Boot project created using Spring Initializr
* Project runs successfully on `localhost:8080`
* A controller (`HealthController`) with endpoints:

  * `/health`
  * `/hello`

At this stage:

* The app works **only on the local machine**
* `localhost` means *this computer only* — no one else can access it

---

## 3. Key Concepts Introduced in This Chapter

### 3.1 Localhost vs Internet

* `localhost` → loopback address (127.0.0.1)
* Only the same machine can access it
* Cloud deployment requires listening on **real network interfaces**

---

### 3.2 JAR (Java ARchive)

A **JAR** is a packaged, deployable artifact.

In Spring Boot, a JAR contains:

* Compiled Java code
* All dependencies
* Embedded web server (Tomcat)
* Spring Boot bootstrap logic

**Mental model:**

> JAR = a suitcase containing your entire backend

This allows the same app to run:

* On your laptop
* On EC2
* On any JVM-enabled machine

---

### 3.3 EC2 (Elastic Compute Cloud)

EC2 is a **virtual Linux server** rented from AWS.

Important EC2 characteristics:

* You pay for time + resources
* It behaves like a real computer
* You access it using SSH

---

### 3.4 Public Subnet + Internet Gateway

To expose a backend to the internet:

* EC2 must be in a **public subnet**
* The subnet must have a route:

  * `0.0.0.0/0 → Internet Gateway`

**Mental model:**

* VPC = city
* Subnet = neighborhood
* IGW = highway to the internet

---

### 3.5 Security Groups (Firewall)

Security Groups control **what traffic is allowed**.

In this chapter we allowed:

* Port 22 (SSH) → from our IP
* Port 8080 (HTTP) → from anywhere

Even if EC2 is public:

> ❌ No SG rule = no access

---

## 4. Step‑by‑Step Breakdown (With Why)

---

### 4.1 Verifying the App Locally

Commands used:

```
./mvnw spring-boot:run
curl http://localhost:8080/health
curl http://localhost:8080/hello
```

**Why this matters:**

* Never deploy unverified code
* Confirms controllers and routing work

---

### 4.2 Building the JAR

Command:

```
./mvnw clean package
```

What happens internally:

* `clean` → removes old builds
* `package` → compiles + tests + builds JAR

Output:

* `target/api-0.0.1-SNAPSHOT.jar`

---

### 4.3 Making the App Network‑Accessible

File modified:

```
src/main/resources/application.properties
```

Configuration:

```
server.address=0.0.0.0
server.port=8080
```

**Why this is critical:**

* EC2 traffic does not come from `localhost`
* `0.0.0.0` means listen on all interfaces

---

### 4.4 Creating the EC2 Instance

Key decisions:

* Amazon Linux 2023
* t2.micro / t3.micro
* Public subnet
* Auto‑assign public IP enabled

**Why:**

* Public IP allows internet access
* Linux OS is lightweight and standard

---

### 4.5 Installing Java on EC2

Command:

```
sudo dnf install -y java-17-amazon-corretto
```

**Why:**

* Spring Boot requires a JVM
* Amazon Corretto is AWS‑supported

---

### 4.6 Copying the JAR to EC2

Command:

```
scp -i key.pem target/*.jar ec2-user@<EC2_IP>:/home/ec2-user/app.jar
```

**Why:**

* Secure file transfer over SSH
* Moves our deployable artifact to the server

---

### 4.7 Running the App on EC2

Command:

```
java -jar app.jar
```

What we observed:

* Tomcat starts on port 8080
* App logs confirm successful startup

This proved:

* The JAR is portable
* The app runs in the cloud

---

### 4.8 Background Execution (nohup)

Command:

```
nohup java -jar app.jar &
```

**Why:**

* Keeps app running after SSH disconnect

Limitation:

* No auto‑restart
* No startup on reboot

---

### 4.9 systemd Service (Production‑Style)

We created:

```
/etc/systemd/system/api.service
```

What systemd provides:

* Auto‑start on boot
* Auto‑restart on crash
* Centralized logs

**Mental model:**

> systemd = Linux service manager (autopilot)

---

## 5. Verifying Public Access

From local machine:

```
curl http://<EC2_PUBLIC_IP>:8080/health
curl http://<EC2_PUBLIC_IP>:8080/hello
```

Traffic flow:

* Laptop → Internet → IGW → EC2 → Spring Boot

This confirms:

* Networking is correct
* Firewall rules are correct
* App is reachable publicly

---

## 6. Cost Awareness (Very Important)

Resources that **cost money**:

* EC2 instances
* EBS volumes
* Elastic IPs (if unused)
* NAT Gateways
* Load Balancers

Resources that are **free to exist**:

* VPC
* Subnets
* Route tables
* Security Groups
* IGW

---

## 7. Teardown & Reset (End of Chapter 3)

To avoid charges and prepare for next chapter:

Deleted:

* EC2 instance
* EBS volumes
* Elastic IPs
* NAT Gateways
* Load Balancers
* VPC

Result:

* Clean AWS account
* Zero running cost
* Ready to rebuild from memory

---

## 8. What Chapter 3 Proves (Interview‑Level Summary)

After this chapter, you can confidently say:

* I can deploy a Spring Boot backend to AWS
* I understand EC2, networking, and Linux basics
* I know how public access actually works
* I can package, deploy, run, and manage services
* I understand AWS cost responsibility

---

## 9. What Comes Next (Chapter 4 Preview)

Chapter 4 will improve this deployment by adding:

* Better traffic handling (ALB or Nginx)
* Cleaner architecture
* Production‑style access patterns
* Possibly domains and HTTPS

---

**End of Chapter 3 – Revision Notes**
