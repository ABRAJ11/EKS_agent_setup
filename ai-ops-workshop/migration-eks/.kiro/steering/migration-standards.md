# Container Migration Standards (EKS)

When migrating workloads from EC2 to Amazon EKS:

- Analyze the existing Docker Compose file before making changes
- Create ECR repositories in the same region as the target cluster
- Convert Docker Compose services to Kubernetes Deployments and Services
- Use ConfigMaps for environment variables
- Configure liveness and readiness probes
- Use Ingress for external access via ALB
