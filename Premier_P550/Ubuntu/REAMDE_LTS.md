---
sys: ubuntu
sys_ver: 24.04.3
sys_var: LTS
status: good
last_update: 2026-07-07
---

# Ubuntu 24.04.3 LTS HiFive Premier P550 Test Report

## Test Environment

### Hardware Information

- Development Board: HiFive Premier P550
- Other Hardware:
  - A MicroSD card
  - A USB Type A to C cable

### Operating System Information

- OS Version: Ubuntu 2025.11.00 (Based on Ubuntu 24.04.3 LTS) (SiFive official support)
- Download Links: <https://github.com/sifiveinc/hifive-premier-p550-ubuntu/releases/tag/2025.11.00>
- Reference Installation Document: <https://www.sifive.com/document-file/hifive-premier-p550-image-update-procedure>
- Reference Softwave Document: <https://www.sifive.com/document-file/hifive-premier-p550-software-reference-manual>

## Installation Steps

1. Download and extract the OS image and bootloader files.

    ```bash
    wget https://github.com/sifiveinc/hifive-premier-p550-ubuntu/releases/download/2025.11.00/ubuntu-24.04-preinstalled-server-riscv64.img.xz

    wget https://github.com/sifiveinc/freedom-u-sdk/releases/download/2026.07.00-HFP550/bootloader_ddr5_secboot.bin

    unxz -d ubuntu-24.04-preinstalled-server-riscv64.img.xz
    ```

2. Mount the MicroSD card and use the `cp` command to copy the image files to the MicroSD card. (Assuming `/dev/sdc` is the MicroSD card device)

    ```bash
    #Linux
    sudo mount /dev/sdc /mnt/sd
    
    sudo cp ./ubuntu-24.04-preinstalled-server-riscv64.img ./bootloader_ddr5_secboot.bin /mnt/sd

    sync

    sudo umount /mnt/sd
    
    #macOS(Assuming `/dev/disk4` is the MicroSD card device)：
	diskutil list external physical
	brew install e2fsprogs e2tools
	diskutil unmountDisk /dev/disk4
	sudo $(brew --prefix e2fsprogs)/sbin/mkfs.ext4 -F -L SDCARD /dev/disk4
	ls -lh ubuntu-24.04-preinstalled-server-riscv64.img bootloader_ddr5_secboot.bin
	sudo e2cp ubuntu-24.04-preinstalled-server-riscv64.img /dev/disk4:/
	sudo e2cp bootloader_ddr5_secboot.bin /dev/disk4:/
	sudo e2ls -l /dev/disk4:/
	sync
	diskutil eject /dev/disk4
    ```

3. Insert the MicroSD card, connect the serial port on the development board, boot the development board, and enter the U-Boot in the SPI Flash.
    - Ensure that the DIP switch is the SPI Flash startup mode: `DIP_SW1[3:0] = 0100`. (SW's ON = 0, OFF = 1)
    - Quickly press Enter when `Hit any key to stop autoboot` appears on the serial terminal to enter the U-Boot command line interface.

4. Burn the image using the custom U-Boot command.
    - Check that the images on the MMC card are readable.

        ```bash
        => ls mmc 1
        ```

    - Flash the new bootloader if necessary.

        ```bash
        => ext4load mmc 1 0x90000000 bootloader_ddr5_secboot.bin

        => es_burn write 0x90000000 flash
        ```

    - Flash the os image.

        ```bash
        => es_fs write mmc 1 ubuntu-24.04-preinstalled-server-riscv64.img mmc 0
        ```

5. Power cycle the board to boot with the new images.

## Expected Results

The system boots up normally and allows login through the onboard serial port.

## Actual Results

The system boots up normally and login through the onboard serial port is successful.

### Boot Information

```log
[  OK  ] Finished cloud-final.service - Cloud-init: Final Stage.
[  OK  ] Reached target cloud-init.target - Cloud-init target.

Ubuntu 24.04.3 LTS ubuntu ttyS0

ubuntu login: ubuntu
Password:

Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.12.33-1-premier riscv64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

System information as of Thu Nov 27 14:15:34 UTC 2025

  System load:     1.02
  Usage of /home:  unknown
  Memory usage:    49%
  Swap usage:      32%
  Temperature:     61.0 C
  Processes:       433
  Users logged in: 0

=> There is 1 zombie process.
=> There were exceptions while processing one or more plugins.
   See /var/log/landscape/sysinfo.log for more information.

The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts.
Check your Internet connection or proxy settings.

ubuntu@ubuntu:~$ uname -a
Linux ubuntu 6.12.33-1-premier #6 SMP PREEMPT_DYNAMIC Thu Nov 27 07:15:04 UTC 2025 riscv64 riscv64 riscv64 GNU/Linux

ubuntu@ubuntu:~$ cat /etc/os-release
PRETTY_NAME="Ubuntu 24.04.3 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.3 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo

ubuntu@ubuntu:~$

```

Screen recording:

[![asciicast](https://asciinema.org/a/SjboeKgUBPB3uFaF.svg)](https://asciinema.org/a/SjboeKgUBPB3uFaF)
## Test Criteria

Successful: The actual result matches the expected result.

Failed: The actual result does not match the expected result.

## Test Conclusion

Test successful.
