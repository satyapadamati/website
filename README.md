# 🚀 AWS CI/CD Pipeline: GitHub → ECS Fargate with Custom Domain

This project demonstrates a complete DevOps pipeline to automatically build, deploy, and host a containerized web application using AWS services.

## 🔧 Tech Stack & Services

- **Source Control**: GitHub
- **CI/CD**: AWS CodePipeline + CodeBuild
- **Container Orchestration**: Amazon ECS (Fargate launch type)
- **Image Registry**: Amazon ECR
- **Networking**: Application Load Balancer (ALB)
- **Domain Hosting**: GoDaddy
- **Auto Scaling**: ECS service auto-scaling based on traffic

---

## 📌 Project Workflow

GitHub → AWS CodePipeline → AWS CodeBuild → Amazon ECR → Amazon ECS → ALB → GoDaddy Domain


1. **Push to GitHub**: Any change in the main branch triggers the pipeline.
2. **AWS CodePipeline**: Detects changes and starts a new build.
3. **AWS CodeBuild**:
   - Builds the Docker image.
   - Pushes it to Amazon ECR.
   - Generates `imagedefinitions.json` for ECS.
4. **Amazon ECS**: 
   - Pulls the new image.
   - Updates the running task in the Fargate service.
   - Auto-scales based on CPU/memory metrics.
5. **Application Load Balancer**:
   - Distributes traffic to running tasks.
   - Provides a public endpoint.
6. **Custom Domain (GoDaddy)**:
   - The ALB DNS is mapped to a domain using GoDaddy DNS settings.

---

## 📁 Repository Structure

├── Dockerfile # Container configuration
├── buildspec.yml # CodeBuild build instructions
├── imagedefinitions.json # (Auto-generated during build)
└── application/ # Your web app source code


---

## ⚙️ buildspec.yml (CI Instructions)

```yaml
version: 0.2

env:
  variables:
    AWS_ACCOUNT_ID: "<your-account-id>"
    REPOSITORY_NAME: "website"
    CONTAINER_NAME: "your-container-name"
    CLUSTER_NAME: "your-ecs-cluster"
    AWS_DEFAULT_REGION: "eu-north-1"
    IMAGE_TAG: "latest"

phases:
  pre_build:
    commands:
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
      - REPOSITORY_URI=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$REPOSITORY_NAME
  build:
    commands:
      - docker build -t $REPOSITORY_NAME:$IMAGE_TAG .
      - docker tag $REPOSITORY_NAME:$IMAGE_TAG $REPOSITORY_URI:$IMAGE_TAG
  post_build:
    commands:
      - docker push $REPOSITORY_URI:$IMAGE_TAG
      - printf '[{"name":"%s","imageUri":"%s"}]' $CONTAINER_NAME $REPOSITORY_URI:$IMAGE_TAG > imagedefinitions.json

artifacts:
  files: imagedefinitions.json

🌐 Domain Setup (GoDaddy)
Go to your GoDaddy DNS settings.

Add a CNAME or A Record pointing to your Application Load Balancer DNS name (found in the EC2 → Load Balancers section).

Ensure your domain resolves to the ALB.

📈 Auto Scaling
Configured based on ECS service metrics:

CPU Utilization

Memory Utilization

ECS adds/removes tasks dynamically based on traffic load.

✅ Future Improvements
HTTPS using ACM + ALB listener rules

Add CloudWatch monitoring dashboards

Add health checks and rollback strategy

🙌 Author
Satyakiran Padamati
🔗 LinkedIn
💻 GitHub

📜 License
MIT License – feel free to use, fork, and improve!
---


