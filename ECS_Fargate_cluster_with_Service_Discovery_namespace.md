# ECS Fargate Cluster with Cloud Map Service Discovery

This CloudFormation template creates:

* ECS Cluster
* FARGATE Capacity Provider
* FARGATE_SPOT Capacity Provider
* Cloud Map Private DNS Namespace
* ECS Service Connect integration
* ECS Exec configuration
* Container Insights enabled

This stack acts as the foundational infrastructure stack for ECS Fargate workloads.

---

# Architecture

```text
VPC
 └── Cloud Map Namespace
       └── demo.internal

ECS Cluster
 ├── FARGATE
 └── FARGATE_SPOT
```

Application services deployed later can automatically use:

```text
payment.demo.internal
orders.demo.internal
user.demo.internal
```

for internal service-to-service communication.

---

# Features

* ECS Fargate only
* No EC2
* No Auto Scaling Groups
* No ECS EC2 Capacity Providers
* Cloud Map Namespace
* Service Discovery ready
* Service Connect ready
* Container Insights enabled
* ECS Exec enabled
* FARGATE_SPOT ready

---

# Resources Created

| Resource              | Type                                       |
| --------------------- | ------------------------------------------ |
| ECS Cluster           | AWS::ECS::Cluster                          |
| Private DNS Namespace | AWS::ServiceDiscovery::PrivateDnsNamespace |

---

# Prerequisites

Before deploying this stack, ensure:

* VPC already exists
* Private/Public subnets already exist
* Security groups already exist
* IAM permissions for ECS and Cloud Map are available

---

# Parameters

| Parameter     | Description                                 | Example           |
| ------------- | ------------------------------------------- | ----------------- |
| NamespaceName | Private DNS namespace for service discovery | demo.internal     |
| VpcId         | VPC ID for namespace association            | vpc-0abc123456789 |
| ClusterName   | ECS Cluster Name                            | prod-cluster      |

---

# Example Values

| Parameter     | Example                |
| ------------- | ---------------------- |
| NamespaceName | demo.internal          |
| VpcId         | vpc-0ab123cd456ef789   |
| ClusterName   | production-ecs-cluster |

---

# Cloud Map Service Discovery

This stack creates a private DNS namespace inside the VPC.

Example:

```text
demo.internal
```

Applications deployed later can register services such as:

```text
payment.demo.internal
orders.demo.internal
inventory.demo.internal
```

AWS automatically manages:

* Route53 private hosted zone
* DNS records
* Task IP registration
* Health-based deregistration

---

# Capacity Providers

The cluster supports:

| Capacity Provider | Purpose                       |
| ----------------- | ----------------------------- |
| FARGATE           | Standard production workloads |
| FARGATE_SPOT      | Cost optimized workloads      |

Default strategy:

```yaml
FARGATE
```

Applications can later use:

```yaml
CapacityProviderStrategy:
  - CapacityProvider: FARGATE
    Weight: 1

  - CapacityProvider: FARGATE_SPOT
    Weight: 2
```

---

# Service Connect

This stack enables ECS Service Connect using:

```yaml
ServiceConnectDefaults:
  Namespace: demo.internal
```

Applications can communicate internally using:

```text
http://payment
http://orders
```

without needing full DNS names.

---

# ECS Exec

ECS Exec is enabled through:

```yaml
ExecuteCommandConfiguration
```

This allows secure shell access into running containers.

Example:

```bash
aws ecs execute-command \
  --cluster prod-cluster \
  --task TASK_ID \
  --container payment \
  --interactive \
  --command "/bin/sh"
```

---

# Monitoring

Container Insights is enabled:

```yaml
containerInsights: enabled
```

Metrics available:

* CPU utilization
* Memory utilization
* Network traffic
* Task metrics
* Service metrics

---

# Deployment Command

```bash
aws cloudformation create-stack \
  --stack-name ecs-cluster \
  --template-body file://ecs-cluster.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameters \
ParameterKey=NamespaceName,ParameterValue=demo.internal \
ParameterKey=VpcId,ParameterValue=vpc-0ab123cd456ef789 \
ParameterKey=ClusterName,ParameterValue=production-ecs-cluster
```

---

# Outputs

| Output      | Description            |
| ----------- | ---------------------- |
| NamespaceId | Cloud Map Namespace ID |
| ClusterArn  | ECS Cluster ARN        |

---

# Example Deployment Flow

## Step 1

Deploy foundational stack:

```text
ecs-cluster-stack
```

Creates:

* ECS Cluster
* Cloud Map Namespace

---

## Step 2

Deploy application stacks:

```text
payment-service-stack
orders-service-stack
inventory-service-stack
```

Each stack creates:

* ECS Service
* Task Definition
* Cloud Map Service

---

# Internal DNS Examples

| Service   | Internal URL            |
| --------- | ----------------------- |
| payment   | payment.demo.internal   |
| orders    | orders.demo.internal    |
| inventory | inventory.demo.internal |

---

# Best Practices

* Keep namespace in foundational stack
* Use separate stack per application
* Use FARGATE_SPOT for non-critical workloads
* Use private subnets for production
* Enable ECS Exec only when required
* Use ALB health checks
* Use CloudWatch log retention policies

---

# Recommended Production Architecture

```text
Internet
   ↓
Application Load Balancer
   ↓
ECS Fargate Services
   ↓
Cloud Map Service Discovery
   ↓
Internal Service Communication
```
