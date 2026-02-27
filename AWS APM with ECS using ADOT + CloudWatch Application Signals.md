# 🚀 AWS APM with ECS using ADOT + CloudWatch Application Signals

---

# 📌 Overview

This guide explains how to enable **full AWS APM (Application Performance Monitoring)** for applications running on **Amazon ECS (EC2 launch type)** using:

* AWS Distro for OpenTelemetry (ADOT)
* OpenTelemetry Java Agent
* Amazon CloudWatch Application Signals
* AWS X-Ray

After completing this setup, you will get:

✅ Service map
✅ Transaction search
✅ RED metrics (Request, Error, Duration)
✅ Latency percentiles (P50/P90/P99)
✅ Dependency graph
✅ SLO capability

---

# 🏗 Architecture

```
+----------------------+
|   Spring Boot App    |
|  (OTEL Java Agent)   |
+----------+-----------+
           |
           | OTLP (4317)
           v
+----------------------+
|    ADOT Collector    |
|  (ECS EC2 - Bridge)  |
+----------+-----------+
           |
           +----> AWS X-Ray (Traces)
           |
           +----> CloudWatch EMF (Metrics)
                        |
                        v
            CloudWatch Application Signals
```

---

# 🔎 How It Works

1. OpenTelemetry Java Agent instruments your app.
2. It generates spans and metrics.
3. Application Signals mode aggregates RED metrics.
4. ADOT Collector exports:

   * Traces → AWS X-Ray
   * Metrics → CloudWatch (EMF format)
5. CloudWatch builds the APM dashboards automatically.

---

# 🧱 Prerequisites

* ECS Cluster (EC2 launch type)
* Bridge network mode
* IAM Role for ADOT
* Java 8+ application
* OpenTelemetry Java Agent

---

# 🔐 IAM Policy (ecs-adot-task-role)

Attach this policy to your ADOT task role:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "cloudwatch:PutMetricData",
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents",
        "xray:PutTraceSegments",
        "xray:PutTelemetryRecords",
        "ecs:DescribeTasks",
        "ecs:DescribeServices",
        "ecs:DescribeClusters",
        "ec2:DescribeInstances"
      ],
      "Resource": "*"
    }
  ]
}
```

---

# 🐳 ADOT Collector Task Definition (ECS EC2)

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
        { "containerPort": 4317, "hostPort": 4317 },
        { "containerPort": 4318, "hostPort": 4318 },
        { "containerPort": 4316, "hostPort": 4316 }
      ],
      "command": ["--config", "env:AOT_CONFIG_CONTENT"],
      "environment": [
        {
          "name": "AOT_CONFIG_CONTENT",
          "value": "receivers:\n  otlp:\n    protocols:\n      grpc:\n        endpoint: 0.0.0.0:4317\n      http:\n        endpoint: 0.0.0.0:4318\n\nprocessors:\n  batch:\n  resourcedetection:\n    detectors: [env, ecs, ec2]\n    timeout: 2s\n    override: false\n\nexporters:\n  awsxray:\n    region: \"ap-south-1\"\n  awsemf:\n    namespace: \"aws/application_signals/metrics\"\n    log_group_name: \"/aws/application-signals/data\"\n    log_stream_name: \"{TaskId}\"\n    region: \"ap-south-1\"\n    dimension_rollup_option: NoDimensionRollup\n    metric_declarations:\n      - dimensions:\n          - [\"Service\", \"Operation\", \"RemoteService\", \"RemoteOperation\"]\n          - [\"Service\", \"Operation\"]\n          - [\"Service\", \"RemoteService\"]\n          - [\"Service\"]\n        metric_name_selectors:\n          - \"latency\"\n          - \"error\"\n          - \"fault\"\n          - \"success_rate\"\n\nservice:\n  pipelines:\n    traces:\n      receivers: [otlp]\n      processors: [batch, resourcedetection]\n      exporters: [awsxray]\n    metrics:\n      receivers: [otlp]\n      processors: [batch, resourcedetection]\n      exporters: [awsemf]\n  telemetry:\n    logs:\n      level: \"info\""
        }
      ]
    }
  ]
}
```

---

# 🧠 Application Container Environment Variables

Add this to your Java app container:

```bash
ENV OTEL_EXPORTER_OTLP_ENDPOINT=http://172.17.0.1:4317
ENV OTEL_EXPORTER_OTLP_PROTOCOL=grpc
ENV OTEL_TRACES_EXPORTER=otlp
ENV OTEL_METRICS_EXPORTER=otlp
ENV OTEL_LOGS_EXPORTER=none
ENV OTEL_SERVICE_NAME=my-service-name
ENV OTEL_RESOURCE_ATTRIBUTES=deployment.environment=dev
ENV OTEL_AWS_APPLICATION_SIGNALS_ENABLED=true
ENV OTEL_AWS_APPLICATION_SIGNALS_EXPORTER_ENDPOINT=http://172.17.0.1:4316
```

---

# 📊 What You Should See

Go to:

CloudWatch → Application Signals → Services

You should now see:

* Service list
* Request rate
* Error rate
* Latency percentiles
* Dependency map
* Transaction search

---

# 🛠 Troubleshooting

## 1️⃣ No Service Appearing

Check CloudWatch → Metrics → Namespace:

```
aws/application_signals/metrics
```

If empty:

* Verify OTEL_AWS_APPLICATION_SIGNALS_ENABLED=true
* Verify port 4316 is open
* Check ADOT logs

---

## 2️⃣ Traces Only, No Metrics

Ensure:

```
OTEL_METRICS_EXPORTER=otlp
```

---

## 3️⃣ ECS Task Not Starting

Check memory usage on EC2:

```
docker stats
free -m
```

---

# 📚 Official AWS References

* AWS Distro for OpenTelemetry (ADOT)
  [https://aws-otel.github.io/](https://aws-otel.github.io/)

* CloudWatch Application Signals
  [https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Application-Signals.html](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Application-Signals.html)

* X-Ray Documentation
  [https://docs.aws.amazon.com/xray/](https://docs.aws.amazon.com/xray/)

---

# 🏁 Conclusion

This setup provides a fully functional AWS-native APM solution using:

* OpenTelemetry
* ADOT Collector
* AWS X-Ray
* CloudWatch Application Signals

It works for ECS EC2 environments and scales with Auto Scaling Groups.

---
