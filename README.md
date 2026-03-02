# Project: Three-Tier WordPress Solution with LVM

## Phase 1: Storage Infrastructure Setup (Web Server)

In this phase, we configure the storage on the **Web Server** using LVM to ensure flexibility for future scaling.

### 1.1 Provisioning and Attaching EBS Volumes

1. Launch a **RedHat EC2** instance to serve as the Web Server.
2. Create three **10 GiB** EBS volumes in the same Availability Zone (AZ).
3. Attach all three volumes to your Web Server instance.