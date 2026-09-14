---
marp: true
theme: default
paginate: true
header: "TrueNAS Administration Training | UKM Warisan"
footer: "© UKM Warisan — TrueNAS Storage Administration"
style: |
  section {
    font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
    background-color: #f8fafc;
    color: #1e293b;
    padding: 40px 60px;
  }
  h1 {
    color: #0f4c81;
    font-weight: 700;
  }
  h2 {
    color: #1e40af;
    border-bottom: 2px solid #cbd5e1;
    padding-bottom: 6px;
  }
  h3 {
    color: #0369a1;
  }
  .highlight {
    background-color: #e0f2fe;
    border-left: 6px solid #0284c7;
    padding: 10px 16px;
    border-radius: 4px;
    font-size: 0.9em;
  }
  .warning-box {
    background-color: #fef2f2;
    border-left: 6px solid #ef4444;
    padding: 10px 16px;
    border-radius: 4px;
    font-size: 0.9em;
  }
  .tip-box {
    background-color: #f0fdf4;
    border-left: 6px solid #22c55e;
    padding: 10px 16px;
    border-radius: 4px;
    font-size: 0.9em;
  }
  code {
    background-color: #e2e8f0;
    color: #0f172a;
    padding: 2px 6px;
    border-radius: 4px;
    font-size: 0.85em;
  }
  table {
    font-size: 0.85em;
    width: 100%;
    border-collapse: collapse;
  }
  th {
    background-color: #0f4c81;
    color: white;
    padding: 8px 12px;
  }
  td {
    border: 1px solid #cbd5e1;
    padding: 6px 12px;
  }
  .grid-2 {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 24px;
  }
  .badge {
    display: inline-block;
    background-color: #0f4c81;
    color: white;
    padding: 3px 8px;
    border-radius: 12px;
    font-size: 0.75em;
    font-weight: bold;
  }
---

<!-- _class: lead -->
# TrueNAS Administration Training
**UKM Warisan Storage Infrastructure**

*Schedule: 9:00 AM – 4:00 PM (Lunch: 1:00 PM – 2:00 PM)*

---

## 🎯 Course Objectives


1. **Understand Storage Architecture**: Master ZFS hierarchy (Disks $\to$ VDEVs $\to$ Pools $\to$ Datasets).
2. **Deploy & Configure TrueNAS**: Navigate the WebUI, configure network interfaces, and provision resilient storage pools.
3. **Manage Access & File Shares**: Configure local users/groups, enforce POSIX & NFSv4 ACL permissions, and publish secure NFS exports.
4. **Implement Data Protection**: Schedule automated snapshots, execute instant file rollbacks, and set up replication tasks.
5. **Monitor & Maintain Storage**: Interpret alerts, automate SMART & Scrub schedules, and perform a live failed-disk replacement.

---

## ⏱️ Training Schedule

| Time | Session / Topic | 
| :--- | :--- |
| **09:00 - 09:30** | Course Welcome, Objectives & Lab Verification |
| **09:30 - 10:15** | **Module 1: TrueNAS Architecture & ZFS Fundamentals** |
| **10:15 - 10:30** | ☕ *Morning Refreshment Break* |
| **10:30 - 11:30** | **Module 2: Setup, Networking & Storage Pool Configuration (Lab 1)** |

---

## ⏱️ Training Schedule

| Time | Session / Topic | 
| :--- | :--- |
| **11:30 - 13:00** | **Module 3: Access Control & Network Sharing: NFS (Lab 2)** |
| **01:00 - 02:00** | 🍱 *Lunch Break & Solat* | 
| **02:00 - 03:00** | **Module 4: Data Protection, Snapshots & Disaster Recovery (Lab 3)** | 
| **03:00 - 03:45** | **Module 5: System Health, Monitoring & Disk Replacement (Lab 4)** |
| **03:45 - 04:00** | Wrap-Up & Q&A | 

---

<!-- _class: lead -->
# Module 1
## TrueNAS Architecture & ZFS Fundamentals
*(09:30 – 10:15)*

