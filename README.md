# Project: Three-Tier WordPress Solution with LVM

## Phase 1: Storage Infrastructure Setup (Web Server)

In this phase, we configure the storage on the **Web Server** using LVM to ensure flexibility for future scaling.

### 1.1 Provisioning and Attaching EBS Volumes

1. Launch a **RedHat EC2** instance to serve as the Web Server.
2. Create three **10 GiB** EBS volumes in the same Availability Zone (AZ).
3. Attach all three volumes to your Web Server instance.

### 1.2 Disk Partitioning with `gdisk`

1. Use `lsblk` to inspect the newly attached block devices (typically `xvdf`, `xvdg`, `xvdh`).
2. Run `sudo gdisk /dev/xvdf` to create a new partition. Use hex code `8E00` for Linux LVM.
3. Repeat this for the other two disks (`xvdg` and `xvdh`).

> **Expected Output:** `lsblk` should show a partition (e.g., `xvdf1`) under each disk.

### 1.3 Configuring the LVM Stack

1. **Install LVM2**: `sudo yum install lvm2 -y`.
2. **Create Physical Volumes (PV)**: Mark the partitions for LVM use:
`sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`.
3. **Create Volume Group (VG)**: Add all 3 PVs to a group named `webdata-vg`:
`sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`.
4. **Create Logical Volumes (LV)**:
* `apps-lv` (14G) for website data: `sudo lvcreate -n apps-lv -L 14G webdata-vg`.
* `logs-lv` (14G) for log storage: `sudo lvcreate -n logs-lv -L 14G webdata-vg`.

> **Expected Output:** `sudo lvs` should display both logical volumes with their assigned sizes.

### 1.4 Formatting and Mounting

1. **Format LVs**: Use `mkfs.ext4` for both `apps-lv` and `logs-lv`.
2. **Mount Web Root**: Mount `apps-lv` to `/var/www/html`:
`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`.
3. **Mount Logs**: Backup `/var/log` using `rsync`, then mount `logs-lv` to `/var/log`.
4. **Persist Mounts**: Update `/etc/fstab` using the UUIDs obtained from `sudo blkid`.

---