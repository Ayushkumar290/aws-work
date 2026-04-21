# ☁️ Cloud Computing Practicals
### B.Sc. (H) Computer Science — DSC18 / GE7d / DSE8e (NEP UGCF 2022)

---

## 📋 Table of Contents

1. [Introduction to Cloud Platforms](#1-introduction-to-cloud-platforms)
2. [Launch Your First Amazon EC2 Instance](#2-launch-your-first-amazon-ec2-instance)
3. [Set Up a VPC](#3-set-up-a-vpc)
4. [Configure Auto Scaling and Load Balancing](#4-configure-auto-scaling-and-load-balancing)
5. [Deploying a Static Website on the Cloud](#5-deploying-a-static-website-on-the-cloud)
6. [Monitor Resources Using AWS CloudWatch](#6-monitor-resources-using-aws-cloudwatch)
7. [Install OpenStack](#7-install-openstack)
8. [Launch Your First Instance (OpenStack)](#8-launch-your-first-instance-openstack)
9. [Set Up Networking (OpenStack Neutron)](#9-set-up-networking-openstack-neutron)
10. [Cloud Security](#10-cloud-security)

---

## 1. Introduction to Cloud Platforms

**Objective:** Familiarize students with cloud platforms and their interfaces.

### Steps

1. Create free-tier accounts on the following platforms:
   - [AWS Free Tier](https://aws.amazon.com/free/)
   - [Azure Free Account](https://azure.microsoft.com/en-us/free/)
   - [GCP Free Tier](https://cloud.google.com/free)

2. Explore the dashboards and identify key services:
   - **Compute** — EC2 (AWS), Virtual Machines (Azure), Compute Engine (GCP)
   - **Storage** — S3 (AWS), Blob Storage (Azure), Cloud Storage (GCP)
   - **Networking** — VPC, Load Balancers, DNS

3. Use the pricing calculators to estimate costs:
   - [AWS Pricing Calculator](https://calculator.aws/)
   - [Azure Pricing Calculator](https://azure.microsoft.com/en-us/pricing/calculator/)
   - [GCP Pricing Calculator](https://cloud.google.com/products/calculator)

### Expected Outcome
Students will be comfortable navigating cloud consoles and understand the core service categories offered by major providers.

---

## 2. Launch Your First Amazon EC2 Instance

**Objective:** Deploy a virtual machine on AWS using Amazon EC2.

### Prerequisites
- AWS account with free-tier access
- Basic understanding of SSH

### Steps

1. Log in to the **AWS Management Console** and navigate to **EC2**.
2. Click **Launch Instance** and provide a name for your instance.
3. Select a pre-configured AMI — use **Amazon Linux 2** (free-tier eligible).
4. Choose the **t2.micro** instance type (free-tier eligible).
5. Create or select an existing **Key Pair** (`.pem` file) for SSH access — download and store it securely.
6. Under **Security Groups**, add an inbound rule to allow **SSH (port 22)** from your IP.
7. Launch the instance and wait for it to reach the **Running** state.
8. Connect via SSH:
   ```bash
   chmod 400 your-key.pem
   ssh -i "your-key.pem" ec2-user@<your-public-ip>
   ```

### Expected Outcome
A running Linux virtual machine accessible via SSH on AWS.

---

## 3. Set Up a VPC

**Objective:** Create and configure a Virtual Private Cloud (VPC).

### Prerequisites
- AWS account
- Completed Practical 2 (EC2 basics)

### Steps

1. Navigate to **VPC Dashboard** in the AWS Console and click **Create VPC**.
2. Define a CIDR block (e.g., `10.0.0.0/16`).
3. Create two subnets:
   - **Public Subnet** — e.g., `10.0.1.0/24`
   - **Private Subnet** — e.g., `10.0.2.0/24`
4. Create and attach an **Internet Gateway (IGW)** to your VPC.
5. Update the **Route Table** for the public subnet to route `0.0.0.0/0` traffic through the IGW.
6. Launch an EC2 instance in the **public subnet** (for internet-accessible services).
7. Launch another EC2 instance in the **private subnet** (for internal services).
8. Set up a **NAT Gateway** in the public subnet and update the private subnet's route table to allow outbound internet access through it.

### Network Diagram
```
Internet
    │
[Internet Gateway]
    │
[Public Subnet]  ──── EC2 (Web Server)
    │
[NAT Gateway]
    │
[Private Subnet] ──── EC2 (Database / Internal)
```

### Expected Outcome
A functional VPC with public/private subnet isolation, internet access for both subnets in the appropriate direction.

---

## 4. Configure Auto Scaling and Load Balancing

**Objective:** Set up an auto-scaling group and load balancer to handle variable traffic.

### Prerequisites
- Functional VPC from Practical 3
- At least two subnets in different Availability Zones

### Steps

1. **Create a Launch Template:**
   - Navigate to **EC2 → Launch Templates** and create one with your desired AMI and instance type.

2. **Create an Auto Scaling Group (ASG):**
   - Navigate to **EC2 → Auto Scaling Groups → Create**.
   - Attach the launch template created above.
   - Set the desired/minimum/maximum capacity (e.g., 1 / 1 / 3).

3. **Configure Scaling Policy:**
   - Add a **Target Tracking Policy**.
   - Set metric: **CPU Utilization > 70%** → scale out.

4. **Create an Application Load Balancer (ALB):**
   - Navigate to **EC2 → Load Balancers → Create**.
   - Choose **Application Load Balancer**, internet-facing.
   - Register your ASG instances as the **Target Group**.

5. **Test Auto Scaling:**
   - SSH into an instance and simulate CPU load:
     ```bash
     sudo yum install stress -y
     stress --cpu 4 --timeout 300
     ```
   - Observe new instances launching in the ASG activity log.

### Expected Outcome
Traffic automatically distributed across instances, with new instances launching under high load and terminating when load drops.

---

## 5. Deploying a Static Website on the Cloud

**Objective:** Host a static website using cloud object storage services.

### Prerequisites
- Account on any one of: AWS, Azure, or GCP
- A static website (HTML/CSS/JS files)

### Option A — AWS S3

1. Create an **S3 bucket** with a globally unique name.
2. Disable **"Block all public access"** for the bucket.
3. Enable **Static Website Hosting** under Bucket Properties.
4. Upload your `index.html` and other assets.
5. Add the following **Bucket Policy** to make content public:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [{
       "Effect": "Allow",
       "Principal": "*",
       "Action": "s3:GetObject",
       "Resource": "arn:aws:s3:::your-bucket-name/*"
     }]
   }
   ```
6. Access your website via the S3 endpoint URL provided in the console.

### Option B — Azure Blob Storage

1. Create a **Storage Account** in the Azure Portal.
2. Enable **Static Website** under the Data Management section.
3. Upload files to the `$web` container.
4. Access the website via the **Primary Endpoint** URL.

### Option C — GCP Cloud Storage

1. Create a **Cloud Storage bucket** with a unique name.
2. Set the access control to **Uniform** and make objects public.
3. Upload your website files.
4. Set the **Main page suffix** to `index.html`.
5. Access via `https://storage.googleapis.com/<bucket-name>/index.html`.

### Expected Outcome
A publicly accessible static website served from cloud storage with no server management required.

---

## 6. Monitor Resources Using AWS CloudWatch

**Objective:** Use CloudWatch to monitor AWS resources and send automated alerts.

### Prerequisites
- A running EC2 instance
- AWS SNS access

### Steps

1. **Enable CloudWatch Metrics:**
   - Navigate to **CloudWatch → Metrics → EC2**.
   - Select your instance and view **CPUUtilization** metrics.

2. **Create a CloudWatch Alarm:**
   - Go to **CloudWatch → Alarms → Create Alarm**.
   - Select metric: `EC2 → Per-Instance Metrics → CPUUtilization`.
   - Set condition: **Greater than 70%** for 2 consecutive periods.

3. **Configure SNS Notification:**
   - Create a new **SNS Topic** (e.g., `cpu-alert-topic`).
   - Add your email as a subscriber and confirm the subscription email.
   - Attach this SNS topic to your CloudWatch Alarm.

4. **Test the Setup:**
   - SSH into your EC2 instance and simulate high CPU:
     ```bash
     sudo yum install stress -y
     stress --cpu 2 --timeout 180
     ```
   - Wait for the alarm to trigger and check your email for the notification.

### Expected Outcome
An automated alert system that notifies you via email when EC2 CPU usage crosses the defined threshold.

---

## 7. Install OpenStack

**Objective:** Set up a local OpenStack environment for hands-on cloud infrastructure practice.

### Prerequisites
- A machine with at least **8 GB RAM**, **50 GB disk**, and a **multi-core CPU**
- Ubuntu 22.04 LTS (recommended)
- Internet access

### Steps

**Using DevStack (recommended for learning):**

1. Create a new user for DevStack:
   ```bash
   sudo useradd -s /bin/bash -d /opt/stack -m stack
   echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
   sudo -u stack -i
   ```

2. Clone the DevStack repository:
   ```bash
   git clone https://opendev.org/openstack/devstack
   cd devstack
   ```

3. Create a `local.conf` file:
   ```ini
   [[local|localrc]]
   ADMIN_PASSWORD=secret
   DATABASE_PASSWORD=$ADMIN_PASSWORD
   RABBIT_PASSWORD=$ADMIN_PASSWORD
   SERVICE_PASSWORD=$ADMIN_PASSWORD
   ```

4. Run the installer (this may take 20–40 minutes):
   ```bash
   ./stack.sh
   ```

5. Access the **Horizon Dashboard** at `http://<your-ip>/dashboard`.

### Expected Outcome
A fully functional single-node OpenStack installation accessible via the Horizon web UI and CLI.

---

## 8. Launch Your First Instance (OpenStack)

**Objective:** Create a virtual machine (VM) using OpenStack.

### Prerequisites
- OpenStack installed (Practical 7)
- Access to Horizon Dashboard or OpenStack CLI

### Steps

1. **Create a Project and Assign Roles:**
   - Go to **Identity → Projects → Create Project**.
   - Navigate to **Identity → Users** and assign the user to the project with the `member` role.

2. **Upload a Cloud Image:**
   - Download the Ubuntu cloud image:
     ```bash
     wget https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img
     ```
   - Go to **Compute → Images → Create Image** in Horizon and upload the file.

3. **Define a Flavor:**
   - Navigate to **Admin → Compute → Flavors → Create Flavor**.
   - Set vCPUs, RAM (e.g., 1 vCPU, 512 MB RAM, 5 GB disk).

4. **Launch the Instance:**
   - Go to **Compute → Instances → Launch Instance**.
   - Select the uploaded image, your flavor, and a network.
   - Click **Launch** and wait for the instance to reach **Active** status.

   Or via CLI:
   ```bash
   openstack server create \
     --image ubuntu-22.04 \
     --flavor m1.small \
     --network private-net \
     my-first-instance
   ```

### Expected Outcome
A running VM visible in the Horizon dashboard and accessible via its assigned IP address.

---

## 9. Set Up Networking (OpenStack Neutron)

**Objective:** Configure OpenStack Neutron to provide networking for instances.

### Prerequisites
- OpenStack installed with Neutron enabled
- At least one running instance (Practical 8)

### Steps

1. **Create a Private Network:**
   ```bash
   openstack network create private-net
   openstack subnet create --network private-net \
     --subnet-range 192.168.10.0/24 private-subnet
   ```

2. **Create a Public Network (Admin only):**
   ```bash
   openstack network create --external public-net
   openstack subnet create --network public-net \
     --subnet-range 203.0.113.0/24 \
     --no-dhcp public-subnet
   ```

3. **Create and Attach a Router:**
   ```bash
   openstack router create my-router
   openstack router set --external-gateway public-net my-router
   openstack router add subnet my-router private-subnet
   ```

4. **Assign a Floating IP to an Instance:**
   ```bash
   openstack floating ip create public-net
   openstack server add floating ip my-first-instance <floating-ip>
   ```

5. Verify connectivity by pinging the floating IP from the host machine.

### Network Architecture
```
[External Network / Internet]
          │
     [Router]
          │
  [Private Network] ──── [Instance 1]
                    ──── [Instance 2]
```

### Expected Outcome
Instances on the private network are reachable externally via floating IPs, with routing handled by Neutron.

---

## 10. Cloud Security

**Objective:** Understand and implement fundamental security practices in the cloud.

### Prerequisites
- AWS account (or equivalent Azure/GCP account)
- Completed Practicals 2 and 3

### Steps

1. **Implement IAM Roles and Policies:**
   - Navigate to **IAM → Policies → Create Policy**.
   - Grant only the minimum required permissions (principle of least privilege).
   - Example — read-only S3 access:
     ```json
     {
       "Version": "2012-10-17",
       "Statement": [{
         "Effect": "Allow",
         "Action": ["s3:GetObject", "s3:ListBucket"],
         "Resource": "*"
       }]
     }
     ```

2. **Create and Assign Least-Privilege Roles:**
   - Go to **IAM → Roles → Create Role**.
   - Attach your custom policy to the role.
   - Assign the role to an EC2 instance or IAM user.

3. **Enable Storage Encryption:**
   - Navigate to **S3 → Select Bucket → Properties**.
   - Enable **Server-Side Encryption** using AWS-managed keys (SSE-S3) or KMS (SSE-KMS).
   - For EBS volumes, enable encryption during EC2 instance launch.

4. **Configure and Test Firewall Rules (Security Groups):**
   - Navigate to **EC2 → Security Groups → Create Security Group**.
   - Add inbound rules — allow only required ports (e.g., HTTP: 80, HTTPS: 443).
   - Deny all other inbound traffic by default (AWS denies by default; verify no `0.0.0.0/0` on sensitive ports like 22 except from trusted IPs).
   - Test by attempting to connect on a blocked port — connection should time out.

### Security Best Practices Checklist

- [ ] MFA enabled on root and IAM accounts
- [ ] No access keys attached to the root account
- [ ] All S3 buckets have appropriate access controls
- [ ] Encryption enabled for storage (S3, EBS)
- [ ] Security Groups follow the principle of least privilege
- [ ] CloudTrail enabled for audit logging

### Expected Outcome
Hands-on familiarity with IAM, encryption, and network security controls — the three pillars of cloud security.

---

---

*Guidelines based on NEP UGCF 2022 — Effective from Academic Year 2024-25*