---

## What is TrueNAS?

- **Enterprise Open-Source Storage Operating System** developed by iXsystems.
- Powered by the **OpenZFS** file system (128-bit file system and volume manager).
- based on Debian operating system
- used Linux KVM virtualization and Docker container support
- Modern hardware driver compatibility and Scale-Out clustering

---

## Why ZFS? The Foundation of TrueNAS

Traditional Storage Stack:
$$\text{Hardware RAID Controller} \longrightarrow \text{Volume Manager (LVM)} \longrightarrow \text{File System (ext4/NTFS)}$$

**ZFS Unified Architecture:**
$$\text{Raw Physical Disks (HBA)} \longrightarrow \mathbf{OpenZFS} \text{ (Volume Manager + File System + Cache)}$$

---

## Why ZFS? The Foundation of TrueNAS (cont.)

### Core ZFS Superpowers:
- **Copy-on-Write (CoW)**: Data is never overwritten in-place; prevents silent file corruption on crashes.
- **End-to-End Data Integrity**: Cryptographic 256-bit checksums for every data and metadata block.
- **Self-Healing**: Automatically detects silent data corruption ("bit rot") and repairs it using redundant parity.
- **Instant Snapshots & Clones**: Zero-cost, immutable point-in-time states.

---

## The ZFS Storage Hierarchy

```text
+-------------------------------------------------------------+
|                      DATASETS & ZVOLS                       |
|  /warisan_pool/documents (Dataset: POSIX/NFSv4 ACL)         |
|  /warisan_pool/multimedia (Dataset: Custom Recordsize)      |
|  /warisan_pool/vm-disk-01 (Zvol: Block device for iSCSI)    |
+-------------------------------------------------------------+
															|
+-------------------------------------------------------------+
|                        ZFS POOL                             |
|  (e.g., "warisan_pool" - aggregates capacity and IOPS)      |
+-------------------------------------------------------------+
               |                               |
        +---------------+               +---------------+
        |    VDEV 1     |               |    VDEV 2     |
        |  (RAIDZ2)     |               |  (RAIDZ2)     |
        +---------------+               +---------------+
         /   |   |   \                   /   |   |   \
       [D1] [D2] [D3] [D4]             [D5] [D6] [D7] [D8]
```

---

## Understanding VDEVs (Virtual Devices)

- A **VDEV** is a logical group of physical disks providing redundancy.
- **A Pool is composed of one or more VDEVs.**
- Data is **striped** across all VDEVs in the pool.

<div class="warning-box">
<strong>CRITICAL RULE OF ZFS:</strong><br>
If a single VDEV fails with unrecoverable data loss, <strong>THE ENTIRE POOL IS LOST!</strong><br>
Never create a storage pool using non-redundant VDEVs (Stripe) for production archival data!
</div>

---

## Redundancy Options Comparison

| Topology | Min Disks | Fault Tolerance | Usable Capacity | Ideal Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **Mirror** | 2 | 1 disk per pair (50%) | 50% | High IOPS, Databases, VMs |
| **RAIDZ1** | 3 | 1 disk failure | $(N-1) \times \text{Disk Size}$ | Low budget, non-critical (Not recommended for $>4\text{TB}$ drives) |
| **RAIDZ2** | 4 | **2 disk failures** | $(N-2) \times \text{Disk Size}$ | **Standard for Archives & Warisan Media** |
| **RAIDZ3** | 5 | 3 disk failures | $(N-3) \times \text{Disk Size}$ | Ultra-high capacity drives ($>16\text{TB}$) |

<div class="tip-box">
<strong>UKM Warisan Recommendation:</strong> RAIDZ2 provides the optimal balance of capacity and safety during the long resilvering process of high-capacity archival drives.
</div>

---

## Datasets vs. Zvols

### Datasets (File-level)
- Behaves like a rich directory or individual filesystem.
- Inherits or overrides properties: **Compression**, **Quotas**, **ACL Type**, **Recordsize**.
- Used for NFS and local file storage.

