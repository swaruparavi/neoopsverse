---
layout: post
title: "Service Discovery in Amazon ECS"
date: 2025-10-03
tags: [aws, ecs, microservices, devops]
---

## Introduction
When you run multiple microservices in **Amazon ECS (Elastic Container Service)**, those services often need to talk to each other. Instead of hardcoding IPs or manually updating configurations, **service discovery** makes it easy for containers to find and communicate with one another dynamically.

In ECS, service discovery is handled through **AWS Cloud Map**. This integrates DNS-based lookups and optional health checks so that your applications can resolve service names to healthy container endpoints automatically.

---

## How Service Discovery Works in ECS
1. **Create a namespace in AWS Cloud Map**  
   - Example: `myapp.local`
   - This namespace acts as the root domain for your ECS services.

2. **Enable service discovery when creating an ECS service**  
   - While defining your ECS service, select **Enable Service Discovery Integration**.
   - Choose the namespace you created earlier.
   - ECS automatically registers running tasks with Cloud Map.

3. **DNS-based resolution**  
   - Each service gets a DNS name like:  
     ```
     service-name.myapp.local
     ```
   - Applications in your VPC can use this DNS name to talk to the service.

4. **Automatic health management**  
   - When a task stops, ECS deregisters it from Cloud Map.
   - Only healthy tasks remain discoverable.

---

## Example Scenario
Imagine you have:
- **Frontend service** (`frontend.myapp.local`)
- **Backend service** (`backend.myapp.local`)
- **Database service** (`db.myapp.local`)

Your frontend only needs to know the DNS name of the backend service (`backend.myapp.local`). It doesn’t care which ECS task or IP is serving traffic — ECS + Cloud Map handle that automatically.

---

## Benefits
- **Dynamic resolution**: No need to manage IPs.
- **Scalability**: Works seamlessly as services scale in/out.
- **Resilience**: Dead tasks are removed from service discovery.
- **Integration**: Works with ECS, EKS, and even custom workloads in the same VPC.

---

## Quick AWS CLI Example

```bash
# Create a private DNS namespace for ECS service discovery
aws servicediscovery create-private-dns-namespace \
  --name myapp.local \
  --vpc vpc-1234567890abcdef

# When creating ECS service, link it with service discovery
aws ecs create-service \
  --cluster myCluster \
  --service-name backend \
  --task-definition backend-task:1 \
  --desired-count 2 \
  --service-registries registryArn=<CLOUD_MAP_REGISTRY_ARN>
