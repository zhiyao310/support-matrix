---
sys: ubuntu
sys_ver: 24.04.4
sys_var: LTS
status: good
last_update: 2026-07-07
---

# Ubuntu 24.04.4 LTS ESWIN 7702 D560 Test Report

## Test Environment

### Hardware Information

- Development Board: ESWIN 7702 D560
- Other Hardware:
  - A MicroSD card
  - A USB Type A to C cable

### Operating System Information

- OS Version: Ubuntu 2025.12.30 (Based on Ubuntu 24.04.4 LTS)
- Download Links: <https://github.com/eswincomputing/ebc7702-dev-board-ubuntu/releases/tag/2025.12.30>
- Reference Installation Document: <https://github.com/eswincomputing/ebc7702-dev-board-ubuntu/blob/main/README.md>

## Installation Steps

### Download and extract the OS image and bootloader files.

```bash
wget https://github.com/eswincomputing/ebc7702-dev-board-ubuntu/releases/download/2025.12.30/bootloader_EBC7702-D01_die0.bin
wget https://github.com/eswincomputing/ebc7702-dev-board-ubuntu/releases/download/2025.12.30/bootloader_EBC7702-D01_die1.bin
wget https://github.com/eswincomputing/ebc7702-dev-board-ubuntu/releases/download/2025.12.30/d560-24.04-preinstalled-server-riscv64_20260313_1628_70.img.zst

zstd -d d560-24.04-preinstalled-server-riscv64_20260313_1628_70.img.zst
```

### Prepare the MicroSD card on Linux

Mount the MicroSD card and use the `cp` command to copy the image files to the MicroSD card. (Assuming `/dev/sdc` is the MicroSD card device)

```bash
# 1. Check the MicroSD card device name
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT,MODEL

# Assume the MicroSD card is identified as /dev/sdc
# Note: The following operations will erase all data on /dev/sdc

# 2. Unmount any mounted partitions
sudo umount /dev/sdc* 2>/dev/null || true

# 3. Format the entire MicroSD card as ext4
sudo mkfs.ext4 -F -L SDCARD /dev/sdc

# 4. Create a mount directory
sudo mkdir -p /mnt/sd

# 5. Mount the MicroSD card
sudo mount /dev/sdc /mnt/sd

# 6. Copy the Ubuntu image and bootloader files
sudo cp ./d560-24.04-preinstalled-server-riscv64_20260313_1628_70.img /mnt/sd/
sudo cp ./bootloader_EBC7702-D01_die0.bin /mnt/sd/
sudo cp ./bootloader_EBC7702-D01_die1.bin /mnt/sd/

# 7. Confirm that the files have been copied
ls -lh /mnt/sd/

# 8. Sync writes and unmount
sync
sudo umount /mnt/sd
```


### Prepare the MicroSD card on macOS

```bash
# 1. Check the MicroSD card device name
diskutil list external physical

# Assume the MicroSD card is identified as /dev/disk4
# Note: The following operations will erase all data on /dev/disk4

# 2. Install ext4-related tools
brew install e2fsprogs e2tools

# 3. Unmount any mounted partitions
diskutil unmountDisk /dev/disk4

# 4. Format the entire MicroSD card as ext4
sudo $(brew --prefix e2fsprogs)/sbin/mkfs.ext4 -F -L SDCARD /dev/disk4

# 5. Copy the Ubuntu image and bootloader files to the MicroSD card root directory
sudo e2cp ./d560-24.04-preinstalled-server-riscv64_20260313_1628_70.img /dev/disk4:/
sudo e2cp ./bootloader_EBC7702-D01_die0.bin /dev/disk4:/
sudo e2cp ./bootloader_EBC7702-D01_die1.bin /dev/disk4:/

# 6. Confirm that the files have been copied
sudo e2ls -l /dev/disk4:/

# 7. Sync writes and eject
sync
diskutil eject /dev/disk4
```


### Enter the U-Boot command line

Connect the serial console, power on the board, and press Enter when `Hit any key to stop autoboot` appears to enter the U-Boot command line.

### Burn the image using the custom U-Boot command.

- Check that the images on the MMC card are readable.

```bash
=> ls mmc 1
```

- Flash the new bootloader if necessary.
- Note: EBC7702 is a dual-die board. The die0 and die1 bootloaders must not be mixed.

```bash
ext4load mmc 1 0x100000000 bootloader_EBC7702-D01_die0.bin
es_burn write 0x100000000 flash

ext4load mmc 1 0x100000000 bootloader_EBC7702-D01_die1.bin
es_burn write 0x100000000 flash 1
```

- Flash the OS image.

```bash
es_fs write mmc 1 d560-24.04-preinstalled-server-riscv64_20260313_1628_70.img mmc 0
```

- Power cycle the board to boot with the new images.

```bash
=> reset
```

## Expected Results

The system boots up normally and allows login through the onboard serial port.

## Actual Results

The system boots up normally and login through the onboard serial port is successful.

### Boot Information

```log
[  OK  ] Reached target cloud-init.target - Cloud-init target.

ubuntu login: ubuntu
Password:

You are required to change your password immediately (administrator enforced).
Changing password for ubuntu.
Current password:
New password:
Retype new password:

Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.6.115-2025-eic7702 riscv64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

System information as of Fri Mar 13 08:20:54 UTC 2026

  System load:    1.29
  Usage of /home: unknown
  Memory usage:   13%
  Swap usage:     0%
  Temperature:    61.0 C
  Processes:      27
  Users logged in: 0

=> There were exceptions while processing one or more plugins.
   See /var/log/landscape/sysinfo.log for more information.

The list of available updates is more than a week old.
To check for new updates run: sudo apt update

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

ubuntu@ubuntu:~$ uname -a
Linux ubuntu 6.6.115-2025-eic7702 #12.003 SMP Mon Mar 9 16:18:46 UTC 2026 riscv64 riscv64 riscv64 GNU/Linux

ubuntu@ubuntu:~$ cat /etc/os-release
PRETTY_NAME="Ubuntu 24.04.4 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.4 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo

```

Screen recording:

[![asciicast](https://asciinema.org/a/HvLhRZ91SYEV5pyE.svg)](https://asciinema.org/a/HvLhRZ91SYEV5pyE)

## Test Criteria

Successful: The actual result matches the expected result.

Failed: The actual result does not match the expected result.

## Test Conclusion

Test successful.