### Zvols (Block-level)
- Presents as a raw virtual block device (`/dev/zvol/...`).
- Used as virtual hard disks for Virtual Machines or exported over **iSCSI**.

---

## Datasets vs. Zvols (cont.)

### ARC & L2ARC (Caching)
- **ARC (Adaptive Replacement Cache)**: Lives in RAM. ZFS uses all free RAM as read cache!
- **SLOG (Separate ZFS Intent Log)**: Dedicated ultra-fast SSD for synchronous writes.

---

<!-- _class: lead -->
# Morning Refreshment Break ☕
*(10:15 – 10:30)*

*Next up: Initial WebUI Setup, Network Configuration & Storage Pool Creation.*

---

<!-- _class: lead -->
# Module 2
## System Setup, Networking & Storage Pools
*(10:30 – 11:30)*

---

## TrueNAS Web UI & Initial Walkthrough

- Access via browser: `https://<truenas-ip>`
- **Default Dashboard**:
  - System Information (Uptime, TrueNAS SCALE Version, CPU/Memory utilization)
  - Storage Pool Health & Capacity Gauge
  - Active Network Interfaces & Throughput
- **Key Navigation Menu Items**:
  - **Storage**: Pools, VDEVs, Disks, Disks Management, Topology
  - **Datasets**: Dataset tree, permissions, quotas, encryption
  - **Shares**: Unix (NFS), Block (iSCSI)

---

## TrueNAS Web UI & Initial Walkthrough (cont.)

  - **Credentials**: Local Users, Local Groups, Directory Services (Active Directory / LDAP)
  - **Data Protection**: Periodic Snapshots, Cloud Sync, Replication Tasks, Scrub Tasks

---

## Network Architecture & Best Practices

<div class="grid-2">
<div>

### Key Network Concepts
- **Static IP Assignment**: Essential for servers. 
- **DNS & Gateway**: Required for cloud sync, and system catalog updates.
- **Link Aggregation (LACP / bond0)**: Combines multiple NICs for redundancy and bandwidth.
- **MTU (Jumbo Frames)**: Standard 1500 vs 9000 MTU (Only enable if switch supports it!).

</div>
<div>

### WebUI Configuration Path
1. Go to **Network** $\to$ **Interfaces**
2. Edit target adapter (e.g., `enp3s0`)
3. Uncheck **DHCP**
4. Add IP / Subnet:  
   `192.168.10.50/24`
5. Configure Default Route:  
   `192.168.10.1`
6. Add DNS: `8.8.8.8`, `1.1.1.1`
7. Click **Apply Changes** & **Test Changes** (60-sec auto-revert safety window).

</div>
</div>

---

## Storage Pool Creation Best Practices

1. **Keep VDEV Sizes Sane**:
   - RAIDZ1: 3 to 5 disks
   - RAIDZ2: 6 to 10 disks (6 disks is a sweet spot for Warisan archives)
2. **Never Mix Drive Capacities or RPMs** inside the same VDEV:
   - ZFS will limit capacity of each drive to the smallest disk in the VDEV.
3. **Use CMR (Conventional Magnetic Recording) Disks**:
   - Avoid SMR (Shingled Magnetic Recording) drives—they can cause catastrophic pool drops during resilvering!
4. **Reserve Pool Headroom**:
   - Keep pool utilization **below 80%** to prevent severe write fragmentation.

---

## Dataset Properties & Optimization

| Property | Default | Recommended Setting | Rationale |
| :--- | :--- | :--- | :--- |
| **Compression** | `lz4` | `lz4` or `zstd` | Free CPU-level compression; actually *speeds up* disk I/O! |
| **Recordsize** | `128K` | `128K` (general) / `1M` (video/audio) / `16K` (databases) | Match chunk size to typical file sizes for archival performance. |
| **Atime (Access Time)** | `On` | `Off` | Reduces unnecessary metadata writes every time a file is read. |
| **Case Sensitivity** | `Sensitive` | `Sensitive` | Standard Linux/Unix filesystem convention. |
| **Quota** | `None` | Set per departmental dataset | Prevents one team from exhausting the entire repository. |

