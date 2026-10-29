# Azure NSG Implementation for Multi-Tier Application Security

## Overview

This project demonstrates the implementation of Azure Network Security Groups (NSGs) and Application Security Groups (ASGs) to secure a multi-tier application architecture. The solution applies the principle of least privilege by allowing only required traffic flows while restricting unauthorized access to application and database resources.

The lab showcases how NSGs and ASGs work together to provide scalable, application-aware security policies that simplify network management and improve security posture.

---
## Architecture Diagram

```text
                                Internet Users
                                      │
                                      │ HTTP (80)
                                      ▼
                          +----------------------+
                          |     Public IP        |
                          |      pip-web         |
                          +----------------------+
                                      │
                                      ▼
 ┌─────────────────────────────────────────────────────────────┐
 │                    VNET-LAB (10.0.0.0/16)                  │
 │                                                             │
 │   ┌─────────────────┐          ┌─────────────────┐          │
 │   │ SNET-WEB        │          │ SNET-DB         │          │
 │   │ 10.0.1.0/24     │          │ 10.0.2.0/24     │          │
 │   │                 │          │                 │          │
 │   │  ASG-WEB        │          │  ASG-DB         │          │
 │   │      │          │          │      │          │          │
 │   │   vml-web       │─────────▶│    vml-db       │          │
 │   │                 │ TCP 1433 │                 │          │
 │   └─────────────────┘ Allowed  └─────────────────┘          │
 │                                                             │
 └─────────────────────────────────────────────────────────────┘

Security Rules
--------------
✅ allow-http-from-internet (Priority 100)
   Internet → ASG-WEB : TCP 80

✅ allow-database-web (Priority 110)
   ASG-WEB → ASG-DB : TCP 1433

❌ deny-database-all (Priority 120)
   Any → ASG-DB : TCP 1433

Effective Access
----------------
✅ Internet → Web VM        (Allowed)
✅ Web VM → Database VM     (Allowed)
❌ Internet → Database VM   (Denied)
❌ Other VNet → Database VM (Denied)
```

### Traffic Flow

```text
Internet
   │
   ├── HTTP (80) Allowed
   ▼
Web VM (ASG-WEB)
   │
   ├── SQL (1433) Allowed
   ▼
Database VM (ASG-DB)

Internet
   │
   └── SQL (1433) Denied
       ▼
Database VM (ASG-DB)
```
---

## Project Objectives

- Create and configure Application Security Groups (ASGs)
- Create and associate Network Security Groups (NSGs)
- Implement custom inbound security rules
- Control Internet access to the web tier
- Protect the database tier from external access
- Allow only authorized Web-to-Database communication
- Validate traffic flow using connectivity testing

---

## Architecture

### Components

| Resource | Purpose |
|-----------|---------|
| VNET-LAB (10.0.0.0/16) | Virtual Network |
| SNET-WEB (10.0.1.0/24) | Web application subnet |
| SNET-DB (10.0.2.0/24) | Database subnet |
| vml-web | Web server VM |
| vml-db | Database server VM |
| ASG-WEB | Application Security Group for web tier |
| ASG-DB | Application Security Group for database tier |
| NSG-LAB | Network Security Group protecting application resources |
| Public IP | External access point for web application |

---

## Network Topology

```text
Internet
    │
    ▼
Public IP
    │
    ▼
Web VM (ASG-WEB)
    │
    │ TCP 1433
    ▼
Database VM (ASG-DB)

```

### Security Design

✅ Internet → Web VM (Allowed)

✅ Web VM → Database VM (Allowed)

❌ Internet → Database VM (Denied)

❌ Other VNet Resources → Database VM (Denied)

---

## Application Security Groups

### ASG-WEB

Contains application servers exposed to end users.

### ASG-DB

Contains backend database servers that require restricted access.

Using ASGs allows security rules to be applied dynamically without referencing individual IP addresses.

---

## NSG Security Rules

### Rule 1: Allow Internet Access to Web Tier

| Setting | Value |
|----------|---------|
| Name | allow-http-from-internet |
| Priority | 100 |
| Source | Any |
| Destination | ASG-WEB |
| Protocol | TCP |
| Port | 80 |
| Action | Allow |

**Purpose**

Allows HTTP traffic from the Internet to reach the web application.

---

### Rule 2: Allow Web-to-Database Communication

| Setting | Value |
|----------|---------|
| Name | allow-database-web |
| Priority | 110 |
| Source | ASG-WEB |
| Destination | ASG-DB |
| Protocol | TCP |
| Port | 1433 |
| Action | Allow |

**Purpose**

Allows SQL traffic from the web tier to the database tier.

---

### Rule 3: Deny All Other Database Access

| Setting | Value |
|----------|---------|
| Name | deny-database-all |
| Priority | 120 |
| Source | Any |
| Destination | ASG-DB |
| Protocol | TCP |
| Port | 1433 |
| Action | Deny |

**Purpose**

Blocks all unauthorized database access while preserving approved web-to-database communication.

---

## Rule Evaluation Order

Azure evaluates NSG rules by priority.

| Priority | Rule | Result |
|----------|------|---------|
| 100 | allow-http-from-internet | Allow Internet → Web |
| 110 | allow-database-web | Allow Web → Database |
| 120 | deny-database-all | Deny All Other Database Access |
| 65000+ | Default NSG Rules | Process Remaining Traffic |

---

## Security Validation

### Test 1 - Default NSG Rules

**Result:** ❌ Internet access to Web VM failed

Reason: Default `DenyAllInBound` rule blocked incoming traffic.

---

### Test 2 - HTTP Rule Added

**Result:** ✅ Web application accessible

Reason: Custom HTTP rule overrides default deny rule.

---

### Test 3 - Database Connectivity Before Restriction

**Result:** ✅ Database connection succeeded

Reason: Default `AllowVnetInBound` rule permitted communication within the VNet.

---

### Test 4 - Database Deny Rule Added

**Result:** ❌ Database connection failed

Reason: All SQL traffic to ASG-DB was blocked.

---

### Test 5 - Web-to-Database Allow Rule Added

**Result:** ✅ Database connection succeeded

Reason: Priority 110 allow rule was processed before priority 120 deny rule.

---

## Security Outcomes

- Enforced least-privilege network access
- Protected database resources from external exposure
- Allowed only authorized application-tier communication
- Improved scalability using ASGs
- Demonstrated layered network security architecture
- Simplified security management through logical grouping

---

## Technologies Used

- Microsoft Azure
- Azure Virtual Network (VNet)
- Azure Network Security Groups (NSGs)
- Azure Application Security Groups (ASGs)
- Azure Virtual Machines
- TCP/IP Networking
- Layered Security Controls

---

## Key Skills Demonstrated

- Network Security Administration
- Azure Networking
- Access Control Design
- Security Rule Evaluation
- Application Segmentation
- Network Traffic Filtering
- Infrastructure Security
- Cloud Security Architecture

---

## Project Outcome

Successfully implemented a secure multi-tier Azure environment where:

- Web traffic is allowed from the Internet
- Database resources remain protected from direct external access
- Only approved application-tier traffic is permitted
- Security policies scale automatically using Application Security Groups

This project demonstrates practical experience in designing and implementing Azure network security controls for enterprise-style workloads.

---
**Author:** Mubarak Mohamed  
**Role:** Azure Network Security Engineer (Self-Directed Project)
