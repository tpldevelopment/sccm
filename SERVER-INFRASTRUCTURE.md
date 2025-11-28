# SCCM Server Infrastructure Requirements

## Overview

This document defines the server infrastructure required for a Microsoft System Center Configuration Manager (SCCM) deployment using **Windows Server 2025 Datacenter Edition**.

**Environment Size:** Small Build (< 5,000 clients)

## Server Architecture

### Small Production Environment

```
┌─────────────────────────────────────────────────────────────┐
│                    SCCM Infrastructure                       │
├─────────────────────────────────────────────────────────────┤
│  Primary Site Server (HA) + SQL Always On Availability      │
│  Distribution Points + Management Points + Software Update  │
│  Reporting Services + Application Catalog                   │
└─────────────────────────────────────────────────────────────┘
```

## Required Servers

### 1. Primary Site Server (1 Server)

**Purpose:** SCCM primary site server

**Server Specifications:**
- **OS:** Windows Server 2025 Datacenter Edition
- **Role:** SCCM Primary Site Server
- **CPU:** 8 cores minimum
- **RAM:** 32 GB minimum
- **Storage:**
  - C: 150 GB (OS and SCCM installation)
  - D: 500 GB (Content library)
  - E: 200 GB (Logs and temp files)
- **Network:** Dual 1 Gbps NICs (teamed for redundancy)

**Installed Roles:**
- SCCM Primary Site Server
- Management Point
- SMS Provider

**Server Names:**
```
SCCM-PRIMARY-01
```

---

### 2. SQL Server Cluster (2 Servers for Always On AG)

**Purpose:** Database backend for SCCM with Always On Availability Groups

**Server Specifications:**
- **OS:** Windows Server 2025 Datacenter Edition
- **SQL Version:** SQL Server 2022 Enterprise Edition
- **CPU:** 16 cores minimum
- **RAM:** 64 GB minimum
  - Reserve 4-8 GB for OS
  - Allocate remaining to SQL Server
- **Storage:**
  - C: 150 GB (OS)
  - D: 100 GB (SQL Server installation)
  - E: 500 GB (Database files - MDF)
  - F: 300 GB (Transaction logs - LDF)
  - G: 200 GB (TempDB)
  - H: 500 GB (Backups - local staging)
- **Network:** Dual 1 Gbps NICs (teamed)

**SQL Configuration:**
- Always On Availability Groups enabled
- Synchronous commit mode (2-node cluster)
- Automatic failover
- Readable secondary replica for reporting

**Server Names:**
```
SQL-SCCM-01 (Primary Replica)
SQL-SCCM-02 (Secondary Replica - Synchronous)

Availability Group Listener: SQL-SCCM-AG (Virtual Name)
```

**Required Features:**
- Windows Server Failover Clustering (WSFC)
- SQL Server Database Engine
- SQL Server Replication
- Full-Text Search
- SQL Server Agent
- File Share Witness or Cloud Witness (for quorum)

---

### 3. SQL Reporting Services Server (1 Server)

**Purpose:** SCCM reporting services point

**Server Specifications:**
- **OS:** Windows Server 2025 Datacenter Edition
- **SQL Version:** SQL Server 2022 with SSRS
- **CPU:** 4 cores
- **RAM:** 16 GB minimum
- **Storage:**
  - C: 150 GB (OS)
  - D: 100 GB (SSRS and ReportServer DB)
  - E: 200 GB (Report cache and temp)
- **Network:** Dual 1 Gbps NICs

**Server Names:**
```
SSRS-SCCM-01
```

**Installed Roles:**
- SCCM Reporting Services Point
- IIS (for Report Server)
- SQL Server Reporting Services (Native Mode)

---

### 4. Distribution Point (1 Server)

**Purpose:** Content distribution to client devices

**Server Specifications:**
- **OS:** Windows Server 2025 Datacenter Edition
- **CPU:** 4 cores minimum
- **RAM:** 16 GB minimum
- **Storage:**
  - C: 150 GB (OS)
  - D: 1 TB (Content library for small environment)
- **Network:** Dual 1 Gbps NICs

**Server Names:**
```
DP-SCCM-01
```

**Installed Roles:**
- SCCM Distribution Point
- IIS with BITS
- Remote Differential Compression

