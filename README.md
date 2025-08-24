# Cloudstatic-project
Hosting a static site with S3+IAM (Users and Policies). This project demonstrates **AWS static website hosting, IAM access control, and VPC design**.  

It covers:
- Hosting a static website on **Amazon S3** with CloudFront distribution.
- Creating IAM users with **custom S3 and VPC permissions**.
- Designing a **secure VPC** with public, private, and a DB subnets and security groups.

---

## PROJECT STRUCTURE
Cloudstatic-project/

│── README.md  

│── policies/ <br>
│   ├── s3-policy.json      
│   ├── vpc-readonly-policy.json  

│── screenshots/ 

---

## Static Website Hosting (S3 + IAM)

### 🪣 Buckets
- **cloudstatic-assignment**
  - Public access enabled
  - Static Website Hosting turned on
- **cloudstatic-private-bucket**
  - Blocked public access
- **cloudstatic-visible-only**
  - Blocked public access

### 🌐 Website Hosting
- Uploaded `index.html` 
- Configured **bucket policy** for public read access:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::cloudstatic-assignment/*"
    }
  ]
}
```
### 📦 CLOUDFRONT
-The distribution was created with Cloudstatic-assignment endpoint as the origin.
-Enabled HTTP → HTTPS redirection.
-Attached a Custom SSL certificate.

---
### 📋 IAM User & POLICIES
#### <ins> IAM </ins>
Created an IAM user: cloudstatic-user
#### <ins> S3 Policy. </ins> 
Describing **ACTIONS** for the buckets

- S3 List on all buckets.
- GetObject on assignment bucket.
- GetObject + PutObject on private bucket.

❌ No access to visible-only bucket.
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:ListBucket",
            "Resource": [
                "arn:aws:s3:::cloudstatic-assignment",
                "arn:aws:s3:::cloudstatic-private-bucket",
                "arn:aws:s3:::cloudstatic-visible-only"
            ]
        },
        {
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::cloudstatic-assignment/*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::cloudstatic-private-bucket/*"
        }
    ]
}
```
#### <ins> VPC Policy. </ins>
Describing VPC design
- **Created a VPC**
  - Name: cloudlaunch-vpc
  - CIDR: /16
- **Attached Subnets to the VPC**
  - Public Subnet (10.0.1.0/24) → Web.
  - App Subnet (10.0.2.0/24) → Application (private) servers.
  - DB Subnet (10.0.3.0/28) → Database servers.
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeVpcs",
                "ec2:DescribeSubnets",
                "ec2:DescribeRouteTables",
                "ec2:DescribeSecurityGroups"
            ],
            "Resource": "*"
        }
    ]
}
```
## 🛡️ NETWORK & SECURITY
#### <ins> Route Table.</ins>
-Public Route Table (cloudstatic-public-rt) → Associated with Public Subnet.
   -Route: 0.0.0.0/0 → IGW. <br>
-Private Route Table (cloudstatic-private-rt) → Associated with App Subnet.<br>
-DB Route Table (cloudstatic-db-rt) → Associated with DB Subnet. <br>
❌ No internet access to Private Route Table & DB Route Table

#### <ins> Security Groups. </ins>
-Cloudstatic-app-sg → Allow HTTP (80) from within VPC main CIDR
-Cloudstatic-db-sg → Allow MySQL (3306) only from App Subnet.

---
### 🔗 URLS
-S3 Website URL:
   - https://cloudstatic-assignment.s3.eu-west-2.amazonaws.com/index.html.

-Cloudfront URL:
   - d1o7ukoex2422u.cloudfront.net
