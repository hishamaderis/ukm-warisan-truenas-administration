# Local Lab Preparation for TrueNAS SCALE
## VMware Workstation Setup Guide (Linux & Windows Hosts)

This guide provides step-by-step instructions to set up a fully functional TrueNAS SCALE virtual lab environment using **VMware Workstation Pro / Player** on either **Linux** or **Windows** host operating systems.

---

## Lab Architecture & Virtualization Strategy

To master TrueNAS SCALE without physical rack hardware, we simulate an enterprise storage server using VMware Workstation.

```text
+-----------------------------------------------------------------------------+
|                     Host Workstation (Linux or Windows 10/11)               |
|            [Web Browser (Firefox/Chrome/Edge): TrueNAS WebUI Access]        |
|            [Terminal / PowerShell: SSH, Ping, Network Verification]         |
+-----------------------------------------------------------------------------+
                                       |
                       (Bridged or NAT Network / VMnet)
                                       |
+-----------------------------------------------------------------------------+
|                        TrueNAS SCALE Virtual Machine                        |
|  +-----------------------------------------------------------------------+  |
|  |              TrueNAS SCALE OS (Debian Linux Base)                     |  |
|  +-----------------------------------------------------------------------+  |
|  |       boot-pool (20 GB)        |          warisan_pool (RAIDZ)        |  |
|  +--------------------------------+--------------------------------------+  |
|  | Disk 1 (sda - 20 GB OS Drive)  | Disks 2–5 (sdb, sdc, sdd, sde - 4x20GB) |  |
|  +--------------------------------+--------------------------------------+  |
+-----------------------------------------------------------------------------+
```

### Lab Specifications
- **Hypervisor**: VMware Workstation Pro 17.x or VMware Workstation Player (free for personal use).
- **Guest OS**: TrueNAS SCALE 24.x (Dragonfish) or 23.x (Cobia) — 64-bit.
- **Compute**: 2 vCPUs minimum (4 vCPUs recommended).
- **Memory**: 8 GB RAM minimum (12–16 GB recommended for ZFS ARC caching).
- **Virtual Storage Topology**:
  - **1x 20 GB SCSI Disk (`sda`)**: Dedicated OS boot drive (`boot-pool`).
  - **4x 20 GB SCSI Disks (`sdb`, `sdc`, `sdd`, `sde`)**: Unassigned data drives for downstream hands-on labs (Pool creation, RAIDZ1/RAIDZ2, Snapshots, and Disk Replacement).
- **Networking**: Bridged Adapter (preferred for LAN visibility) or NAT (`VMnet8`).

---

## Host Prerequisites

### A. If Your Host is Linux (Ubuntu, Debian, Fedora, RHEL, Arch)

1. **Verify Hardware Virtualization**:
   ```bash
   egrep -c '(vmx|svm)' /proc/cpuinfo
   # Output must be >= 1
   ```

2. **Verify VMware Services & Kernel Modules**:
   ```bash
   sudo systemctl status vmware
   sudo systemctl status vmware-networks
   ```

3. **Check Host Resources**:
   - Ensure at least **120 GB free disk space** (`df -h`).
   - Ensure at least **16 GB RAM** on the host (`free -h`).

---

### B. If Your Host is Windows (Windows 10 / Windows 11 64-bit)

1. **Verify Hardware Virtualization in BIOS/UEFI**:
   - Open **Task Manager** (`Ctrl + Shift + Esc`) $\to$ Click **Performance** tab $\to$ Select **CPU**.
   - Confirm that **Virtualization: Enabled** is displayed in the bottom-right details.
   - *Alternatively, check via PowerShell*:
     ```powershell
     Get-CimInstance Win32_Processor | Select-Object Name, VirtualizationFirmwareEnabled
     ```

2. **Verify VMware Workstation Installation & Services**:
   - Download and install **VMware Workstation Pro 17** for Windows (free for personal use from Broadcom).
   - Open PowerShell as Administrator and ensure VMware background services are running:
     ```powershell
     Get-Service -Name "VMware*"
     ```
   - If services are stopped, start them:
     ```powershell
     Start-Service -Name VMAuthdService, "VMware NAT Service", "VMnetDHCP"
     ```

3. **Check Host Resources**:
   - Ensure at least **120 GB free space** on an NTFS SSD drive (`C:` or `D:`).
   - Ensure at least **16 GB RAM** installed on your Windows machine.

---

