# TechnoDev — DevOps CI/CD with GitHub Self-Hosted Runners + WebSocket API

**LKS Nasional 2026 · Cloud Computing · Modul 3**

---

## Overview

TechnoDev e-commerce platform with automated CI/CD pipeline using GitHub Actions on self-hosted EC2 runners, deployed on AWS Academy Learner Lab infrastructure.

## Architecture

| Component | Details |
|-----------|---------|
| **VPC** | `devops-vpc` (210.0.0.0/16) — 2 public + 2 private subnets |
| **Compute** | 2× EC2 t3.medium (GitHub Actions runners), 4× Lambda functions |
| **Database** | RDS PostgreSQL Multi-AZ (db.t3.small, 20GB) — 5 tables |
| **Messaging** | 3× SQS queues, 2× SNS topics |
| **API** | REST API (`/users`, `/orders`, `/products`) + WebSocket API (real-time) |
| **Auth** | Cognito User Pool (`devops-user-pool`) + SPA Client + Domain + Authorizer |
| **Container Registry** | 4× ECR repositories |
| **Frontend** | AWS Amplify SPA (Tailwind CSS + DaisyUI) |
| **CI/CD** | GitHub Actions on self-hosted runners |
| **Monitoring** | CloudWatch dashboard + alarms, VPC Flow Logs, Secrets Manager |

## Project Structure

```
.
├── .github/workflows/
│   ├── test-runner.yaml          # Verify runners (workflow_dispatch)
│   ├── ci.yaml                   # CI: lint + build + test on PR
│   └── deploy.yaml               # CD: build + push ECR + deploy Lambda + Amplify
├── lambda/
│   ├── user-api/                 # HTTP: CRUD users
│   │   ├── app.py
│   │   └── Dockerfile
│   ├── order-api/                # HTTP: CRUD orders + SQS publish
│   │   ├── app.py
│   │   └── Dockerfile
│   ├── notification-worker/      # SQS trigger → SNS publish
│   │   ├── app.py
│   │   └── Dockerfile
│   └── websocket-api/            # WebSocket: real-time events
│       ├── app.py
│       └── Dockerfile
├── frontend/
│   └── index.html                # SPA dashboard (4 tabs)
└── sql/
    └── schema.sql                # DDL + seed data
```

## Lambda Functions

| Function | Image | Memory | Timeout | Trigger |
|----------|-------|--------|---------|---------|
| `devops-user-api` | `devops/user-api` | 512 MB | 30s | REST API `/users` |
| `devops-order-api` | `devops/order-api` | 512 MB | 30s | REST API `/orders`, `/products` |
| `devops-notif-worker` | `devops/notif-worker` | 256 MB | 60s | SQS `devops-notifications-queue` |
| `devops-websocket-api` | `devops/websocket-api` | 512 MB | 30s | WebSocket API `$connect/$disconnect/$default/sendMessage` |

## Database Tables

| Table | Purpose |
|-------|---------|
| `users` | User accounts (3 sample) |
| `products` | Product catalog (5 sample) |
| `orders` | Order records with status tracking |
| `notifications` | Notification log (email/sms/push) |
| `ws_connections` | WebSocket connection tracking |

## WebSocket API Actions

| Action | Description |
|--------|-------------|
| `ping` | Health check → returns `pong` |
| `get_connections` | List all active WebSocket connections |
| `get_orders` | Real-time order list from database |
| `order_status_update` | Update order status + broadcast to all clients |
| `send_notification` | Send notification to specific user or broadcast |

## Frontend (Tailwind CSS + DaisyUI)

| Tab | Features |
|-----|----------|
| **Products** | Read-only product table with stock badges |
| **Users** | CRUD users with DaisyUI modal dialogs |
| **Orders** | CRUD orders with status badges + update modal |
| **Real-Time** | WebSocket event feed (dark terminal), auto-reconnect, Ping/Connections/Clear buttons, badge counter |

## Quick Start

1. Provision infrastructure (VPC, subnets, IGW, NAT GW, security groups)
2. Launch EC2 instances, install Docker + AWS CLI + GitHub Actions runner
3. Register runners to GitHub repo
4. Create RDS, SQS, SNS, ECR resources
5. Push code to `main` → triggers `deploy.yaml` → builds + deploys everything
6. Access frontend at Amplify URL, configure REST API and WebSocket URLs
7. Cognito authentication auto-configured (User Pool, Client, Domain, Authorizer, Test User)

## Cognito Configuration (Task 25)

| Resource | Name | Details |
|----------|------|---------|
| User Pool | `devops-user-pool` | Email sign-in, password policy 8+ chars |
| Client | `devops-spa-client` | SPA (no secret), implicit OAuth flow |
| Domain | `devops-auth` | `{prefix}.auth.{region}.amazoncognito.com` |
| Authorizer | `devops-cognito-authorizer` | Cognito User Pools authorizer on REST API |
| Test User | `admin@technodev.local` | Password: `TechnoDev2026!` |
| Secret | `devops/cognito-config` | Stores pool_id, client_id, domain |

## References

- [GitHub Actions Self-Hosted Runners](https://docs.github.com/en/actions/hosting-your-own-runners)
- [API Gateway WebSocket API](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-websocket-api.html)
- [AWS Lambda Container Images](https://docs.aws.amazon.com/lambda/latest/dg/images-create.html)
- [Tailwind CSS](https://tailwindcss.com/)
- [DaisyUI](https://daisyui.com/)

---

*LKS Nasional 2026 · Cloud Computing*