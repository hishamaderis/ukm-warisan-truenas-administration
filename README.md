# UKM Warisan — TrueNAS Storage Administration Training

> **Enterprise Storage Administration for Beginners & Digital Heritage Preservation**  
> A 1-day, hands-on technical bootcamp focused on **TrueNAS SCALE** and **OpenZFS** storage architecture, data resilience, access control, disaster recovery, and preventive maintenance.

---

## 📌 Overview

This repository contains the training materials, slide decks, lab preparation guides, and fast-reference cheat sheets for the **TrueNAS Administration Workshop** tailored for **UKM Warisan**.

UKM Warisan manages invaluable institutional digital assets—including high-resolution historical documents, oral history audio, archival video footage, and research databases. This curriculum is designed to equip systems administrators, IT officers, and technical custodians with the practical skills needed to deploy, configure, secure, and maintain enterprise-grade TrueNAS SCALE infrastructure.

---

## 📚 Repository Structure

| File | Description |
| :--- | :--- |
| [**`SLIDES.md`**](SLIDES.md) | Full Marp-powered presentation deck covering Modules 0 through 5, architecture diagrams, lab walk-throughs, and review quizzes. |
| [**`lab-preparation.md`**](lab-preparation.md) | Comprehensive step-by-step lab setup guide for VMware Workstation Pro/Player on both **Linux** and **Windows** hosts. |
| [**`quick-reference.md`**](quick-reference.md) | One-page cheat sheet containing TrueNAS WebUI fast-finders and essential ZFS CLI commands (`zpool`, `zfs`). |

---

## ⏱️ Training Agenda & Curriculum

The training is structured into a 1-day intensive workshop (9:00 AM – 4:00 PM):

| Time | Session / Module | Core Topics | Hands-On Lab |
| :--- | :--- | :--- | :--- |
| **09:00 – 09:30** | **Orientation & Lab Verification** | Environment verification, IP connectivity, WebUI check | Lab Verification |
| **09:30 – 10:15** | **Module 1: TrueNAS & ZFS Fundamentals** | OpenZFS architecture, Copy-on-Write (CoW), Self-Healing, Bit rot protection, VDEV topologies (Mirror, RAIDZ1, RAIDZ2) | Conceptual Review |
| **10:15 – 10:30** | ☕ *Morning Refreshment Break* | — | — |
| **10:30 – 11:30** | **Module 2: Setup, Networking & Storage Pools** | TrueNAS Console Setup, static IP, DNS, storage pool creation, dataset properties (compression, recordsize, quotas) | **Lab 1: Pool & Dataset Setup** |
| **11:30 – 13:00** | **Module 3: Access Control & Network Sharing (NFS)** | Local Users & Groups, POSIX vs. NFSv4 ACL permissions, UNIX NFS exports, Linux/Windows host mounting | **Lab 2: Access Control & NFS Sharing** |
| **13:00 – 14:00** | 🍱 *Lunch Break & Solat* | — | — |
| **14:00 – 15:00** | **Module 4: Data Protection & Disaster Recovery** | Instant snapshots, hidden `.zfs/snapshot` access, one-click rollbacks, dataset cloning, replication tasks, 3-2-1 backup strategy | **Lab 3: Snapshots, Rollback & Recovery** |
| **15:00 – 15:45** | **Module 5: System Health, Monitoring & Maintenance** | Alerts & email notifications, automated SMART tests, pool scrubs, configuration database backups, drive failure & resilvering | **Lab 4: Disk Failure & Resilvering Simulation** |
| **15:45 – 16:00** | **Wrap-Up & Assessment** | Best practices checklist, interactive quiz review, Q&A | Course Wrap-Up |

---

## 🛠️ Hands-On Labs Summary

Each participant works within their own virtualized TrueNAS SCALE node configured with 5 virtual disks:

1. **Lab 1: Storage Pool & Hierarchical Datasets**
   - Provision a resilient `warisan_pool` using 4 disks in a RAIDZ1/RAIDZ2 topology.
   - Configure optimized datasets: `archives` (with `zstd` compression and 1M recordsize for media) and `general-documents` (with `lz4` compression and a 10 GiB quota).

