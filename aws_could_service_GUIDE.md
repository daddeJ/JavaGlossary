# AWS-Based CI/CD Deployment Pipeline: Complete Guide

## Table of Contents
1. [Architecture Overview](#architecture-overview)
2. [Local Development Phase](#local-development-phase)
3. [Source Control Integration](#source-control-integration)
4. [Continuous Integration (CI)](#continuous-integration-ci)
5. [Database Management](#database-management)
6. [Container Registry & Orchestration](#container-registry--orchestration)
7. [Secrets Management](#secrets-management)
8. [Continuous Deployment (CD)](#continuous-deployment-cd)
9. [Monitoring & Health Checks](#monitoring--health-checks)
10. [Cost & Complexity Comparison](#cost--complexity-comparison)

---

## Architecture Overview

### AWS-Based Architecture Components

```
Developer → GitHub → GitHub Actions (CI) → ECR/S3 → ECS/RDS → ALB → Users
                ↓
            CodePipeline/CodeBuild (CD) → Secrets Manager → CloudWatch
```

**Intention:** Create a fully managed, scalable, and secure deployment pipeline that automates everything from code commit to production deployment while maintaining high availability and observability.

### Key AWS Services Used
- **GitHub Actions**: CI pipeline automation
- **ECR (Elastic Container Registry)**: Docker image storage
- **ECS Fargate**: Serverless container orchestration
- **RDS SQL Server**: Managed database
- **S3**: Static asset and script storage
- **ALB (Application Load Balancer)**: Traffic distribution
- **Secrets Manager**: Credential management
- **CloudWatch**: Monitoring and logging
- **CodePipeline/CodeBuild**: CD automation

---

## Local Development Phase

### Step 1: Local Development Environment

**AWS Approach:**
```bash
# Docker Compose for local development
docker-compose up -d
dotnet run --project src/API/ContactService.API.csproj
curl http://localhost:5000/health
```

**Intention:** 
- Maintain development-production parity using Docker
- Enable rapid iteration and testing
- Simulate production environment locally

**Non-AWS Alternative:**
```bash
# Traditional approach without containers
dotnet run
# Or using local SQL Server/PostgreSQL
sqlcmd -S localhost -E -i init.sql
```

**Problems Solved vs Non-AWS:**
- **Environment Consistency**: Docker ensures identical environments across dev/staging/prod
- **Dependency Management**: All dependencies packaged in containers
- **Local Testing**: Can test full stack locally before deployment

---

## Source Control Integration

### Step 2: GitHub Repository Management

**AWS Approach:**
```bash
# Developer workflow
git add .
git commit -m "feat: add new contact endpoint"
git push origin main  # Triggers automated CI/CD
```

**Intention:**
- Single source of truth for code
- Automated triggering of deployment pipeline
- Version control with rollback capabilities

**Non-AWS Alternative:**
```bash
# Manual deployment after push
git push origin main
ssh user@server "cd /app && git pull && ./deploy.sh"
```

**Problems Solved vs Non-AWS:**
- **Automation**: No manual intervention required
- **Consistency**: Same deployment process every time
- **Auditability**: Complete deployment history and rollback capability

---

## Continuous Integration (CI)

### Step 3: GitHub Actions CI Pipeline

**AWS Approach:**
```yaml
# .github/workflows/ci.yml
name: CI - Build, Test, Push
on:
  push:
    branches: [ main ]

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup .NET
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: '8.0.x'

      - name: Restore & Test
        run: |
          dotnet restore ./src/API/ContactService.API.csproj
          dotnet test ./tests --no-build --verbosity normal

      - name: Build & Publish
        run: |
          dotnet publish ./src/API/ContactService.API.csproj -c Release -o ./publish

      - name: Build Docker Image
        run: |
          docker build -t contact-service ./backend/services/contact-service

      - name: Container Health Test
        run: |
          docker run -d --name contact-test -p 5002:5002 contact-service
          sleep 10
          curl --fail http://localhost:5002/health || (docker logs contact-test && exit 1)

      - name: Upload DB Scripts to S3
        run: |
          aws s3 cp ./db s3://$S3_BUCKET/db/${{ github.sha }}/ --recursive

      - name: Push to ECR
        run: |
          aws ecr get-login-password | docker login --username AWS --password-stdin $ECR_REGISTRY
          docker tag contact-service:latest $ECR_REGISTRY/contact-service:${{ github.sha }}
          docker push $ECR_REGISTRY/contact-service:${{ github.sha }}
```

**Intention:**
- **Quality Gates**: Automated testing prevents bad code from deploying
- **Container Validation**: Health checks ensure deployable containers
- **Artifact Management**: Versioned storage of both code artifacts and database scripts
- **Immutable Deployments**: Each commit gets unique, traceable artifacts

**Non-AWS Alternative:**
```bash
# Jenkins or manual CI
./scripts/test.sh
./scripts/build.sh
scp ./dist/* user@server:/app/
ssh user@server "systemctl restart myapp"
```

**Problems Solved vs Non-AWS:**
- **Reliability**: Built-in retry mechanisms and error handling
- **Scalability**: Parallel job execution, unlimited concurrent builds
- **Integration**: Native AWS service integration
- **Cost**: Pay-per-use vs maintaining CI servers
- **Maintenance**: No server management, automatic updates

---

## Database Management

### Step 4: Database Initialization & Migration

**AWS Approach - Option A (CodeBuild):**
```bash
# buildspec.yml for CodeBuild
version: 0.2
phases:
  install:
    runtime-versions:
      dotnet: 6.0
  pre_build:
    commands:
      - aws s3 cp s3://$S3_BUCKET/db/$COMMIT_SHA/init.sql ./init.sql
      - DB_SECRET=$(aws secretsmanager get-secret-value --secret-id $DB_SECRET_ARN --query SecretString --output text)
      - CONNECTION_STRING=$(echo $DB_SECRET | jq -r '.ConnectionString')
  build:
    commands:
      - sqlcmd -S $DB_HOST -d $DB_NAME -Q "$(cat init.sql)"
```

**AWS Approach - Option B (Lambda):**
```csharp
// Lambda function for DB initialization
public async Task<APIGatewayProxyResponse> FunctionHandler(APIGatewayProxyRequest request)
{
    var s3Client = new AmazonS3Client();
    var secretsClient = new AmazonSecretsManagerClient();
    
    // Download SQL script
    var sqlScript = await s3Client.GetObjectAsync(bucketName, $"db/{commitSha}/init.sql");
    
    // Get DB credentials
    var secret = await secretsClient.GetSecretValueAsync(new GetSecretValueRequest 
    { 
        SecretId = dbSecretArn 
    });
    
    // Execute SQL
    using var connection = new SqlConnection(connectionString);
    await connection.ExecuteAsync(sqlScript);
    
    return new APIGatewayProxyResponse { StatusCode = 200 };
}
```

**Intention:**
- **Version Control for DB**: Database changes tracked alongside code
- **Automated Execution**: No manual SQL execution required
- **Environment Isolation**: Separate database initialization per environment
- **Rollback Capability**: Can revert to previous database state

**Non-AWS Alternative:**
```bash
# Manual database deployment
scp ./db/migrations/*.sql user@dbserver:/tmp/
ssh user@dbserver "for f in /tmp/*.sql; do mysql mydb < $f; done"
```

**Problems Solved vs Non-AWS:**
- **Security**: Credentials stored in Secrets Manager, not in scripts
- **Automation**: No manual intervention or human error
- **Auditability**: Complete log of database changes
- **Consistency**: Same process across all environments
- **VPC Security**: Database access from secure, managed environment

---

## Container Registry & Orchestration

### Step 5: ECR and ECS Deployment

**AWS ECR (Elastic Container Registry):**
```bash
# Push to ECR (automated in CI)
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789.dkr.ecr.us-east-1.amazonaws.com
docker tag contact-service:latest 123456789.dkr.ecr.us-east-1.amazonaws.com/contact-service:v1.2.3
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/contact-service:v1.2.3
```

**AWS ECS Task Definition:**
```json
{
  "family": "contact-service-task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::123456789:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::123456789:role/ecsTaskRole",
  "containerDefinitions": [
    {
      "name": "contact-service",
      "image": "123456789.dkr.ecr.us-east-1.amazonaws.com/contact-service:v1.2.3",
      "portMappings": [
        {
          "containerPort": 5002,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "ASPNETCORE_ENVIRONMENT",
          "value": "Production"
        }
      ],
      "secrets": [
        {
          "name": "ConnectionStrings__DefaultConnection",
          "valueFrom": "arn:aws:secretsmanager:us-east-1:123456789:secret:contact-db-secret"
        }
      ],
      "healthCheck": {
        "command": ["CMD-SHELL", "curl -f http://localhost:5002/health || exit 1"],
        "interval": 30,
        "timeout": 5,
        "retries": 3
      }
    }
  ]
}
```

**ECS Service Deployment:**
```bash
# Update ECS service with new task definition
aws ecs update-service \
  --cluster contact-service-cluster \
  --service contact-service \
  --task-definition contact-service-task:25
```

**Intention:**
- **High Availability**: Automatic container restart and replacement
- **Scalability**: Auto-scaling based on metrics
- **Zero Downtime**: Rolling deployments with health checks
- **Resource Optimization**: Right-sized containers with efficient resource usage

**Non-AWS Alternative:**
```bash
# Docker on VM
docker pull myregistry/contact-service:latest
docker stop contact-service || true
docker rm contact-service || true
docker run -d --name contact-service -p 80:5002 myregistry/contact-service:latest
```

**Problems Solved vs Non-AWS:**
- **High Availability**: ECS automatically replaces failed containers
- **Load Balancing**: Integrated ALB with health checks
- **Scaling**: Automatic scaling based on CPU/memory/custom metrics
- **Security**: VPC networking, IAM roles, secrets management
- **Monitoring**: Built-in CloudWatch integration
- **Management**: No server maintenance, automatic OS patching

---

## Secrets Management

### Step 6: AWS Secrets Manager Integration

**Storing Secrets:**
```bash
# Create database secret
aws secretsmanager create-secret \
  --name contact-db-credentials \
  --secret-string '{
    "username": "dbuser",
    "password": "securepassword123",
    "host": "contact-db.cluster-xyz.us-east-1.rds.amazonaws.com",
    "port": "1433",
    "database": "ContactDB"
  }'

# Create JWT secret
aws secretsmanager create-secret \
  --name contact-jwt-config \
  --secret-string '{
    "secret": "your-256-bit-secret-key",
    "issuer": "contact-service",
    "audience": "contact-api-users"
  }'
```

**Application Integration:**
```csharp
// Program.cs - Secrets Manager integration
public class Program
{
    public static async Task Main(string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);
        
        // Add Secrets Manager configuration
        if (!builder.Environment.IsDevelopment())
        {
            builder.Configuration.AddSecretsManager(region: RegionEndpoint.USEast1);
        }
        
        var app = builder.Build();
        
        // Database connection from secrets
        var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
        builder.Services.AddDbContext<ContactDbContext>(options =>
            options.UseSqlServer(connectionString));
            
        // JWT configuration from secrets
        var jwtConfig = builder.Configuration.GetSection("JwtConfig");
        builder.Services.Configure<JwtConfig>(jwtConfig);
    }
}
```

**ECS Task Role Policy:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue"
      ],
      "Resource": [
        "arn:aws:secretsmanager:us-east-1:123456789:secret:contact-db-credentials*",
        "arn:aws:secretsmanager:us-east-1:123456789:secret:contact-jwt-config*"
      ]
    }
  ]
}
```

**Intention:**
- **Security**: Centralized, encrypted secret storage
- **Rotation**: Automatic credential rotation
- **Access Control**: Fine-grained IAM permissions
- **Audit**: Complete access logging

**Non-AWS Alternative:**
```bash
# Environment variables or config files
export DB_PASSWORD="securepassword123"
echo "ConnectionString=Server=localhost;..." > appsettings.json
```

**Problems Solved vs Non-AWS:**
- **Security**: Secrets never stored in plain text or environment variables
- **Rotation**: Automatic password rotation with zero downtime
- **Access Control**: IAM-based access with detailed logging
- **Encryption**: Automatic encryption at rest and in transit
- **Compliance**: Meets enterprise security requirements

---

## Continuous Deployment (CD)

### Step 7: CodePipeline and CodeBuild CD

**CodePipeline Configuration:**
```json
{
  "name": "contact-service-pipeline",
  "stages": [
    {
      "name": "Source",
      "actions": [
        {
          "name": "SourceAction",
          "actionTypeId": {
            "category": "Source",
            "owner": "AWS",
            "provider": "ECR",
            "version": "1"
          },
          "configuration": {
            "RepositoryName": "contact-service",
            "ImageTag": "latest"
          }
        }
      ]
    },
    {
      "name": "Deploy-Staging",
      "actions": [
        {
          "name": "DeployToStaging",
          "actionTypeId": {
            "category": "Build",
            "owner": "AWS",
            "provider": "CodeBuild",
            "version": "1"
          },
          "configuration": {
            "ProjectName": "contact-service-deploy-staging"
          }
        }
      ]
    },
    {
      "name": "Manual-Approval",
      "actions": [
        {
          "name": "ManualApproval",
          "actionTypeId": {
            "category": "Approval",
            "owner": "AWS",
            "provider": "Manual",
            "version": "1"
          }
        }
      ]
    },
    {
      "name": "Deploy-Production",
      "actions": [
        {
          "name": "DeployToProduction",
          "actionTypeId": {
            "category": "Build",
            "owner": "AWS",
            "provider": "CodeBuild",
            "version": "1"
          },
          "configuration": {
            "ProjectName": "contact-service-deploy-production"
          }
        }
      ]
    }
  ]
}
```

**CodeBuild Deploy Script:**
```bash
#!/bin/bash
# deploy.sh - CodeBuild deployment script

set -e

# Get current task definition
TASK_DEFINITION=$(aws ecs describe-task-definition --task-definition contact-service-task --query 'taskDefinition')

# Update image URI in task definition
NEW_TASK_DEF=$(echo $TASK_DEFINITION | jq --arg IMAGE "$ECR_REGISTRY/contact-service:$IMAGE_TAG" '.containerDefinitions[0].image = $IMAGE')

# Register new task definition
NEW_TASK_DEF_ARN=$(aws ecs register-task-definition --cli-input-json "$NEW_TASK_DEF" --query 'taskDefinition.taskDefinitionArn' --output text)

# Update ECS service
aws ecs update-service \
  --cluster contact-service-cluster \
  --service contact-service \
  --task-definition $NEW_TASK_DEF_ARN

# Wait for deployment to complete
aws ecs wait services-stable \
  --cluster contact-service-cluster \
  --services contact-service

echo "Deployment completed successfully"
```

**Intention:**
- **Progressive Deployment**: Staging → Manual Approval → Production
- **Rollback Capability**: Easy revert to previous versions
- **Approval Gates**: Human oversight for production deployments
- **Monitoring Integration**: Deployment status tracking

**Non-AWS Alternative:**
```bash
# Manual deployment script
#!/bin/bash
ssh staging-server "docker pull myapp:latest && docker-compose up -d"
# Manual testing and approval
ssh prod-server "docker pull myapp:latest && docker-compose up -d"
```

**Problems Solved vs Non-AWS:**
- **Orchestration**: Automated multi-stage deployment pipeline
- **Approval Workflow**: Built-in manual approval steps
- **Rollback**: One-click rollback to previous versions
- **Visibility**: Complete deployment status and history
- **Integration**: Native integration with all AWS services

---

## Monitoring & Health Checks

### Step 8: CloudWatch and ALB Health Checks

**Application Health Check:**
```csharp
// HealthCheck implementation
public void ConfigureServices(IServiceCollection services)
{
    services.AddHealthChecks()
        .AddDbContextCheck<ContactDbContext>()
        .AddCheck<ExternalServiceHealthCheck>("external-service")
        .AddCheck("self", () => HealthCheckResult.Healthy("API is running"));
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.UseHealthChecks("/health", new HealthCheckOptions
    {
        ResponseWriter = UIResponseWriter.WriteHealthCheckUIResponse
    });
}
```

**Dockerfile Health Check:**
```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY . .
EXPOSE 5002

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:5002/health || exit 1

ENTRYPOINT ["dotnet", "ContactService.API.dll"]
```

**ALB Health Check Configuration:**
```bash
# ALB Target Group health check
aws elbv2 modify-target-group \
  --target-group-arn arn:aws:elasticloadbalancing:us-east-1:123456789:targetgroup/contact-service/abc123 \
  --health-check-path "/health" \
  --health-check-interval-seconds 30 \
  --health-check-timeout-seconds 5 \
  --healthy-threshold-count 2 \
  --unhealthy-threshold-count 3
```

**CloudWatch Alarms:**
```bash
# High error rate alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "ContactService-HighErrorRate" \
  --alarm-description "Alert when error rate exceeds 5%" \
  --metric-name "4XXError" \
  --namespace "AWS/ApplicationELB" \
  --statistic "Sum" \
  --period 300 \
  --threshold 5 \
  --comparison-operator "GreaterThanThreshold" \
  --alarm-actions "arn:aws:sns:us-east-1:123456789:contact-service-alerts"

# High CPU utilization alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "ContactService-HighCPU" \
  --alarm-description "Alert when CPU exceeds 80%" \
  --metric-name "CPUUtilization" \
  --namespace "AWS/ECS" \
  --statistic "Average" \
  --period 300 \
  --threshold 80 \
  --comparison-operator "GreaterThanThreshold"
```

**Intention:**
- **Proactive Monitoring**: Detect issues before they impact users
- **Automated Recovery**: Unhealthy containers automatically replaced
- **Comprehensive Metrics**: Application, infrastructure, and business metrics
- **Alerting**: Immediate notification of critical issues

**Non-AWS Alternative:**
```bash
# Basic monitoring with cron
*/5 * * * * curl -f http://localhost:5000/health || systemctl restart myapp
# Manual log checking
tail -f /var/log/myapp.log | grep ERROR
```

**Problems Solved vs Non-AWS:**
- **Comprehensive Monitoring**: Application + Infrastructure metrics
- **Automated Actions**: Auto-scaling, restart, notifications
- **Centralized Logging**: Structured logs with search capability
- **Dashboards**: Visual monitoring and alerting
- **Retention**: Long-term metric and log storage

---

## Cost & Complexity Comparison

### AWS-Based Deployment

**Monthly Cost Estimate (Small Application):**
- **ECS Fargate**: 2 tasks × 0.25 vCPU × 0.5GB RAM = ~$15/month
- **RDS SQL Server**: db.t3.micro = ~$25/month
- **ALB**: ~$20/month
- **ECR**: <1GB storage = ~$0.10/month
- **S3**: <1GB = ~$0.25/month
- **Secrets Manager**: 2 secrets = ~$0.80/month
- **CloudWatch**: Basic logging = ~$5/month
- **Total**: ~$66/month

**Complexity Level**: Medium-High
- **Initial Setup**: Complex (multiple services configuration)
- **Ongoing Maintenance**: Low (managed services)
- **Scaling**: Automatic
- **Security**: Built-in best practices

### Traditional VPS Deployment

**Monthly Cost Estimate:**
- **VPS**: 2GB RAM, 1 vCPU = ~$20/month
- **Database**: Same VPS or separate ($10/month)
- **Load Balancer**: nginx (included)
- **Monitoring**: Self-hosted or basic service = ~$5/month
- **Total**: ~$35/month

**Complexity Level**: Low-Medium
- **Initial Setup**: Simple (single server)
- **Ongoing Maintenance**: High (manual updates, backups, monitoring)
- **Scaling**: Manual (vertical/horizontal)
- **Security**: Manual configuration required

### Key Trade-offs

| Aspect | AWS Approach | Traditional VPS |
|--------|--------------|----------------|
| **Initial Cost** | Higher (~$66/month) | Lower (~$35/month) |
| **Scalability** | Automatic, instant | Manual, time-consuming |
| **Maintenance** | Minimal (managed) | High (manual) |
| **Security** | Enterprise-grade | Depends on configuration |
| **Reliability** | 99.9%+ SLA | Depends on provider |
| **Complexity** | High initial, low ongoing | Low initial, high ongoing |
| **Vendor Lock-in** | High | Low |
| **Learning Curve** | Steep | Moderate |

### When to Choose AWS Approach

**Choose AWS when:**
- Application requires high availability (99.9%+)
- Team size is small (1-5 developers)
- Rapid scaling requirements
- Compliance requirements (SOC2, HIPAA)
- Budget allows for operational efficiency over raw costs
- Long-term maintenance costs are a concern

**Choose Traditional when:**
- Budget is very constrained
- Simple application with predictable traffic
- Team has strong DevOps expertise
- Vendor lock-in is a major concern
- Full control over infrastructure is required

---

## Conclusion

The AWS-based approach provides a production-ready, enterprise-grade deployment pipeline with minimal ongoing maintenance. While the initial complexity and cost are higher, the long-term benefits include:

1. **Automated Operations**: Reduces human error and operational overhead
2. **Scalability**: Handles traffic spikes without manual intervention  
3. **Security**: Enterprise-grade security with minimal configuration
4. **Reliability**: Built-in redundancy and automatic failover
5. **Observability**: Comprehensive monitoring and logging out-of-the-box

The traditional approach offers lower costs and more control but requires significant operational expertise and ongoing maintenance. The choice depends on your team's skills, budget constraints, and application requirements.