---

## 🛠️ Hands-on Lab 1: Pool & Dataset Setup

### Objective
Create your primary storage pool `warisan_pool` and configure structured departmental datasets with compression and quotas.

### Step-by-Step Tasks:
1. Navigate to **Storage** $\to$ Click **Create Pool**.
2. Name the pool: `warisan_pool`.
3. Add 4 disks into a **RAIDZ1** or **RAIDZ2** VDEV. Click **Create Pool**.


---

## 🛠️ Hands-on Lab 1: Pool & Dataset Setup (cont.)

### Step-by-Step Tasks:
4. Navigate to **Datasets** $\to$ Select `warisan_pool` $\to$ Click **Add Dataset**:
   - Name: `archives` | Compression: `zstd` | Recordsize: `1M` (Archival media)
5. Create a second dataset:
   - Name: `general-documents` | Compression: `lz4` | Dataset Quota: `10 GiB`
6. Verify dataset inheritance and properties in the dataset tree.

---

<!-- _class: lead -->
# Module 3
## Access Control & Network Sharing (NFS)
*(11:30 – 12:45)*

---

## User & Group Management

- **Root is for Administration only!** Never use root or share credentials across users for file access.
- **Local Users & Groups**:
  - Located under **Credentials** $\to$ **Local Users** & **Local Groups**
  - Always assign users to specific organizational groups (e.g., `students`, `researchers`).
  - TrueNAS automatically assigns a unique UID/GID to each user/group.

```text
User: 'ahmad'   ──> Primary Group: 'student' ──> Read/Write to /warisan_pool/archives
User: 'sarah'   ──> Primary Group: 'researchers' ──> Read-Only to /warisan_pool/archives
```

---

## Permissions: POSIX vs. NFSv4 ACLs

<div class="grid-2">
<div>

### POSIX Permissions (Classic Unix)
- Basic 3-tier model: **Owner**, **Group**, **Others**.
- Modes: Read (`r`), Write (`w`), Execute (`x`).
- Example: `chmod 770 /mnt/warisan_pool/data`
- Limitation: Cannot grant custom permissions to multiple separate groups.

</div>
<div>

### NFSv4 ACLs (Enterprise Granular)
- Fine-grained Access Control Lists.
- Multiple Users and Groups can have distinct rights on the same dataset/directory.
- Supports **Inheritance** (Child files & folders automatically inherit parent rules).
- Standard access model for enterprise Unix/Linux environments.

</div>
</div>

---

## Configuring Network File System (NFS) Exports

- **Network File System (NFS)**: The primary open standard for Linux servers, HPC clusters, and archival ingestion scripts.
- **Key NFS Settings in TrueNAS**:
  - **Path**: Target dataset (e.g., `/mnt/warisan_pool/general-documents`)
  - **Networks**: Restrict access to authorized client subnets (e.g., `192.168.10.0/24`)
  - **Hosts**: Restrict access to specific client IP addresses
  - **Access Mode**: Read-Only (`ro`) vs Read-Write (`rw`)
  - **Maproot User / Maproot Group**: Security defense mapping remote client root calls to local TrueNAS user accounts (prevents unauthorized root takeover)

---

## Mounting NFS Shares on Linux Clients

<div class="grid-1">
<div>

### From Linux Client:
```bash
# 1. Install NFS client utilities
sudo apt update && sudo apt install nfs-common -y

# 2. Create mount point & mount export
sudo mkdir -p /mnt/warisan-docs
sudo mount -t nfs 192.168.10.50:/mnt/warisan_pool/general-documents /mnt/warisan-docs

# 3. Persistent mount in /etc/fstab:
# 192.168.10.50:/mnt/warisan_pool/general-documents /mnt/warisan-docs nfs defaults 0 0
```
</div>

---

## 🛠️ Hands-on Lab 2: User Access Control & NFS Sharing

### Objective
Create dedicated user accounts, configure ACL permissions, and export datasets securely over NFS.

