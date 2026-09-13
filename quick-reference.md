# TrueNAS & ZFS Quick Reference Cheat Sheet
**UKM Warisan Storage Administration**

---

## 🧭 WebUI Navigation Fast-Finder

| What you want to do | Where to find it in WebUI |
| :--- | :--- |
| **Check Pool Health & Capacity** | **Storage** or **Dashboard** |
| **View / Add Disks to Pool** | **Storage** $\to$ **Manage Devices** |
| **Create or Edit Datasets** | **Datasets** |
| **Configure Dataset Permissions (ACLs)** | **Datasets** $\to$ Select dataset $\to$ **Permissions** $\to$ **Edit** |
| **Create / Manage NFS Exports** | **Shares** $\to$ **Unix Shares (NFS)** |
| **Add Users & Groups** | **Credentials** $\to$ **Local Users** / **Local Groups** |
| **Automate Snapshots** | **Data Protection** $\to$ **Periodic Snapshot Tasks** |
| **Automate Pool Scrubs** | **Data Protection** $\to$ **Scrub Tasks** |
| **Automate SMART Tests** | **Data Protection** $\to$ **S.M.A.R.T. Tests** |
| **Set up Remote Replication** | **Data Protection** $\to$ **Replication Tasks** |
| **Configure Static IP / DNS / Gateway** | **Network** $\to$ **Interfaces** & **Global Configuration** |
| **Configure Email Alerts** | **System Settings** $\to$ **General** $\to$ **Email** |
| **Download Configuration Backup** | **System Settings** $\to$ **General** $\to$ **Save Configuration** |
| **System Terminal / Shell** | **System Settings** $\to$ **Shell** (or top-right `>_` icon) |

---

## 💻 Essential ZFS CLI Commands

### 1. Pool Status & Health
```bash
# Check detailed status of all pools (identifies degraded drives, checksum errors)
zpool status

# Check specific pool status
zpool status warisan_pool

# View pool capacity, fragmentation, and deduplication ratio
zpool list

# View real-time I/O throughput and latency per disk (refreshes every 2 seconds)
zpool iostat -v warisan_pool 2

# View history of all administrative zpool commands executed
zpool history warisan_pool
```

### 2. Dataset Management
```bash
# List all datasets and their mount points
zfs list

# List datasets with compression ratio and quota information
zfs list -o name,used,avail,refer,quota,compressratio,mountpoint

# Set compression on a dataset
zfs set compression=zstd warisan_pool/archives

# Set a 50GB quota on a dataset
zfs set quota=50G warisan_pool/general-documents

# View all properties of a specific dataset
zfs get all warisan_pool/archives
```

### 3. Snapshots & Clones
```bash
# Take a manual snapshot
zfs snapshot warisan_pool/archives@backup-2026-09-12

# List all snapshots
zfs list -t snapshot

# Roll back a dataset to a snapshot (WARNING: Overwrites subsequent changes)
zfs rollback warisan_pool/archives@backup-2026-09-12

# Clone a snapshot into a new writable dataset
zfs clone warisan_pool/archives@backup-2026-09-12 warisan_pool/archives-test-branch

# Destroy an unwanted snapshot to free space
zfs destroy warisan_pool/archives@backup-2026-09-12
```

### 4. Pool Maintenance & Disk Operations
```bash
# Start a pool scrub
zpool scrub warisan_pool

# Stop / pause a running scrub
zpool scrub -s warisan_pool

# Take a failing disk offline before replacement
zpool offline warisan_pool sdc

# Bring a disk back online
zpool online warisan_pool sdc

# Replace a failed disk with a new drive
zpool replace warisan_pool sdc /dev/sdf

# Clear error counters after fixing a problem
zpool clear warisan_pool
```

### 5. Drive Diagnostics (SMART)
```bash
# Check drive health status
smartctl -H /dev/sdb

# View detailed SMART attributes (reallocated sectors, power hours)
smartctl -A /dev/sdb

# Run a short SMART test manually
smartctl -t short /dev/sdb

# View SMART test log results
smartctl -l selftest /dev/sdb
```

---

## 🚨 Troubleshooting Rules of Thumb

1. **Pool is `DEGRADED`**:
   - Check which drive is `FAULTED` or `OFFLINE` with `zpool status`.
   - Ensure cables are seated properly.
   - If drive has excessive `READ` or `WRITE` errors, initiate physical replacement immediately.
2. **Cannot write to share (Disk Full error) even though pool has space**:
   - Check if a dataset quota has been reached: `zfs get quota,refquota <dataset>`.
   - Check if snapshots are holding onto deleted files.
3. **NFS Share Permission Denied / Mount Refused**:
   - Verify that client IP is listed under **Networks** or **Hosts** in the NFS Share settings.
   - Check the dataset ACL in **Datasets** $\to$ **Permissions**. Ensure the user/group has appropriate read/write rights.
   - Verify that Maproot User / Group or Mapall User / Group is set appropriately if root client access is needed.
4. **Resilver is slow**:
   - Resilvering prioritizes data integrity. If performance is critical, TrueNAS SCALE allows setting resilver priority under System Advanced settings.
