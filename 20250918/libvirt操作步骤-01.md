- **libvirt 本身是一个后台守护进程与一套 API/工具集，不是单一 GUI。** 它既提供终端工具（virsh、virt-install、virsh net-* 等）也有图形客户端（常用的是 virt-manager）。

---

### 如何用图形界面（virt-manager）添加镜像与配置参数

1. 启动 virt-manager（桌面环境）：在终端运行 `virt-manager` 或从应用菜单打开。
2. 新建虚拟机：点击 “新建虚拟机”（New）→ 选择“本地安装介质（ISO）”或“已有磁盘镜像”。
3. 指定 ISO/镜像：在“安装介质”步骤，选择本地 ISO 文件或浏览已有的磁盘映像（qcow2、raw、vdi 等）。也可在 Storage Pools 中先导入镜像。
4. 分配资源：设置 CPU、内存、磁盘大小与类型（建议使用 virtio 驱动），选择引导顺序（UEFI/BIOS）。
5. 网络配置：选择网络接口类型——“默认（NAT）”通常是 virbr0；如需桥接到物理网，选用你创建的 bridge（如 br0）。还可以添加第二块网卡用于隔离内网。
6. 磁盘与存储：可选择新建磁盘或使用已有 qcow2。qcow2 支持快照，raw 性能更好。
7. 高级选项：在“Overview”或“Show virtual hardware details”内可设置固件类型、CPU 模型、NUMA、显卡、输入设备与设备模型（virtio、e1000 等）。
8. 完成安装并打开控制台进行操作系统安装。创建后可在虚拟机详情页点击 “Add Hardware” 添加网卡、磁盘、USB、NIC 型号、光驱等。

---

### 如何用终端（virsh / virt-install / qemu-img）添加镜像与配置

1. 准备磁盘镜像：用 qemu-img 创建或转换：

```
qemu-img create -f qcow2 /var/lib/libvirt/images/debian_lab.qcow2 40G
```

2. 使用 virt-install 从 ISO 安装（示例）：

```
virt-install \
--name debian-lab \
--ram 4096 --vcpus 2 \
--disk path=/var/lib/libvirt/images/debian_lab.qcow2,format=qcow2 \
--cdrom /path/to/debian.iso \
--os-variant debian12 \
--network network=default,model=virtio \
--graphics spice \
--boot useserial=on
```

3. 使用 virsh 管理与修改（示例）：列出/启动/编辑

```
virsh list --all
virsh start debian-lab
virsh shutdown debian-lab
virsh edit debian-lab   # 编辑 XML 配置（要小心）
```

在 virsh edit 打开后，你可以修改 <memory>、<vcpu>、<disk>、<interface> 等 XML 节点来调整参数。修改后保存，libvirt 会应用支持的更改或提示重启。

---

### 存储池与网络（常用命令）

- 列出/定义存储池：

```
virsh pool-list --all
virsh pool-create-as --name default --type dir --target /var/lib/libvirt/images
```

- 导入已有磁盘到存储池（或直接放到 target 路径并刷新）：

```
virsh pool-refresh default
```

- 列出/管理虚拟网络：

```
virsh net-list --all
virsh net-start default
virsh net-autostart default
virsh net-define mynet.xml   # 用 XML 定义自定义网络（bridge/internal）
```

---

如果你告诉我要做的具体操作（比如：导入某个 ISO、创建带两张网卡的 VM、把 VM 放到桥接网络 br0、或想用 qcow2 快照策略），我可以给出对应的完整命令或 virt-manager 的逐步点击说明。
