下面给出从零开始在 Debian GNOME 上安装并用好 virt-manager（KVM/QEMU）的简明入门步骤，按顺序执行并把报错贴出来我再帮你排查。

重要准备（先做这几步）

1. 打开终端（GNOME：按 Super 键，输入“终端”）。
2. 更新包列表：

```
sudo apt update
sudo apt upgrade -y
```

安装必要软件

```
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager virt-viewer
```

（如果你之前装过 virt-manager，建议先 apt update 再重装以上包。）

把当前用户加入组（避免每次用 sudo）

```
sudo adduser $USER libvirt
sudo adduser $USER kvm
```

完成后登出并重新登录（或重启），使组变更生效。

检查服务与内核模块

```
systemctl status libvirtd
lsmod | grep kvm
```

- 正常情况：**libvirtd 服务 active (running)**；lsmod 能看到 kvm 或 kvm_intel/kvm_amd。
- 如果 libvirtd 未启动：`sudo systemctl enable --now libvirtd`

启动 virt-manager（图形）

- 从应用菜单打开 “Virtual Machine Manager”（或终端运行 `virt-manager`）。
- 连接到本地：左上角会显示 “QEMU/KVM” 或 qemu:///system，应该显示已连接。

创建第一个虚拟机（常用流程）

1. 在 virt-manager 点击 “New VM”。
2. 选择 “Local install media (ISO image or CDROM)” → Next。
3. 点击 Browse，选择你下载好的 ISO（例如 Debian/Ubuntu ISO）。如果没自动识别 OS，选择相应类型与版本。
4. 分配 CPU、内存、磁盘大小（默认即可：比如 2 CPU、2048 MB、20 GB）。
5. 网络保持默认 NAT（通常叫 “default” 网络）→ Finish 启动安装器。
6. 在虚拟机窗口里按正常安装流程安装操作系统。

常见问题与快速排查

- 无法安装/无法连接 libvirt：
  - 确认服务在运行：`sudo systemctl status libvirtd`
  - 权限问题：是否已重登以应用组变更？
- 报错 “network 'default' is not active”：
  ```
  sudo virsh net-list --all
  sudo virsh net-start default
  sudo virsh net-autostart default
  ```
- 找不到 kvm/没有硬件虚拟化支持：
  - 查看 CPU 是否支持：`egrep -c '(vmx|svm)' /proc/cpuinfo`（输出 >0 表示支持）
  - BIOS/UEFI 里开启 Intel VT-x 或 AMD-V
- 虚拟机窗口黑屏或无法显示：
  - 安装 spice/vnc 支持：`sudo apt install virt-viewer gir1.2-spiceclientgtk-3.0`
  - 在创建时选对显示类型（VNC 或 Spice）
- 安装失败、ISO 无法识别：确认 ISO 完整（sha256sum）且选对架构（amd64 vs i386）

如果还有问题，请把下面信息贴过来（复制粘贴即可）：

1. `lsb_release -a`（Debian 版本）
2. `sudo systemctl status libvirtd` 的输出
3. `lsmod | grep kvm`
4. `virsh net-list --all`
5. 你创建虚拟机时报的完整错误信息（如果有截图可描述文字）

我看到具体输出后帮你定位并给出具体命令和修复步骤。