### Step-by-Step Tasks:
1. Go to **Credentials** $\to$ **Local Groups** $\to$ Create group `researchers`.
2. Go to **Credentials** $\to$ **Local Users** $\to$ Create user `researcher1` (set password, assign to `researchers`).

---

## 🛠️ Hands-on Lab 2: User Access Control & NFS Sharing (cont.)

### Step-by-Step Tasks:
3. Go to **Datasets** $\to$ Select `general-documents` $\to$ Click **Edit Permissions**:
   - Apply an **NFSv4 ACL** preset (`RESTRICTED` or `HOME`).
   - Add an ACL item: Group `researchers` $\to$ **Full Control** $\to$ Apply recursively.
4. Go to **Shares** $\to$ **Unix Shares (NFS)** $\to$ Add export for `/mnt/warisan_pool/general-documents`:
   - Set authorized network (e.g., `192.168.10.0/24` or client IP).
   - Start the **NFS Service**.
5. On your client machine, mount the NFS share and create a test directory to verify write access!

---

<!-- _class: lead -->
# Lunch Break & Solat 🍱
*(01:00 – 02:00)*

*Enjoy your lunch!*  
*Afternoon session resumes at 2:00 PM with:*  
*Data Protection, Snapshots, Disaster Recovery & Disk Maintenance.*

---

<!-- _class: lead -->
# Module 4
## Data Protection, Snapshots & Disaster Recovery
*(02:00 – 03:00)*

---

## The Magic of ZFS Snapshots

- A snapshot is a **read-only, frozen record** of a dataset at an exact second in time.
- **Zero Space Initially**: Takes 0 bytes when first created!
- Only consumes storage space as underlying data blocks are modified or deleted.
- Creating a snapshot takes **less than 1 millisecond**, regardless of whether the dataset is 10 GB or 100 TB.
- **Ransomware Protection**: ZFS snapshots are immutable—even if ransomware encrypts your network share, you can roll back the entire dataset in seconds!

---

## Copy-on-Write (CoW) in Action

```text
1. INITIAL STATE:
   Snapshot "snap1" ---> [ Block A ] [ Block B ] [ Block C ]

2. USER MODIFIES BLOCK B TO B':
   ZFS writes Block B' to a NEW physical location:
   Active Dataset   ---> [ Block A ] [ Block B' ] [ Block C ]
                               \
   Snapshot "snap1" -----------> [ Block B ] (Preserved!)
```

- When you delete or modify files, the old blocks remain locked by the snapshot.
- To free disk space, you must delete old snapshots that reference those blocks.

---

## Snapshot Access via the Hidden `.zfs/snapshot` Directory

### Built-in Self-Service Snapshot Recovery:
- Every ZFS dataset maintains a hidden, read-only directory: `.zfs/snapshot`
- Accessible directly over **NFS** mounts and local CLI without any special client software!

<div class="grid-2">
<div>

### Browsing Snapshots on Client:
```bash
cd /mnt/warisan-docs
ls -la .zfs/snapshot
# Output:
# auto-2026-09-12_00-00
# manual-backup-01
```

</div>
<div>

### Instant Self-Service Restore:
```bash
# Users can copy deleted or older file versions:
cp .zfs/snapshot/manual-backup-01/letter.txt ./
```
*Zero downtime, zero administrator intervention!*

</div>
</div>

---

## Periodic Snapshot Tasks & Retention Policy

<div class="grid-2">
<div>

### Configuring Automated Tasks
- Path: **Data Protection** $\to$ **Periodic Snapshot Tasks** $\to$ **Add**
- Select Dataset: `/warisan_pool/archives`
- **Recursive**: Check to snapshot all child datasets.
- **Schedule**: Cron preset (e.g., Hourly, Daily).

</div>
<div>

### Recommended Schedule
- **Frequent**: Every 1 hour (Keep for 24 hours)
- **Daily**: Every midnight (Keep for 14 days)
- **Weekly**: Every Sunday (Keep for 8 weeks)
- **Monthly**: 1st of month (Keep for 12 months)