### Common Prerequisite: Download TrueNAS SCALE ISO
- Download the official **TrueNAS SCALE ISO installer** from [ixsystems.com/download-truenas-scale](https://www.truenas.com/download-truenas-scale/).
- Save the ISO file to a convenient local path:
  - Linux: `~/Downloads/TrueNAS-SCALE.iso`
  - Windows: `C:\ISO\TrueNAS-SCALE.iso` (or `D:\VMs\ISO\TrueNAS-SCALE.iso`)

---

## Step 1: Create Virtual Machine in VMware Workstation

Follow these steps in VMware Workstation (the wizard is identical on Linux and Windows):

1. Launch **VMware Workstation**:
   - **Linux**: Run `vmware &` in terminal or search applications.
   - **Windows**: Launch **VMware Workstation Pro** from Start Menu or desktop shortcut.

2. Click **File** $\to$ **New Virtual Machine...** (or click **Create a New Virtual Machine** on the home tab).

3. Select **Custom (advanced)** $\to$ Click **Next**.

4. **Hardware Compatibility**: Choose **Workstation 17.x** (default) $\to$ Click **Next**.

5. **Guest Operating System Installation**:
   - Select **"I will install the operating system later"** $\to$ Click **Next**.
   - *(Important: Do NOT select "Installer disc image file (iso)" here to prevent VMware from triggering unattended automated installation).*

6. **Select Guest Operating System**:
   - Guest Operating System: **Linux**
   - Version: **Debian 12.x 64-bit** *(or Debian 11.x 64-bit)*
   - *(Rationale: TrueNAS SCALE is built upon Debian Linux; this ensures correct kernel optimizations and virtual drivers).*
   - Click **Next**.

7. **Virtual Machine Name & Storage Location**:
   - Virtual machine name: `UKM-TrueNAS-SCALE`
   - Location:
     - **Linux**: `/home/<user>/vmware/UKM-TrueNAS-SCALE`
     - **Windows**: `C:\VMs\UKM-TrueNAS-SCALE` (or `D:\VirtualMachines\UKM-TrueNAS-SCALE`)
   - Click **Next**.

8. **Firmware Type**:
   - Select **UEFI** *(Recommended)* or **BIOS**.
   - *(Note: If UEFI is selected, leave "Secure Boot" unchecked).*
   - Click **Next**.

---

## Step 2: CPU, Memory & Network Configuration

1. **Processor Configuration**:
   - Number of processors: `1`
   - Number of cores per processor: `2` (or `4` if your host CPU has 8+ cores)
   - Total processor cores: `2` to `4`
   - Virtualization engine *(Optional)*: Check **Virtualize Intel VT-x/EPT or AMD-V/RVI** (useful if running nested containers/apps in TrueNAS).
   - Click **Next**.

2. **Memory Allocation**:
   - Set RAM to at least **8192 MB (8 GB)**.
   - Recommended: **12288 MB (12 GB)** or **16384 MB (16 GB)**.
   - *(ZFS utilizes idle RAM for its Adaptive Replacement Cache (ARC); 8 GB is the production minimum for stable operations).*
   - Click **Next**.

3. **Network Connection Type**:
   - **Option A: Bridged Networking (Recommended for training labs)**:
     - Select **Use bridged networking**.
     - Check **Replicate physical network connection state**.
     - *Result*: The VM receives an independent IP address from your local router/DHCP subnet, making it directly accessible from any browser or NFS client on your network.
   - **Option B: NAT (Network Address Translation - Campus Wi-Fi Alternative)**:
     - Select **Use network address translation (NAT)**.
     - *When to use*: If connected to university 802.1X enterprise Wi-Fi (such as eduroam or UKM-Campus) which restricts multiple MAC addresses per port/radio. TrueNAS will obtain an IP from `VMnet8` and remain fully accessible from your host machine.
   - Click **Next**.

4. **Select I/O Controller Types**:
   - SCSI Controller: **LSI Logic SAS (Recommended)**.
   - Click **Next**.

5. **Select a Disk Type**:
   - Virtual disk type: **SCSI (Recommended)**.
   - Click **Next**.

---

## Step 3: Provisioning OS Boot Disk & Mounting ISO

### 1. Configure the 20 GB OS Boot Disk:
- Select **Create a new virtual disk** $\to$ Click **Next**.
- Maximum disk size (GB): `20.0 GB`.
- Select **"Store virtual disk as a single file"**.
- *(Leave "Allocate all disk space now" unchecked for thin provisioning).*
- Disk file name: `UKM-TrueNAS-SCALE-os.vmdk` $\to$ Click **Next**.
- Click **Finish** to close the initial wizard.

> [!WARNING]
> **ZFS Boot Disk Rule**: TrueNAS SCALE requires a dedicated drive for its operating system (`boot-pool`). You **cannot** partition or use this drive to store user data or storage pools!

### 2. Attach TrueNAS SCALE ISO Image:
- In VMware Workstation, highlight `UKM-TrueNAS-SCALE` and click **Edit virtual machine settings**.
- In the Hardware tab, select **CD/DVD (SATA)**.
- Under Device status, check **Connect at power on**.
- Under Connection, select **"Use ISO image file"**.
- Click **Browse...** and navigate to your downloaded ISO:
  - **Linux**: `/home/<user>/Downloads/TrueNAS-SCALE.iso`
  - **Windows**: `C:\ISO\TrueNAS-SCALE.iso`
- Keep the settings window open for Step 4.

---

## Step 4: Adding 4x Lab Data Disks for ZFS Storage Pools

To complete the upcoming hands-on labs (Storage Pool creation, RAIDZ1/RAIDZ2, Snapshots, and Disk Replacement), attach **4 additional virtual SCSI disks**:

While still in **Virtual Machine Settings**:
1. Click **Add...** at the bottom of the Hardware tab.
2. Select **Hard Disk** $\to$ Click **Next**.
3. Select **SCSI** $\to$ Click **Next**.
4. Select **Create a new virtual disk** $\to$ Click **Next**.
5. Set Maximum disk size: `20.0 GB` $\to$ Select **Store virtual disk as a single file**.
6. Set Disk file name (e.g., `UKM-TrueNAS-SCALE-data1.vmdk`) $\to$ Click **Finish**.
7. **Repeat steps 1 to 6 three more times** to create a total of four 20 GB data disks:
   - `data1.vmdk` (20 GB) $\to$ SCSI 0:1
   - `data2.vmdk` (20 GB) $\to$ SCSI 0:2
   - `data3.vmdk` (20 GB) $\to$ SCSI 0:3
   - `data4.vmdk` (20 GB) $\to$ SCSI 0:4
8. Click **OK** to save the virtual machine settings.

### Summary of VM Hardware Inventory:
```text
Device                          Target Function
-----------------------------------------------------------------
Memory: 8 GB - 16 GB            ZFS ARC Cache & Operating System
Processors: 2 - 4 Cores         System & Storage Engine
Hard Disk (SCSI 0:0) - 20 GB    TrueNAS OS Boot Drive (sda)
Hard Disk (SCSI 0:1) - 20 GB    ZFS Pool Data Disk 1 (sdb)
Hard Disk (SCSI 0:2) - 20 GB    ZFS Pool Data Disk 2 (sdc)
Hard Disk (SCSI 0:3) - 20 GB    ZFS Pool Data Disk 3 (sdd)
Hard Disk (SCSI 0:4) - 20 GB    ZFS Pool Data Disk 4 (sde)
CD/DVD (SATA 0:0)               TrueNAS-SCALE installer ISO
Network Adapter                 Bridged (or NAT)
```

---

## Step 5: Booting the TrueNAS SCALE Installer

1. Power on the VM:
   - Click **Power on this virtual machine** (or press `Ctrl + B`).
2. VMware boots from the virtual CD/DVD into the GNU GRUB installer menu.
3. Select **Start TrueNAS SCALE Installation** and press `Enter`.
4. The TrueNAS Console Setup text-based menu appears:

```text
+------------------------ TrueNAS SCALE Console Setup -----------------------+
|                                                                            |
|                     1 Install/Upgrade                                      |
|                     2 Shell                                                |
|                     3 Reboot System                                        |
|                     4 Shut Down System                                     |
|                                                                            |
+----------------------------------------------------------------------------+
|                         [  OK  ]       [ Cancel ]                          |
+----------------------------------------------------------------------------+
```

- Highlight **1 Install/Upgrade** and press `Enter` (or press `Tab` to `[ OK ]` and `Enter`).

---

## Step 6: Target Disk Selection & Credentials

> [!CAUTION]
> **CRITICAL STEP**: The installer lists all 5 virtual disks (`sda`, `sdb`, `sdc`, `sdd`, `sde`). You must install TrueNAS **only** onto the 20 GB OS drive (`sda`). Do NOT select any other drive!

### 1. Select the Boot Drive:
- Use arrow keys to highlight `sda (20 GiB, VMware Virtual S...)`.
- Press the **`[Space]`** bar. An asterisk `[*]` will appear next to `sda`.
- Verify that `sdb`, `sdc`, `sdd`, and `sde` remain **UNCHECKED** `[ ]`.
- Press `Tab` to select `[ OK ]` and press `Enter`.
- Confirmation prompt appears: *"WARNING: This will erase all partitions and data on sda. Do you wish to proceed?"* $\to$ Select **Yes** and press `Enter`.

### 2. Set the Administrator Password:
- The installer asks for administrative authentication method.
- Choose: **1 Administrative user (admin)**.
- Enter a secure password for the `admin` account:
  ```text
  Ukmtruenas2026!
  ```
- Confirm the password and select `[ OK ]`.

> [!IMPORTANT]
> Keep note of this password! You will use `admin` and this password to access the TrueNAS WebUI, SSH, and local console throughout the entire training workshop.

---

## Step 7: Swap Creation & Installation Progress

1. **Swap Allocation**:
   - Prompt: *"Create 16GB swap partition on boot drive?"*
   - Select: **Create swap** *(Recommended)*.
   - Press `Enter`. (Swap helps prevent Out-Of-Memory kernel panics when ZFS memory cache expands during heavy I/O operations).

2. **Installation Execution**:
   - The installer partitions `sda`, creates the `boot-pool` ZFS dataset, extracts base system packages, and configures the bootloader.
   - This process takes approximately **2 to 4 minutes**.

3. **Installation Succeeded**:
   ```text
   +---------------------------------------------------+
   | TrueNAS SCALE installation on sda succeeded!      |
   |                                                   |
   | Please reboot and remove the installation media.  |
   |                                                   |
   |                      [  OK  ]                     |
   +---------------------------------------------------+
   ```
   - Press `Enter` on **[ OK ]**.
   - From the main console menu, select **3 Reboot System** and press `Enter`.

---

## Step 8: Post-Installation & First Boot

1. **Disconnect the Installation ISO**:
   - Before the VM restarts (or while it is rebooting), disconnect the ISO from VMware so it doesn't boot back into the installer:
   - In VMware top menu: Click **VM** $\to$ **Removable Devices** $\to$ **CD/DVD (SATA)** $\to$ **Disconnect**.
   - *(On Windows: Alternatively, right-click the CD/DVD icon in the bottom-right status bar of VMware and click "Disconnect").*

2. **Booting into TrueNAS SCALE**:
   - TrueNAS automatically loads via GRUB from virtual disk `sda`.
   - Linux systemd initializes kernel modules, network interfaces, and storage daemons (~60–90 seconds).

3. **TrueNAS Console Menu Display**:
   Once booted, the console displays the TrueNAS banner with its assigned IP address:
   ```text
   =====================================================
                    TrueNAS SCALE (24.x)
   =====================================================
   
   The web user interface is at:
   
   http://192.168.10.75
   https://192.168.10.75
   
   1) Configure Network Interfaces
   2) Configure Network Settings
   3) Configure Default Route
   4) Configure Static Routes
   5) Configure Link Aggregation
   6) Configure VLAN Interface
   7) Configure Network Bridge
   8) Reset Network Configuration
   9) Open Shell
   10) Reboot
   11) Shut Down
   ```
   - **Note down the IP address** displayed on your screen!

---

## Step 9: Verify WebUI Access from Host

### A. Testing from a Linux Host:

1. Open a terminal and test connectivity:
   ```bash
   # 1. Ping the TrueNAS VM
   ping -c 3 192.168.10.75

   # 2. Test HTTPS port response
   curl -k -I https://192.168.10.75
   ```

2. Open browser (**Firefox** or **Chrome**):
   - Navigate to: `https://192.168.10.75`
   - Bypass the self-signed certificate warning: Click **Advanced** $\to$ **Accept the Risk and Continue**.

---

### B. Testing from a Windows Host:

1. Open **PowerShell** or **Command Prompt**:
   ```powershell
   # 1. Ping the TrueNAS VM
   ping 192.168.10.75

   # 2. Test port 443 (HTTPS) connectivity via PowerShell
   Test-NetConnection -ComputerName 192.168.10.75 -Port 443

   # 3. Test HTTP/HTTPS response using curl.exe
   curl.exe -k -I https://192.168.10.75
   ```

2. Open browser (**Microsoft Edge**, **Google Chrome**, or **Mozilla Firefox**):
   - Navigate to: `https://192.168.10.75`
   - In **Microsoft Edge / Google Chrome**: When the *"Your connection isn't private"* alert appears, click **Advanced** $\to$ click **Continue to 192.168.10.75 (unsafe)**.

---

### Log In to the TrueNAS Dashboard:
1. On the login page:
   - **Username**: `admin`
   - **Password**: `Ukmtruenas2026!`
2. Verify Disks for Upcoming Labs:
   - In the left sidebar, click **Storage** $\to$ **Disks**.
   - Confirm you see **5 disks total**:
     - `sda` (20 GB) marked as Member of `boot-pool`.
     - `sdb`, `sdc`, `sdd`, `sde` (4x 20 GB) marked as **Unassigned**.

> [!TIP]
> Your lab environment is now fully primed! The 4 unassigned virtual disks will be used in **Module 2 (Lab 1)** to construct `warisan_pool` using RAIDZ1 / RAIDZ2 topologies.

---

## 🛠️ Windows Host: Enabling Native NFS Client (Bonus for Module 3)

During **Module 3: Access Control & Network Sharing (NFS)**, participants test accessing TrueNAS NFS exports directly from their host workstation. Windows includes a built-in NFS client that can be activated in seconds:

### Via PowerShell (Run as Administrator):
```powershell
Enable-WindowsOptionalFeature -Online -FeatureName ServicesForNFS-ClientOnly,ClientForNFS-Infrastructure -NoRestart
```

### Via Windows GUI:
1. Press `Win + R`, type `OptionalFeatures.exe`, and press `Enter`.
2. Scroll down to **Services for NFS**.
3. Expand it and check **Client for NFS** and **Administrative Tools**.
4. Click **OK** and allow Windows to install the feature.

Once enabled, you can mount TrueNAS NFS exports in Windows Command Prompt:
```cmd
mount -o anon \\192.168.10.75\mnt\warisan_pool\archives Z:
```

---

## 🔧 Host Troubleshooting Reference

### Linux Host Troubleshooting:

| Issue | Root Cause | Recommended Fix |
| :--- | :--- | :--- |
| **No IP shown in TrueNAS console** | Bridged adapter bound to inactive interface | Run `sudo vmware-netcfg` on Linux host. Ensure `VMnet0` is explicitly bridged to your active physical NIC (`eth0` or `wlan0`). |
| **VMware kernel module error** | Linux kernel was updated | Rebuild kernel drivers: `sudo vmware-modconfig --console --install-all` |
| **WebUI unreachable from host browser** | Linux host firewall blocking VM traffic | Allow incoming traffic: `sudo ufw allow in on vmnet+` or inspect routing with `ip route`. |
| **Mouse / keyboard trapped in console** | VMware window focus grab | Press `Ctrl + Alt` simultaneously to release the input cursor back to Linux desktop. |
| **Installer reloads upon reboot** | ISO remains connected | Top menu: **VM** $\to$ **Removable Devices** $\to$ **CD/DVD** $\to$ **Disconnect**, then reboot VM. |

---

### Windows Host Troubleshooting:

| Issue | Root Cause | Recommended Fix |
| :--- | :--- | :--- |
| **"VMware Workstation and Device/Credential Guard are not compatible"** | Windows 11 Core Isolation or Hyper-V conflict | 1. Open Windows Security $\to$ Device Security $\to$ **Core Isolation details** $\to$ Turn OFF **Memory Integrity**.<br>2. Or enable Windows Hypervisor Platform in PowerShell as Admin:<br>`Enable-WindowsOptionalFeature -Online -FeatureName HypervisorPlatform -NoRestart` |
| **TrueNAS receives no IP on Campus Wi-Fi** | Campus 802.1X / eduroam Wi-Fi blocks multiple MACs | Switch VM network adapter from **Bridged** to **NAT** in VM Settings. TrueNAS will obtain an IP from `VMnet8` and communicate directly with Windows host. |
| **Bridged mode bridges to wrong adapter** | Automatic bridging selects inactive VPN / Virtual adapter | Open Start Menu $\to$ search **Virtual Network Editor** (Run as Admin). Select **VMnet0** $\to$ Change "Bridged to: Automatic" to your specific physical Wi-Fi/Ethernet card (e.g. *Intel Wi-Fi 6 AX201*). |
| **Windows cannot ping TrueNAS VM** | Windows Defender Firewall blocking VMnet subnet | Open Windows Defender Firewall with Advanced Security $\to$ Verify that inbound ICMP traffic is permitted on Private/Domain networks, or verify VM is on `VMnet8`. |
| **VMware services fail to start** | Windows background service disabled | Open PowerShell as Admin and run:<br>`Start-Service VMAuthdService`<br>`Start-Service "VMware NAT Service"`<br>`Start-Service "VMnetDHCP"` |
| **Keyboard cursor trapped in VM** | VM console focus | Press `Ctrl + Alt` to release mouse and keyboard focus back to Windows. |