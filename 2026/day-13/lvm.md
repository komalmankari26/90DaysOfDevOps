# Day 13 – Linux Volume Management (LVM)

## Objective

The objective of this task was to learn Linux Logical Volume Management (LVM) by creating a Physical Volume (PV), Volume Group (VG), Logical Volume (LV), formatting it with a filesystem, and mounting it for use.

---

## Environment

- AWS EC2 Ubuntu Instance
- Connected using Git Bash
- LVM Tools (`lvm2`)

---

## Commands Used

### 1. Switch to Root User

```bash
sudo -i
```

### 2. Install LVM

```bash
apt update
apt install lvm2 -y
```

### 3. Create a Virtual Disk

> Note: Due to limited storage on the EC2 instance, a **300 MB** virtual disk was created instead of 1 GB.

```bash
dd if=/dev/zero of=/tmp/disk1.img bs=1M count=300
```

### 4. Attach the Virtual Disk

```bash
losetup -fP /tmp/disk1.img
losetup -a
```

The virtual disk was attached as:

```text
/dev/loop5
```

### 5. Check Existing Storage

```bash
lsblk
pvs
vgs
lvs
df -h
```

### 6. Create Physical Volume

```bash
pvcreate /dev/loop5
```

### 7. Create Volume Group

```bash
vgcreate devops-vg /dev/loop5
```

### 8. Create Logical Volume

```bash
lvcreate -L 200M -n app-data devops-vg
```

### 9. Format the Logical Volume

```bash
mkfs.ext4 /dev/devops-vg/app-data
```

### 10. Create Mount Point

```bash
mkdir -p /mnt/app-data
```

### 11. Mount the Logical Volume

```bash
mount /dev/devops-vg/app-data /mnt/app-data
```

### 12. Verify the Mount

```bash
df -h
lsblk
lvs
vgs
pvs
```

---

## Screenshots

Include screenshots of:

- `lsblk`
- `losetup -a`
- `pvs`
- `vgs`
- `lvs`
- `df -h`
- `mount`
- `mkfs.ext4`

---

## Challenges Faced

- The EC2 root volume had limited free space.
- Creating a 1 GB virtual disk resulted in the error:
  ```
  dd: error writing '/tmp/disk1.img': No space left on device
  ```
- A 300 MB virtual disk was created successfully instead.
- The virtual disk was attached as `/dev/loop5` because `/dev/loop0` to `/dev/loop4` were already in use by Snap packages.

---

## What I Learned

1. LVM provides flexible storage management by allowing logical volumes to be created independently of physical disks.
2. A Physical Volume (PV) is combined into a Volume Group (VG), from which Logical Volumes (LVs) are created.
3. Loop devices can be used to simulate physical disks for learning and testing LVM without adding a real disk.

---

## Conclusion

This exercise helped me understand the complete LVM workflow, including creating a virtual disk, configuring Physical Volumes, Volume Groups, and Logical Volumes, formatting the volume, and mounting it on an AWS EC2 Ubuntu instance.
<img width="1920" height="1080" alt="Screenshot (293)" src="https://github.com/user-attachments/assets/437e5a44-0999-432f-bc3e-0a942a0bf1c5" />
<img width="1920" height="1080" alt="Screenshot (295)" src="https://github.com/user-attachments/assets/aec66af8-e1fd-4bdb-8fdb-c3b0ed508f9e" />