**Configuration:**
- Enable PXE responder for OS deployment
- Enable multicast (if needed)
- Configure BranchCache (for remote sites)
- Enable content validation
- Configure distribution point group membership

---

### 5. Software Update Point (1 Server)

**Purpose:** Windows Server Update Services (WSUS) integration

**Server Specifications:**
- **OS:** Windows Server 2025 Datacenter Edition
- **CPU:** 4 cores
- **RAM:** 16 GB minimum
- **Storage:**
  - C: 150 GB (OS)
  - D: 100 GB (WSUS installation)
  - E: 500 GB (WSUS content storage)
  - F: 200 GB (WSUS database - Windows Internal Database)
- **Network:** Dual 1 Gbps NICs

**Server Names:**
```
SUP-SCCM-01
```

**Installed Roles:**
- SCCM Software Update Point
- WSUS role
- IIS
- Windows Internal Database OR connect to SQL cluster

---

### 6. State Migration Point (Optional, for OS Deployment)

**Purpose:** User state storage during OS migrations

**Server Specifications:**
- **OS:** Windows Server 2025 Datacenter Edition
- **CPU:** 4 cores
- **RAM:** 8 GB minimum
- **Storage:**
  - C: 150 GB (OS)
  - D: 1 TB (User state storage for small environment)
- **Network:** Dual 1 Gbps NICs

**Server Names:**
```
SMP-SCCM-01
```

**Note:** Can be omitted for small builds if not performing OS deployments.

---

### 7. Cloud Management Gateway Connector (Optional, for Internet Clients)

**Purpose:** Bridge between on-premises SCCM and Azure CMG

**Server Specifications:**
- **OS:** Windows Server 2025 Datacenter Edition
- **CPU:** 4 cores
- **RAM:** 8 GB
- **Storage:**
  - C: 150 GB (OS)
  - D: 50 GB (Connector service)
- **Network:** Internet connectivity required

**Server Names:**
```
CMG-CONNECTOR-01
```

**Note:** Not required for small builds with on-premises clients only.

---

## Network Requirements

### Internal Network
- Dedicated SCCM VLAN (recommended)
- Layer 3 switching with jumbo frames (9000 MTU) for server-to-server
- QoS policies for SCCM traffic

### Firewall Ports

| Source | Destination | Port | Protocol | Purpose |
|--------|-------------|------|----------|---------|
| Clients | Management Point | 80, 443 | TCP | Client communication |
| Site Server | SQL Server | 1433 | TCP | Database connection |
| Site Server | Distribution Point | 445 | TCP | Content deployment |
| Clients | Distribution Point | 80, 443 | TCP | Content download |
| Distribution Point | Distribution Point | 8003 | TCP | Multicast |
| Clients | Software Update Point | 80, 8530, 8531 | TCP | Windows Updates |
| Site Server | Site Server | 10123 | TCP | Site-to-site communication |
| Management Point | SQL Server | 1433 | TCP | Database queries |

### DNS Requirements
- Forward and reverse lookup zones
- CNAME or A records for:
  - SQL AG Listener (SQL-SCCM-AG)
  - Management Points (for load balancing)
  - SCCM site server

---

## Active Directory Requirements

### Service Accounts

Create the following domain service accounts:

1. **SQL Server Service Account**
   - Account: `svc_SQL_SCCM`
   - Permissions: Local admin on SQL servers, dbcreator, sysadmin

2. **SQL Server Agent Account**
   - Account: `svc_SQLAgent_SCCM`
   - Permissions: Local admin on SQL servers

3. **SCCM Site Server Computer Account**
   - Automatically created, grant permissions to AD System Management container

4. **SCCM Network Access Account**
   - Account: `svc_SCCM_NAA`
   - Permissions: Read access to distribution points and package sources

5. **SCCM Client Push Installation Account**
   - Account: `svc_SCCM_ClientPush`
   - Permissions: Local admin on target client computers

6. **SCCM Reporting Services Point Account**
   - Account: `svc_SCCM_SSRS`
   - Permissions: Read access to SCCM database

### AD System Management Container
- Location: `CN=System Management,CN=System,DC=domain,DC=com`
- Grant full control to SCCM site server computer account
- Required for AD discovery and client deployment

