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
