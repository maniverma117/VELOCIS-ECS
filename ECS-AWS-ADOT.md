# AWS ADOT Integration for ECS EC2 (Spring Boot + OpenTelemetry)

This document explains how to integrate AWS Distro for OpenTelemetry (ADOT)
with ECS EC2 services using the OpenTelemetry Java Agent and exporting traces
to AWS X-Ray.

---

# Architecture Overview

- ECS Launch Type: EC2
- Network Mode: bridge
- Scheduling Strategy: DAEMON (1 collector per EC2 instance)
- OTLP Protocol: gRPC (4317)
- Export Target: AWS X-Ray
- Application Framework: Spring Boot (Java)

Each ECS EC2 instance runs one ADOT collector.
All application containers send traces to:

```

[http://localhost:4317](http://localhost:4317)

````

---

# 1️⃣ ADOT Collector Configuration

## Collector YAML

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  batch:

exporters:
  awsxray: {}

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [awsxray]
````

---

# 2️⃣ IAM Role for ADOT Task

## Trust Policy (ecs-adot-task-role)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ecs-tasks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

## Permissions Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "xray:PutTraceSegments",
        "xray:PutTelemetryRecords",
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "*"
    }
  ]
}
```

Attach this inline policy to:

```
ecs-adot-task-role
```

---

# 3️⃣ ECS Task Definition (Bridge Mode)

```json
{
  "family": "adot-collector",
  "networkMode": "bridge",
  "requiresCompatibilities": ["EC2"],
  "cpu": "256",
  "memory": "512",
  "taskRoleArn": "arn:aws:iam::<ACCOUNT_ID>:role/ecs-adot-task-role",
  "executionRoleArn": "arn:aws:iam::<ACCOUNT_ID>:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "name": "adot-collector",
      "image": "public.ecr.aws/aws-observability/aws-otel-collector:latest",
      "essential": true,
      "portMappings": [
        {
          "containerPort": 4317,
          "hostPort": 4317,
          "protocol": "tcp"
        },
        {
          "containerPort": 4318,
          "hostPort": 4318,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "AOT_CONFIG_CONTENT",
          "value": "receivers:\n  otlp:\n    protocols:\n      grpc:\n        endpoint: 0.0.0.0:4317\n      http:\n        endpoint: 0.0.0.0:4318\n\nprocessors:\n  batch:\n\nexporters:\n  awsxray: {}\n\nservice:\n  pipelines:\n    traces:\n      receivers: [otlp]\n      processors: [batch]\n      exporters: [awsxray]"
        }
      ],
      "command": [
        "--config",
        "env:AOT_CONFIG_CONTENT"
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/adot-collector",
          "awslogs-region": "ap-south-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

---

# 4️⃣ Register Task Definition (CLI)

```bash
aws ecs register-task-definition \
  --cli-input-json file://adot-task-def.json
```

---

# 5️⃣ Create ECS Service (DAEMON Mode)

Daemon mode ensures one ADOT collector per EC2 instance.

```bash
aws ecs create-service \
  --cluster <ECS_CLUSTER_NAME> \
  --service-name adot-collector-service \
  --task-definition adot-collector \
  --scheduling-strategy DAEMON
```

If service already exists:

```bash
aws ecs update-service \
  --cluster <ECS_CLUSTER_NAME> \
  --service adot-collector-service \
  --force-new-deployment
```

---

# 6️⃣ Spring Boot Application Configuration

## Dockerfile

```dockerfile
COPY opentelemetry-javaagent.jar /otel/opentelemetry-javaagent.jar

CMD ["java",
  "-javaagent:/otel/opentelemetry-javaagent.jar",
  "-jar", "app.jar"
]
```

## ECS Environment Variables

```
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
OTEL_EXPORTER_OTLP_PROTOCOL=grpc
OTEL_TRACES_EXPORTER=otlp
OTEL_SERVICE_NAME=<service-name>
OTEL_RESOURCE_ATTRIBUTES: deployment.environment=stg,service.name=<service-name>
```

---

# 7️⃣ Verification

### Check collector running

```
docker ps | grep 4317
```

Expected:

```
0.0.0.0:4317->4317/tcp
```

### Test HTTP OTLP endpoint

```
curl -X POST http://localhost:4318/v1/traces \
  -H "Content-Type: application/json" \
  -d '{}'
```

Expected:

```
{"partialSuccess":{}}
```

### Verify in AWS Console

Go to:

```
AWS Console → X-Ray → Service Map
```

You should see your ECS services.

---

# 8️⃣ Troubleshooting

| Issue                               | Cause                      | Fix                        |
| ----------------------------------- | -------------------------- | -------------------------- |
| 127.0.0.1 binding                   | Missing endpoint config    | Set 0.0.0.0                |
| YAML parsing error                  | Incorrect newline escaping | Fix env config             |
| CannotStartContainerError (/bin/sh) | Distroless image           | Do not override entrypoint |
| Metadata 500 error                  | IAM / ECS agent issue      | Fix instance role          |
| curl HTTP/0.9                       | Testing gRPC with curl     | Expected behavior          |

---

# 9️⃣ Production Notes

* Do NOT expose 4317 to public internet
* Use bridge + daemon mode for ECS EC2
* Use awsvpc + service discovery for Fargate
* Consider enabling CloudWatch Application Signals for SLO tracking

---

# 10️⃣ Result

After setup:

* Distributed tracing enabled
* Automatic instrumentation via Java agent
* Service map available in X-Ray
* Ready for SLO and latency monitoring

```