---

## Storage Requirements

### Shared Storage (Optional, for Site Server HA)
- **Type:** SAN, NAS, or Storage Spaces Direct
- **Capacity:** 1 TB minimum for content library sharing
- **Protocol:** SMB 3.0 or iSCSI
- **Performance:** 10K IOPS minimum

### Backup Storage
- **Capacity:** 5 TB minimum (depends on retention policy)
- **Location:** Separate from production storage
- **Type:** Deduplicated backup target or Azure Blob Storage

---

## Server Naming Convention

```
[Function]-SCCM-[Location]-[Number]

Examples:
- SCCM-PRIMARY-01
- SQL-SCCM-01
- DP-SCCM-HQ-01
- MP-SCCM-02
- SUP-SCCM-01
```

---

## Total Server Count Summary (Small Build)

| Server Type | Quantity | Purpose |
|-------------|----------|---------|
| Primary Site Server | 1 | Single site server (includes MP) |
| SQL Server | 2 | Always On AG (2-node cluster) |
| Reporting Services | 1 | SSRS |
| Distribution Point | 1 | Content distribution |
| Software Update Point | 1 | WSUS integration |
| State Migration Point | 0-1 | Optional for OSD |
| CMG Connector | 0-1 | Optional for internet clients |
| **Total (Core)** | **5** | **Small Build - Core Servers** |
| **Total (with optional)** | **7** | **Including SMP and CMG** |

---

## Scalability Guidelines

| Environment Size | Clients | Site Servers | SQL Servers | MPs | DPs |
|------------------|---------|--------------|-------------|-----|-----|
| Small | < 5,000 | 1 | 1 | 1 | 2 |
| Medium | 5,000 - 50,000 | 2 | 3 | 2-3 | 5-10 |
| Large | 50,000 - 150,000 | 2 | 3 | 3-5 | 10-20 |
| Enterprise | > 150,000 | CAS + Multiple Primary Sites | 3 per site | 5+ per site | 20+ |

---

## Licensing Requirements

### Windows Server 2025 Datacenter Licenses
- Minimum: 5 server licenses for core environment (7 with optional servers)
- Datacenter edition allows unlimited VMs per host (if virtualized)

### SQL Server 2022 Enterprise
- Required for Always On Availability Groups
- License by core (minimum 4 cores per license)
- 2 servers × 16 cores = 32 core licenses needed

### SCCM/Configuration Manager
- Client Management License (CML) per device OR
- Microsoft Endpoint Manager (part of Microsoft 365 licensing)

---

## Virtualization Recommendations

All servers can be virtualized with proper resource allocation:

- **Hypervisor:** Hyper-V, VMware vSphere, or Azure Stack HCI
- **Host Requirements:**
  - Minimum 2 hosts for HA
  - Sufficient CPU/RAM to handle all VMs
  - Anti-affinity rules for HA pairs (keep SQL-01 and SQL-02 on different hosts)
- **Storage:** Shared storage with CSV or VSAN
- **Networking:** Dedicated virtual switches for SCCM traffic

---

## High Availability Summary

| Component | HA Method | RTO | RPO |
|-----------|-----------|-----|-----|
| Site Server | None (single server) | N/A | Backup restore |
| SQL Database | Always On AG (2-node) | < 1 min | 0 (sync commit) |
| Management Point | Single MP on site server | N/A | Backup restore |
| Distribution Point | Single DP | N/A | Content re-sync |
| Software Update Point | Single SUP | N/A | WSUS re-sync |
| Reporting Services | Single SSRS | N/A | Backup restore |

**Note:** This small build prioritizes SQL database HA. Application servers are single instances.

---

## Next Steps

1. ✅ Provision servers according to specifications
2. ✅ Configure networking and VLANs
3. ✅ Install Windows Server 2025 Datacenter on all servers
4. ✅ Join all servers to domain
5. ✅ Create service accounts in Active Directory
6. ✅ Configure Windows Failover Clustering
7. ✅ Install and configure SQL Server cluster
8. ✅ Proceed with SCCM installation (see SCCM-INSTALLATION-TODO.md)

---

**Document Version:** 1.0  
**Last Updated:** November 27, 2025  
**Maintained By:** bml104