*TrueNAS automatically purges expired snapshots according to your retention policy!*

</div>
</div>

---

## Replication Tasks & The 3-2-1 Backup Rule

$$\mathbf{3} \text{ Copies of Data} \quad\vert\quad \mathbf{2} \text{ Different Media} \quad\vert\quad \mathbf{1} \text{ Offsite Copy}$$

```text
+-----------------------+     ZFS Send / Recv      +-----------------------+
|  Primary TrueNAS      | ───────────────────────> |  Secondary TrueNAS    |
|  UKM Data Centre      |   (Encrypted SSH Sync)   |  Offsite / DR Node    |
+-----------------------+                          +-----------------------+
```

### ZFS Replication Advantage:
- Uses `zfs send` and `zfs recv`.
- Only transfers changed blocks (delta), not whole files.
- Extremely efficient over WAN / internet links.
- Can be pushed to an offsite TrueNAS or cloud storage via **Cloud Sync Tasks** (Backblaze B2, AWS S3, Google Cloud).

---

## 🛠️ Hands-on Lab 3: Snapshots, Rollback & Recovery

### Objective
Create manual and scheduled snapshots, simulate an accidental data deletion disaster, and perform an instant recovery.

### Step-by-Step Tasks:
1. In your mounted NFS share (`/mnt/warisan-docs`), create a directory `important_heritage_docs` with 3 text files.
2. In TrueNAS WebUI $\to$ Go to **Datasets** $\to$ Select `general-documents` $\to$ Click **Create Snapshot**:
   - Name: `manual-pre-deletion-backup`
3. Simulate disaster: On your client, delete the entire folder (`rm -rf important_heritage_docs`)!

---

## 🛠️ Hands-on Lab 3: Snapshots, Rollback & Recovery (cont.)

### Step-by-Step Tasks:
4. Recover via WebUI:
   - Go to **Datasets** $\to$ **Manage Snapshots**
   - Find `manual-pre-deletion-backup` $\to$ Click **Rollback Dataset**.
5. Check your client mount: **All files are instantly restored!** (Also browse `.zfs/snapshot` for file-level recovery).

---

<!-- _class: lead -->
# Module 5
## System Health, Monitoring & Disk Replacement
*(03:00 – 03:45)*

---

## Preventive Maintenance: Scrubbing & SMART Tests

<div class="grid-2">
<div>

### ZFS Pool Scrubbing
- **What it does**: Reads all data blocks and verifies their 256-bit checksums.
- Automatically repairs corrupted blocks using parity.
- **Frequency**: Run **bi-weekly or monthly**.
- Path: **Data Protection** $\to$ **Scrub Tasks**.

</div>
<div>

### SMART Tests
- **Self-Monitoring, Analysis, and Reporting Technology** built into hard drives.
- Detects pre-failure conditions (bad sectors, reallocated sectors, spin retries).
- **Short SMART Test**: Daily/Weekly (Tests drive electronics & heads).
- **Long SMART Test**: Monthly (Reads entire disk surface).

</div>
</div>

---

## System Alerting & Notifications

- Never wait for a user to report a slow server!
- Configure TrueNAS to email you immediately when a drive throws errors.
- **Setup Path**:
  1. Click the **Bell Icon** (top right) $\to$ Click **Alert Settings**.
  2. Set alert severity (Critical, Warning, Info).
  3. Go to **System Settings** $\to$ **General** $\to$ **Email**:
     - Configure SMTP Server (e.g., UKM mail gateway or institutional SMTP).
     - Send a test email to verify delivery.
  4. Optional: Integrate with Slack / Discord / Telegram Webhooks for instant mobile alerts.

---

## Diagnosing Degraded Pools

