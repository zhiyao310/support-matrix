# Ubuntu 24.04.3 LTS HiFive Premier P550 测试报告

## 测试环境

### 硬件信息

- 开发板: HiFive Premier P550
- 其他硬件:
  - MicroSD 卡一张
  - USB Type C to A 线缆一条

### 操作系统信息

- 操作系统版本：Ubuntu 2025.11.00 (基于Ubuntu 24.04.3 LTS) (SiFive 官方支持)
- 下载链接: <https://github.com/sifiveinc/hifive-premier-p550-ubuntu/releases/tag/2025.11.00>
- 参考安装文档：<https://www.sifive.com/document-file/hifive-premier-p550-image-update-procedure>
- 软件参考文档：<https://www.sifive.com/document-file/hifive-premier-p550-software-reference-manual>

## 安装步骤

1. 下载操作系统镜像文件并解压。

    ```bash
    wget https://github.com/sifiveinc/hifive-premier-p550-ubuntu/releases/download/2025.11.00/ubuntu-24.04-preinstalled-server-riscv64.img.xz

    wget https://github.com/sifiveinc/freedom-u-sdk/releases/download/2026.07.00-HFP550/bootloader_ddr5_secboot.bin

    unxz -d ubuntu-24.04-preinstalled-server-riscv64.img.xz
    ```

2. 挂载MicroSD卡，并使用 `cp` 命令拷贝镜像文件至MicroSD卡。(假设 `/dev/sdc` 为MicroSD卡设备)

    ```bash
    #Linux可执行版本
    sudo mount /dev/sdc /mnt/sd
    
    sudo cp ./ubuntu-24.04-preinstalled-server-riscv64.img ./bootloader_ddr5_secboot.bin /mnt/sd

    sync

    sudo umount /mnt/sd
    
    #macOS可执行版本(假设 `/dev/disk4` 为MicroSD卡设备)：
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

3. 插入MicroSD卡，连接开发板串口，启动开发板并进入SPI Flash中的U-Boot。
    - 确保拨码开关为SPI Flash的启动模式：`DIP_SW1[3:0] = 0100`。(SW的ON = 0, OFF = 1)
    - 在串口终端中出现 `Hit any key to stop autoboot` 时迅速按下回车键，进入 U-boot 命令行终端。

4. 使用定制的U-Boot命令烧录镜像。

    - 确认MicroSD卡状态。

        ```bash
        => ls mmc 1
        ```

    - 如果需要的话，烧录新的bootloadr。

        ```bash
        => ext4load mmc 1 0x90000000 bootloader_ddr5_secboot.bin

        => es_burn write 0x90000000 flash
        ```

    - 烧录系统镜像。

        ```bash
        => es_fs write mmc 1 ubuntu-24.04-preinstalled-server-riscv64.img mmc 0
        ```

5. 重启开发板。

## 预期结果

系统正常启动，成功通过串口登录。

## 实际结果

系统正常启动，成功通过串口登录。

### 启动信息

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

屏幕录像（从刷写镜像到登录系统）：

[![asciicast](https://asciinema.org/a/SjboeKgUBPB3uFaF.svg)](https://asciinema.org/a/SjboeKgUBPB3uFaF)
## 测试判定标准

测试成功：实际结果与预期结果相符。

测试失败：实际结果与预期结果不符。

## 测试结论

测试成功。