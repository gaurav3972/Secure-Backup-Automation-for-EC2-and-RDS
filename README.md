# 🔐Secure Backup Automation for EC2 and RDS
### 📝 **Introduction :**

This project is about learning how to back up important AWS resources using a service called **AWS Backup**. Backups help keep your data safe in case something goes wrong, like a server crash or accidental deletion. In this project, we will create and set up two resources: an **EC2 instance**, which runs a web server, and an **RDS database**, which stores sample data. Then, we will use AWS Backup to create a **backup plan** that automatically saves copies of these resources. We will test the backup by running it manually and checking that the data is saved properly. This project helps understand how to protect data in the cloud using simple and reliable AWS tools.

## **Objective:**

The objective of this project is to demonstrate the configuration and management of **automated backup and recovery** for AWS resources using **AWS Backup**, a fully managed backup service. The focus is on protecting compute and database services—specifically, **Amazon EC2** and **Amazon RDS**. This project aims to achieve the following:

* Launch and configure a Linux-based EC2 instance with web server and test content.
* Launch and configure an RDS instance with a sample test database.
* Set up AWS Backup service, including Backup Vault and Backup Plan.
* Assign both EC2 and RDS to the backup plan.
* Validate the configuration by performing on-demand backups and viewing recovery points.

---

## 🏗️ **1. Infrastructure Setup**

### 1.1 Launch an EC2 Instance (Web Server)

#### Steps:

1. Log in to the AWS Management Console.
2. Navigate to **EC2 Dashboard** → Click **Launch Instance**.
3. Configure the instance:

   * **Name**: `BackupTest-EC2`
   * **AMI**: Amazon Linux 2 (Free Tier eligible) or Ubuntu Server 22.04
   * **Instance type**: `t2.micro` (Free Tier)
   * **Key pair**: Create or choose an existing key pair
   * **Network settings**:

     * Enable **HTTP (port 80)** and **SSH (port 22)** in the security group.
     * Attach the instance to the default VPC and subnet.
4. Launch the instance.

#### Install Apache and Upload Sample Web Content:

Once the instance is running:

```bash
# Connect via SSH
ssh -i "your-key.pem" ec2-user@<Public-IP>

# Update the system and install Apache
sudo yum update -y
sudo yum install -y httpd

# Start and enable the web server
sudo systemctl start httpd
sudo systemctl enable httpd

# Add sample web page
echo "<html><h1>This is a backup test page for EC2</h1></html>" | sudo tee /var/www/html/index.html
```

#### Optional:

Take a snapshot of the instance after configuration for quick rollback or reuse.

---

### 1.2 Launch an RDS Instance (Database Server)

#### Steps:

1. Go to **RDS Dashboard** → Click **Create Database**.
2. Select the following settings:

   * **Engine type**: MySQL or PostgreSQL
   * **Template**: Free tier
   * **DB instance identifier**: `backup-test-db`
   * **Master username**: `admin`
   * **Master password**: Choose a secure password
   * **DB instance class**: `db.t3.micro`
   * **Storage**: Use default (20 GB SSD)
   * Enable **public access** (for test only)
   * Add **inbound rules** in security group for port **3306** (MySQL) or **5432** (PostgreSQL)

#### Create a Test Database and Table:

Using a database client like DBeaver or MySQL Workbench:

```sql
CREATE DATABASE testdb;
USE testdb;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50),
    email VARCHAR(100)
);

INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com'), ('Bob', 'bob@example.com');
```

---

## 🔒 **2. AWS Backup Setup**

### 2.1 Create Backup Vault

1. Navigate to **AWS Backup** → **Backup Vaults** → **Create Backup Vault**.
2. Provide a name like `MyBackupVault`.
3. Use default encryption settings (AWS-managed key).
4. Optionally enable notifications using Amazon SNS.

### 2.2 Create Backup Plan

1. Go to **Backup Plans** → **Create Backup Plan** → Choose **Build a new plan**.
2. Configuration:

   * **Plan name**: `EC2-RDS-BackupPlan`
   * **Backup rule name**: `DailyBackup`
   * **Backup frequency**: Daily
   * **Backup window**: Use default or customize
   * **Lifecycle rules**:

     * Retention: 7 days (extendable)
     * Transition to cold storage: Optional
   * **Backup vault**: Select `MyBackupVault`

### 2.3 Assign Resources to Backup Plan

1. Go to **Assignments** → **Create assignment**.
2. Assignment name: `EC2-RDS-Assignment`
3. Assign by:

   * **Resource ID**:

     * Select EC2 instance
     * Select RDS database
   * OR use **tags**:

     * Example: Tag both resources with `Key=Backup` and `Value=true`
     * In the backup plan, choose to include resources by tag filter

📌 *Important: Make sure your IAM role has permissions to manage backups across EC2 and RDS.*

---

## 📊 **3. Validation and Reporting**

### 3.1 Trigger On-Demand Backups

1. Go to **Protected Resources** under AWS Backup.
2. Select EC2 and RDS resources.
3. Click **Actions → Backup Now**.
4. Monitor progress under **Backup Jobs**.

### 3.2 Monitor Backup Jobs

1. Go to **Backup Jobs** section.
2. Confirm:

   * EC2 backup job is listed with resource type `EC2`
   * RDS backup job is listed with resource type `RDS`
   * Both show status as `Completed`
3. Review job logs for additional verification.

### 3.3 View Recovery Points

1. Navigate to **Backup Vaults → MyBackupVault**.
2. Click on **Recovery Points**.
3. You will see:

   * Recovery point for EC2 (AMI/Snapshot)
   * Recovery point for RDS (DB Snapshot)
4. Note creation time, size, and expiration dates.
---

## 📂 **4. Additional Notes**

* **IAM Roles**: Ensure AWS Backup service has correct IAM permissions to manage EC2 and RDS backups.
* **Cold Storage**: For long-term retention, you can enable lifecycle rules to move backups to cold storage.
* **Cross-Region Backup**: AWS Backup also supports cross-region copy (useful for DR).
* **Security**: Enable encryption and monitor access logs using AWS CloudTrail.

---
### 📄 **Summary :**

This project focused on implementing a centralized and automated backup solution using AWS Backup for critical cloud infrastructure components—Amazon EC2 and Amazon RDS. The process began by launching an EC2 instance with a web server and test content, and an RDS instance running a MySQL or PostgreSQL database with sample data. These resources were then protected through a backup strategy involving the creation of a custom backup vault and a backup plan configured with daily frequency and a defined retention period.

