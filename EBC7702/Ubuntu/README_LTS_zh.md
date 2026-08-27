# Ubuntu 24.04.4 LTS ESWIN 7702 D560 测试报告

## 测试环境

### 硬件信息

- 开发板: ESWIN 7702 D560
- 其他硬件:
  - MicroSD 卡一张
  - USB Type C to A 线缆一条

### 操作系统信息

- 操作系统版本：Ubuntu 2025.12.30 (基于Ubuntu 24.04.4 LTS)
- 下载链接: <https://github.com/eswincomputing/ebc7702-dev-board-ubuntu/releases/tag/2025.12.30>
- 参考安装文档：<https://github.com/eswincomputing/ebc7702-dev-board-ubuntu/blob/main/README.md>

## 安装步骤

### 下载操作系统镜像文件并解压

```bash
wget https://github.com/eswincomputing/ebc7702-dev-board-ubuntu/releases/download/2025.12.30/bootloader_EBC7702-D01_die0.bin
wget https://github.com/eswincomputing/ebc7702-dev-board-ubuntu/releases/download/2025.12.30/bootloader_EBC7702-D01_die1.bin
wget https://github.com/eswincomputing/ebc7702-dev-board-ubuntu/releases/download/2025.12.30/d560-24.04-preinstalled-server-riscv64_20260313_1628_70.img.zst

zstd -d d560-24.04-preinstalled-server-riscv64_20260313_1628_70.img.zst
```

### 拷贝镜像文件到 MicroSD 卡（Linux）

挂载 MicroSD 卡，并使用 `cp` 命令将镜像文件拷贝到卡中。以下示例假设 `/dev/sdc` 为 MicroSD 卡设备。

```bash
# 1. 查看 MicroSD 卡设备名
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT,MODEL

# 假设确认 MicroSD 卡是 /dev/sdc
# 注意：以下操作会清空 /dev/sdc 上的所有内容

# 2. 卸载已有挂载分区
sudo umount /dev/sdc* 2>/dev/null || true

# 3. 将整张 MicroSD 卡格式化为 ext4
sudo mkfs.ext4 -F -L SDCARD /dev/sdc

# 4. 创建挂载目录
sudo mkdir -p /mnt/sd

# 5. 挂载 MicroSD 卡
sudo mount /dev/sdc /mnt/sd

# 6. 复制 Ubuntu 镜像和 bootloader 文件
sudo cp ./d560-24.04-preinstalled-server-riscv64_20260313_1628_70.img /mnt/sd/
sudo cp ./bootloader_EBC7702-D01_die0.bin /mnt/sd/
sudo cp ./bootloader_EBC7702-D01_die1.bin /mnt/sd/

# 7. 确认文件已复制
ls -lh /mnt/sd/

# 8. 同步写入并卸载
sync
sudo umount /mnt/sd
```


### 拷贝镜像文件到 MicroSD 卡（macOS）

```bash
# 1. 查看 MicroSD 卡设备名
diskutil list external physical

# 假设确认 MicroSD 卡是 /dev/disk4
# 注意：以下操作会清空 /dev/disk4 上的所有内容

# 2. 安装 ext4 相关工具
brew install e2fsprogs e2tools

# 3. 卸载已有挂载分区
diskutil unmountDisk /dev/disk4

# 4. 将整张 MicroSD 卡格式化为 ext4
sudo $(brew --prefix e2fsprogs)/sbin/mkfs.ext4 -F -L SDCARD /dev/disk4

# 5. 复制 Ubuntu 镜像和 bootloader 文件到 MicroSD 卡根目录
sudo e2cp ./d560-24.04-preinstalled-server-riscv64_20260313_1628_70.img /dev/disk4:/
sudo e2cp ./bootloader_EBC7702-D01_die0.bin /dev/disk4:/
sudo e2cp ./bootloader_EBC7702-D01_die1.bin /dev/disk4:/

# 6. 确认文件已复制
sudo e2ls -l /dev/disk4:/

# 7. 同步写入并弹出
sync
diskutil eject /dev/disk4
```


### 进入 U-Boot 命令行

连接开发板串口并上电，在串口终端出现 `Hit any key to stop autoboot` 时按下回车，进入 U-Boot 命令行。

### 使用定制的U-Boot命令烧录镜像。

- 确认MicroSD卡状态。

```bash
=> ls mmc 1
```

- 如果需要的话，烧录新的 bootloader。
- 注意：EBC7702 是双 die 板，die0 / die1 bootloader 不能混用。

```bash
ext4load mmc 1 0x100000000 bootloader_EBC7702-D01_die0.bin
es_burn write 0x100000000 flash

ext4load mmc 1 0x100000000 bootloader_EBC7702-D01_die1.bin
es_burn write 0x100000000 flash 1
```

- 烧录系统镜像。

```bash
es_fs write mmc 1 d560-24.04-preinstalled-server-riscv64_20260313_1628_70.img mmc 0
```

- 重启开发板。

```bash
=> reset
```


## 预期结果

系统正常启动，成功通过串口登录。

## 实际结果

系统正常启动，成功通过串口登录。

### 启动信息

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

屏幕录像（从刷写镜像到登录系统）：

[![asciicast](https://asciinema.org/a/HvLhRZ91SYEV5pyE.svg)](https://asciinema.org/a/HvLhRZ91SYEV5pyE)

## 测试判定标准

测试成功：实际结果与预期结果相符。

测试失败：实际结果与预期结果不符。

## 测试结论

测试成功。
