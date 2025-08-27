# Secure 3-Tier VPC (Console-Only) – Week 1 Lab

**Goal:** Build a production-style VPC with public **ALB** tier, private **App** tier (via **NAT**), and **isolated DB** tier — security-by-default.

## What I built (Phase 0–1)
- S3 **lab bucket** with **Versioning** + **Default encryption** (SSE-S3)
- VPC **10.0.0.0/16** across **2 AZs**
  - Public: 10.0.0.0/24 (a), 10.0.1.0/24 (b) — auto-assign public IP = ON
  - App (private): 10.0.10.0/24 (a), 10.0.11.0/24 (b)
  - DB (isolated): 10.0.20.0/24 (a), 10.0.21.0/24 (b)
- **IGW** attached; **NAT Gateway** in public subnet
- Route tables:
  - `rt-public` → `0.0.0.0/0 → IGW` (public subnets)
  - `rt-app` → `0.0.0.0/0 → NAT` (app subnets)
  - `rt-db` → **local only** (db subnets)
- VPC DNS hostnames & resolution **enabled**

## Why this design
- **Least exposure:** only ALB is internet-facing; app has egress only; DB has no internet path.
- **HA ready:** subnets in two AZs.
- **Security-by-default:** encryption & segmented network from day one.

## Architecture (Mermaid)
```mermaid
flowchart LR
  Internet((Internet)) --> ALB[ALB - Public Subnets]
  subgraph VPC
    subgraph Public[Public Subnets]
      ALB
    end
    subgraph Private[Private App Subnets]
      EC2[(App Instances)]
    end
    subgraph Isolated[Isolated DB Subnets]
      RDS[(Database)]
    end
  end
  ALB ---|HTTP/HTTPS| EC2
  EC2 ---|DB Port| RDS