2. **Lab 2: Granular Access Control & NFS Exports**
   - Create local accounts (`researcher1`) and groups (`researchers`).
   - Configure NFSv4 ACL permissions with fine-grained access rules.
   - Publish NFS exports and mount the share on a Linux workstation or native Windows NFS client.

3. **Lab 3: Immutable Snapshots, Ransomware & Accidental Deletion Recovery**
   - Populate live data onto the NFS mount and trigger manual and periodic snapshots.
   - Simulate a catastrophic accidental deletion (`rm -rf`) of critical archival assets.
   - Execute an instant dataset rollback and browse the read-only `.zfs/snapshot` tree to perform individual file recovery.

4. **Lab 4: Drive Failure Simulation & Online Resilver**
   - Simulate a failed/offline disk drive (`sdd`) to transition `warisan_pool` into a `DEGRADED` state.
   - Inspect diagnostic outputs via TrueNAS WebUI and shell (`zpool status`).
   - Replace the damaged drive with an unassigned spare disk and observe live ZFS resilvering until health is restored to `ONLINE`.

---

## 💻 Lab Hardware & System Prerequisites

Before starting the workshop, participants must set up their local hypervisor following [**`lab-preparation.md`**](lab-preparation.md).

### Host Requirements
- **OS**: Linux (Ubuntu, Debian, Fedora, Arch) or Windows (10/11 64-bit with CPU Virtualization enabled).
- **CPU**: 64-bit multi-core processor with hardware virtualization (VT-x / AMD-V) enabled in BIOS/UEFI.
- **RAM**: Minimum **16 GB** on host (8 GB allocated to the TrueNAS VM).
- **Free Disk Space**: Minimum **120 GB** free on an SSD.

### Virtual Machine Specifications
- **Hypervisor**: VMware Workstation Pro 17.x or VMware Player.
- **Guest OS**: Linux / Debian 12.x 64-bit.
- **vCPU & RAM**: 2–4 vCPUs | 8 GB – 12 GB RAM.
- **Virtual Disks**:
  - `sda` (20 GB SCSI): Dedicated OS boot disk (`boot-pool`).
  - `sdb`, `sdc`, `sdd`, `sde` (4x 20 GB SCSI): Unassigned data drives for `warisan_pool`.
- **Default Lab Credentials**:
  - **Username**: `admin`
  - **Password**: `Ukmtruenas2026!`

---

## 🖥️ How to View & Present the Slides

The training slides in [**`SLIDES.md`**](SLIDES.md) are built using [Marp](https://marp.app/) (Markdown Presentation Ecosystem).

### Option 1: VS Code (Recommended)
1. Install the **Marp for VS Code** extension (`marp-team.marp-vscode`).
2. Open [`SLIDES.md`](SLIDES.md).
3. Click the Marp preview icon in the top-right toolbar or press `Ctrl + K, V` to open live presentation view.
4. Export to PDF or HTML via the Marp icon dropdown.

### Option 2: Marp CLI
Run the following commands using `npx` (requires Node.js):

```bash
# Preview slides in your default browser with live-reload
npx @marp-team/marp-cli@latest -s .

# Export slides to standalone HTML
npx @marp-team/marp-cli@latest SLIDES.md -o slides.html

# Export slides to PDF
npx @marp-team/marp-cli@latest SLIDES.md --pdf -o slides.pdf
```

---

## ⚡ Quick Cheat Sheet Sample

For the full cheat sheet, refer to [**`quick-reference.md`**](quick-reference.md).

```bash
# Check overall health and detect degraded drives or checksum errors
zpool status warisan_pool

# Check pool capacity and allocation
zpool list

# Real-time disk I/O metrics and latency (refreshes every 2s)
zpool iostat -v warisan_pool 2

# Take an instant snapshot
zfs snapshot warisan_pool/archives@snapshot-name

# Instant rollback to a previous point-in-time
zfs rollback warisan_pool/archives@snapshot-name

# Start a storage integrity scrub
zpool scrub warisan_pool
```