```bash
# Check pool status via Shell / CLI
admin@truenas:~$ zpool status warisan_pool
```
```text
  pool: warisan_pool
 state: DEGRADED
status: One or more devices could not be used because the issue is hardware-related.
action: Replace the device using 'zpool replace'.
  scan: scrub repaired 0B in 01:14:22 with 0 errors on Sun Sep 01 02:14:22 2026
config:
	NAME                      STATE     READ WRITE CKSUM
	warisan_pool              DEGRADED     0     0     0
	  raidz2-0                DEGRADED     0     0     0
	    sdb                   ONLINE       0     0     0
	    sdc                   FAULTED      8   120     0  too many errors
	    sdd                   ONLINE       0     0     0
	    sde                   ONLINE       0     0     0
```

---

## Failed Disk Replacement Procedure (WebUI)

```text
Identify Failed Disk Serial Number ──> Take Disk Offline ──> Physically Swap Drive
                     │
                     └──> Initiate Disk Replacement in GUI ──> Resilver Completes
```

1. **Identify the bad disk**: Note the serial number under **Storage** $\to$ **Disks**.
2. **Offline the failed drive**:
   - Go to **Storage** $\to$ Click **Manage Devices** on `warisan_pool`.
   - Select the faulted drive $\to$ Click **Offline**.
3. **Replace the hardware**: Physically pull the drive and insert the replacement disk.
4. **Initiate replacement**:
   - In TrueNAS, click on the offline disk $\to$ Click **Replace**.
   - Select the new drive from the dropdown $\to$ Confirm.
5. **Monitor Resilvering**: Watch progress bar until pool status returns to `ONLINE`.

---

## System Configuration Backup: Golden Rule!

<div class="warning-box">
<strong>CRITICAL DISASTER RECOVERY TIP:</strong><br>
All TrueNAS system configurations (Users, Shares, Network, Pool setups) are stored in a tiny SQLite database.
If your TrueNAS boot drive burns out, you can reinstall TrueNAS on a fresh drive, upload your <code>.tar</code> config backup, and be 100% restored in <strong>5 minutes!</strong>
</div>

### How to Backup System Config:
1. Go to **System Settings** $\to$ **General**.
2. Scroll to **Save Configuration**.
3. Check **Export Password Secret Seed** *(essential for decrypting credentials!)*.
4. Click **Save**: Downloads a lightweight `.tar` file to your PC.
5. Store this backup securely after every major administrative change!

---

## 🛠️ Hands-on Lab 4: Disk Failure & Recovery Simulation

### Objective
Simulate a hard drive failure, inspect degraded status, and execute a disk replacement and resilver.

### Step-by-Step Tasks:
1. Open TrueNAS Shell or WebUI **Storage** $\to$ **Manage Devices**.
2. Simulate failure / offline:
   - Select disk `sdd` $\to$ Click **Offline** (Pool transitions to `DEGRADED`).
3. Check the Dashboard alert banner: Notice the warning notification.

---

## 🛠️ Hands-on Lab 4: Disk Failure & Recovery Simulation (cont.)

### Step-by-Step Tasks:
4. Run `zpool status warisan_pool` in the Shell to examine the degraded VDEV.
5. In WebUI, click **Replace** on the offline disk and select an available spare disk.
6. Observe the **Resilver** process in real-time until the pool status returns to healthy `ONLINE`.

---

<!-- _class: lead -->
# Summary & Course Wrap-Up
*(03:45 – 04:00)*

---

## Key Best Practices Checklist

- [x] **Never use Single Stripe VDEVs** for critical institutional data; use **RAIDZ2** for archives.
- [x] **Keep Pool capacity below 80%** to avoid severe performance degradation.
- [x] **Always enable Compression** (`lz4` or `zstd`) on all datasets.
- [x] **Automate Periodic Snapshots** with layered retention (Hourly, Daily, Monthly).
- [x] **Enable .zfs/snapshot visibility** so users can restore deleted files independently over NFS.
- [x] **Schedule Bi-weekly Scrubs & SMART tests** to catch silent corruption early.
- [x] **Download a Configuration Backup (`.tar`)** every time you make configuration changes.

---

## Final Q&A 

---

<!-- _class: lead -->
# Thank You!
### UKM Warisan TrueNAS Storage Administration

Keep your quick-reference guide handy!
Use the lab-preparation to simulate TrueNAS in local setting, before changing any production configuration.
