Here's a  brief explanation  of the entire project 

---
Architecture Diagram

![image](https://github.com/user-attachments/assets/f476a6c7-d372-4c86-ae6f-025ef42e570e)


  🛠️  Project Overview: Deploying a Dockerized App to AWS ECS Fargate with Terraform 

I have built an  end-to-end infrastructure on AWS  using Terraform that:

1.  Defines the network (VPC, subnets, security groups, etc.) 
2.  Creates ECS Fargate resources 
3.  Deploys a containerized app (Docker image) 
4.  Exposes the app via an Application Load Balancer (ALB) 
5.  Uses GitHub Actions for CI/CD 
6.  Keeps secrets out of Git using `terraform.tfvars` and GitHub Secrets 

---

  🧱 Infrastructure Components

   1.  Networking 

 VPC:  A logically isolated AWS network (`10.0.0.0/16`)
 Subnets:  2 public subnets for high availability (spread across AZs)
 Internet Gateway & Routing:  Allows internet access to your ECS tasks
 Security Group:  Allowed inbound HTTP (port 80) and later port 3000 for the app

   2.  ECS (Elastic Container Service) 

 ECS Cluster:  Fargate launch type (no server management)
 Task Definition:  Defines your container (image, ports, logging)
 ECS Service:  Runs the task, integrates with the ALB

   3.  Application Load Balancer 

 ALB:  Routes internet traffic to your ECS service
 Target Group:  Uses `ip` target type (required for Fargate)
 Health Checks:  Ensures only healthy tasks receive traffic

---

  🚀 Application Deployment

  You built and pushed your  Dockerized app  to  Amazon ECR 
  The ECS task pulls the image from ECR and runs it
  ALB receives HTTP traffic and forwards to ECS task (on port 3000)
  You accessed your app using the  ALB DNS output 

---

  🔐 Secure & Automated

  Sensitive variables like `ecs_image_url` were moved to `terraform.tfvars` and excluded via `.gitignore`
 GitHub Actions  workflow:

    Checks Terraform format
    Initializes Terraform
    Runs a `terraform plan`
    Can be extended to run `apply` securely with secrets

---

  ✅ Outcome

You now have a  production-ready ECS Fargate deployment  with:

  Infrastructure as Code (Terraform)
  CI/CD via GitHub Actions
  Best practices for security and observability

---



# ECS Fargate Terraform Deployment

This project provisions an ECS Fargate cluster with an Application Load Balancer using Terraform. The app container is pulled from a pre-built ECR image and exposed to the internet.

---

  📁 Project Structure

 
ecs-fargate-terraform/
├── app/             		 #Dockerized application source code
├── main.tf				       # Main Terraform configuration
├── variables.tf         # Input variable definitions
├── outputs.tf           # Output values
├── terraform.tfvars     # (Excluded from Git) Sensitive variable values like image URL
├── .gitignore           # Ignored files/folders
├── .github/
│   └── workflows/
│       └── terraform.yml # GitHub Actions pipeline for Terraform
└── README.md            # Project documentation (this file)
 

---

  🚀 Deployment Steps

1.  Set your AWS credentials  (in your environment or GitHub secrets)
2.  Initialize Terraform 
     bash
    terraform init
     
3.  Apply the configuration 
     bash
    terraform apply -auto-approve
     
4.  Access the application 
   - Output: `alb_dns = "http://<your-load-balancer>"`

---

  🔐 Sensitive Variables

Create a `terraform.tfvars` file (excluded from Git) to hold values like:

 hcl
ecs_image_url = "<your_ecr_image_url>"
region        = "ap-south-1"
vpc_cidr      = "10.0.0.0/16"
public_subnet_cidrs = ["10.0.1.0/24", "10.0.2.0/24"]
 

 Note:  This file is included in `.gitignore`.

---

  🛠 GitHub Actions Workflow

The included GitHub Actions file runs `terraform fmt`, `validate`, and `plan` on every push.

 yaml
name: Terraform CI

on:
  push:
    branches:
      - main

jobs:
  terraform:
    name: Terraform Format, Validate, and Plan
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.6.0

      - name: Terraform Init
        run: terraform init

      - name: Terraform Format
        run: terraform fmt -check

      - name: Terraform Validate
        run: terraform validate

      - name: Terraform Plan
        run: terraform plan
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
 

Add your AWS credentials to GitHub Repository Secrets:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`

---

  📌 Notes
- Ensure the ECR image URL is correct and publicly accessible or in the same AWS account.
- Security group must allow inbound traffic on the container port (default 3000).
- `target_type = "ip"` is  required  for Fargate.

---

  🧹 Cleanup
 bash
terraform destroy -auto-approve
 

---

  🙌 Contributions
Feel free to fork, improve, and submit pull requests.

---

  📄 License
MIT
