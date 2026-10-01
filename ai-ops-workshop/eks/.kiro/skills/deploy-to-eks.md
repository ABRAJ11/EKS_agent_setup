---
name: Deploy to EKS
description: Deploy a containerized application to Amazon EKS Auto Mode
---

# Deploy to EKS

Follow these steps to deploy an application to the existing EKS cluster:

1. Build and push the container image to Amazon ECR
2. Create a Kubernetes Deployment with health checks and resource limits
3. Create a Kubernetes Service for internal communication
4. Create an Ingress resource following https://docs.aws.amazon.com/eks/latest/userguide/auto-configure-alb.html for ALB-based external access
5. Verify the deployment is healthy with `kubectl get pods` and `kubectl get ingress`
6. Provide the ALB DNS name for accessing the application
