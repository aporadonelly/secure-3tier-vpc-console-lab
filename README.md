# Secure 3-Tier VPC (Console-Only) – Week 1 Lab

**Goal:** Build a production-style VPC with public **ALB** tier, private **App** tier (via **NAT**), and **isolated DB** tier — security-by-default.

## What I built (Phase 0–1)
- S3 **lab bucket** with **Versioning** + **Default encryption (SSE-S3)** + **Block Public Access = ON**
- VPC **10.0.0.0/16** across **2 AZs**
  - Public: `10.0.0.0/24 (a)`, `10.0.1.0/24 (b)` — auto-assign public IP **ON**
  - App (private): `10.0.10.0/24 (a)`, `10.0.11.0/24 (b)`
  - DB (isolated): `10.0.20.0/24 (a)`, `10.0.21.0/24 (b)`
- **IGW** attached; **NAT Gateway** in a public subnet (with Elastic IP)
- Route tables:
  - **rt-public** → `0.0.0.0/0 → IGW` (public subnets)
  - **rt-app** → `0.0.0.0/0 → NAT` (app subnets)
  - **rt-db** → **local only** (db subnets)
- VPC DNS **resolution + hostnames enabled**

## Why this design
- **Least exposure:** only ALB is internet-facing; app has egress-only via NAT; DB has no internet route.
- **HA-ready:** subnets in two AZs.
- **Security-by-default:** encryption & segmented network from day one.

## Architecture (Mermaid)
```mermaid
flowchart LR
  Internet((Internet))
  IGW[Internet Gateway]
  ALB[ALB\nPublic subnets A/B]
  NAT[NAT Gateway\nPublic A]
  APP[EC2 App instances\nPrivate subnets A/B]
  DB[(RDS / DB\nIsolated subnets A/B)]

  Internet -- 443/80 --> IGW --> ALB
  ALB -- 8080 --> APP
  APP -- 5432 --> DB
  APP -- 0.0.0.0/0 (egress) --> NAT --> IGW

  %% Security-group intent (labels only)
  classDef sg fill:#eef,stroke:#99f,stroke-width:1px;
  class ALB,APP,DB sg;
