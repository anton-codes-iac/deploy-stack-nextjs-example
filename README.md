# my-aws-next-app - Cloud Infrastructure

This project was provisioned by `create-cloud-stack`. It contains a production-ready AWS ECS Fargate architecture and a GitHub Actions deployment pipeline.

## 💰 Cost Estimate & Disclaimer

This infrastructure provisions a highly available Application Load Balancer (ALB) and an ECS Fargate container (Size: **Micro (0.25 vCPU, 512MB RAM)**).
    
* **Estimated Monthly Cost:** ~$25.00 / month
* *Note: AWS bills by the hour. If you destroy this stack after a few hours of testing, it will cost less than $0.20.*

> **⚠️ DISCLAIMER:** This cost is a rough estimate. AWS pricing changes and varies by region. **You are solely responsible for all AWS charges incurred by deploying this infrastructure.** The creators of `create-cloud-stack` are not liable for unexpected cloud costs, compromised credentials, or runaway billing. Always monitor your AWS Billing Dashboard and set up budget alerts.

## 🚀 Deployment Guide

1. **Initial Provisioning:**
   ```bash
   cd terraform
   terraform init
   terraform apply
   ```

2. **Push Secrets (Optional):**
   If your application requires environment variables, create a local `.env` file and sync it directly to AWS Secrets Manager:
   ```bash
   npx create-cloud-stack secrets push .env
   ```

3. **Automated CI/CD (Keyless via OIDC):**
   Push this repository to GitHub. Your deployment pipeline uses AWS IAM OpenID Connect (OIDC) to authenticate securely with temporary credentials—**no long-lived AWS secret keys are required in GitHub Secrets**. Every push to `main` will automatically build, package, and deploy your application.

### ⚠️ Troubleshooting: OIDC Provider Already Exists
AWS only permits one GitHub Actions OIDC provider per AWS account. If `terraform apply` fails with an `EntityAlreadyExists` error regarding the OIDC provider, it indicates GitHub Actions was previously configured in this account.

**The Fix:**
Open `terraform/oidc.tf` and update the default value of `create_oidc_provider` to `false`:
```hcl
variable "create_oidc_provider" {
  type    = bool
  default = false # <--- Change this from true to false
}
```
Re-run `terraform apply` to link directly to your existing provider.

## 🛑 Safe Teardown (Destroying the Stack)

If you are done testing and want to stop all AWS billing, you must destroy the infrastructure. 

Because our Terraform configuration is set to force-delete the ECR image repository (even if images are present), teardown is a single, clean command:

```bash
cd terraform
terraform destroy
```
*Type `yes` when prompted. This will permanently delete the Load Balancer, ECS cluster, log groups, and associated networking components.*