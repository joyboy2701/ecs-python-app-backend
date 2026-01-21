# Backend (Storage Service) – CI/CD with GitHub Actions, ECR, and ECS

This repository contains the **backend storage service**, packaged as a Docker image and automatically deployed to **AWS ECS** using **GitHub Actions**. The deployment is fully automated and triggered only when code is merged into the `master` branch via an approved Pull Request.

---

## 📁 Project Structure

```
.
├── storage
│   ├── app.py          # Backend (storage/worker) application code
│   └── Dockerfile      # Docker image definition
└── .github
    └── workflows
        └── build_push.yml   # GitHub Actions CI/CD pipeline
```

---

## 🚀 What This Pipeline Does

The GitHub Actions workflow automates the entire backend deployment process:

1. Runs **only when a Pull Request to `master` is merged and closed**
2. Builds a Docker image from the `storage/` directory
3. Pushes the image to **Amazon ECR**
4. Forces a new deployment of the backend service on **Amazon ECS**

This ensures that every approved code change is automatically deployed to production.

---

## 🔁 CI/CD Trigger Logic

The workflow is triggered by this GitHub event:

```yaml
on:
  pull_request:
    types: [closed]
    branches:
      - master
```

To avoid deployments on unmerged PRs, the job is conditionally executed:

```yaml
if: github.event.pull_request.merged == true
```

✔ Pipeline runs only for **approved & merged PRs**

---

## 🔐 Secure AWS Authentication (OIDC)

The pipeline uses **GitHub OpenID Connect (OIDC)** to authenticate with AWS. No AWS access keys are stored in GitHub.

An IAM role is assumed dynamically at runtime.

### Required GitHub Secrets

| Secret Name    | Description                                                            |
| -------------- | ---------------------------------------------------------------------- |
| `AWS_ROLE_ARN` | IAM role assumed by GitHub Actions                                     |
| `REGION`       | AWS region (e.g. `us-east-1`)                                          |
| `ECR_REGISTRY` | ECR registry URL (e.g. `123456789012.dkr.ecr.us-east-1.amazonaws.com`) |

---

## 🐳 Docker Image Build & Push (ECR)

The Docker image is built using the backend Dockerfile:

```bash
docker build -t storage-service:latest ./storage
```

The image is then tagged and pushed to Amazon ECR:

```bash
docker tag storage-service:latest <ECR_REGISTRY>/storage-service:latest
docker push <ECR_REGISTRY>/storage-service:latest
```

The `latest` tag is used by the ECS service.

---

## ☁️ ECS Service Deployment

After the image is pushed to ECR, the ECS service is updated:

```bash
aws ecs update-service \
  --cluster ecs-cluster \
  --service worker-service \
  --force-new-deployment
```

This forces ECS to:

* Pull the new Docker image from ECR
* Restart running tasks
* Deploy the updated backend service automatically

---

## 🔄 End-to-End Deployment Flow

1. Create a **new feature branch** from `master`
2. Make backend code changes in `storage/`
3. Commit and push the branch
4. Open a **Pull Request to `master`**
5. PR is **reviewed and approved**
6. PR is **merged manually**
7. GitHub Actions pipeline starts automatically
8. Docker image is built and pushed to ECR
9. ECS backend service is redeployed with the new image

---

## ✅ Advantages of This Setup

* Fully automated backend deployments
* Secure authentication using OIDC
* No manual Docker or ECS steps
* Controlled releases through PR approvals
* Consistent and repeatable deployments

---

## 📌 Important Notes

* The ECS task definition must reference the `latest` image tag
* The IAM role must allow:

  * ECR image push permissions
  * ECS service update permissions
* ECR repositories must already exist

---

## 👤 Author

**Zubair Liaqat**

---

This backend CI/CD pipeline pairs cleanly with the frontend pipeline, giving you a complete automated deployment strategy across services.
