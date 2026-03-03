This is a detailed, professional documentation of your **Phase 1: Web-Server Deployment**. It is written to reflect the exact journey completed, including the specific terminal commands and the logic used to overcome RHEL 10's strict security.

---

# **Project Documentation: Three-Tier WordPress Solution**

## **Phase 1: Web-Server Configuration (Presentation & Application Tier)**

### **1.1 Infrastructure & Storage Provisioning**

The foundation of the Web-Server requires dedicated storage and specific security rules to allow public web traffic.

* **AMI**: Red Hat Enterprise Linux (RHEL) 10.
* **Instance Type**: `t3.micro`.
* **Storage**: 10 GiB (Root, `nvme0n1`) + **2 x 10 GiB EBS Volumes** (`nvme1n1`, `nvme2n1`) attached as `gp3`.
* **Security Group Rules**:
  * **SSH (22)**: Access from `My IP`.
  * **HTTP (80)**: Access from `0.0.0.0/0`.

> **[RESERVE: Screenshot of AWS Console showing the Web-Server instance with the 2 extra EBS volumes attached]**

---

### **1.2 Storage Subsystem (LVM) Setup**

We implemented Logical Volume Management (LVM) to manage application data and logs across the two additional 10 GiB disks.

**Discovery — verifying attached disks:**
Before proceeding, `lsblk` was run to confirm which block devices were actually available:

```bash
lsblk
```

Output confirmed two extra disks (`nvme1n1`, `nvme2n1`) were attached. A third volume (`nvme3n1`) was not present, so all LVM work was performed using the two available disks.

![lsblk output and failed nvme3n1 discovery](screenshoots/2.png)

---

**Step 1 — Disk Partitioning:**
Each disk was partitioned interactively with `fdisk`. A GPT label was created (`g`), a new partition spanning the full disk was added (`n`, accepting defaults), and the partition type was set to **Linux LVM** (`t` → `44`).

```bash
sudo fdisk /dev/nvme1n1
# g → n → (defaults) → t → 44 → w

sudo fdisk /dev/nvme2n1
# g → n → (defaults) → t → 44 → w
```

![fdisk partitioning on nvme1n1 and nvme2n1](screenshoots/1.png)

---

**Step 2 — LVM Stack Creation:**

```bash
# Initialize Physical Volumes
sudo pvcreate /dev/nvme1n1p1 /dev/nvme2n1p1

# Create the Volume Group (20 GiB total)
sudo vgcreate webdata-vg /dev/nvme1n1p1 /dev/nvme2n1p1

# Create Logical Volumes
sudo lvcreate -n apps-lv -L 14G webdata-vg       # 14 GiB for the web application
sudo lvcreate -n logs-lv -l 100%FREE webdata-vg   # ~6 GiB for system logs
```

Verified with:
```bash
sudo lvs
```
```
LV      VG         Attr       LSize
apps-lv webdata-vg -wi-a----- 14.00g
logs-lv webdata-vg -wi-a-----  5.99g
```

![vgcreate, lvcreate, lvs output, mkfs.ext4, and rsync log backup](screenshoots/3.png)

---

**Step 3 — Filesystem & Mounting:**

```bash
# Format both volumes as ext4
sudo mkfs.ext4 /dev/webdata-vg/apps-lv
sudo mkfs.ext4 /dev/webdata-vg/logs-lv

# Create mount points and recovery directory
sudo mkdir -p /var/www/html
sudo mkdir -p /home/recovery/logs

# Preserve existing logs before replacing /var/log
sudo rsync -av /var/log/ /home/recovery/logs/

# Mount the new volumes
sudo mount /dev/webdata-vg/apps-lv /var/www/html/
sudo mount /dev/webdata-vg/logs-lv /var/log

# Restore logs onto the new volume
sudo rsync -av /home/recovery/logs/ /var/log/
```

---

**Step 4 — Persistence via `/etc/fstab`:**

UUIDs were retrieved and written to `/etc/fstab` to ensure mounts survive reboots:

```bash
sudo blkid
```

Key UUIDs captured:
* `apps-lv`: `973897b4-68b2-41a2-851b-0e3fe4ccd4bb`
* `logs-lv`: `c3b73404-fb37-42e0-867b-e30c49f0892f`

```bash
sudo vi /etc/fstab   # Added UUID entries for both LVs

sudo mount -a               # Verify no fstab errors
sudo systemctl daemon-reload
```

![blkid output, /etc/fstab entries, and mount -a verification](screenshoots/4.png)

---
