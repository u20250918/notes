下面给出在 Debian 13（Trixie）上安装 VirtualBox 的简明步骤，适用于 amd64 主机。

---

## 1) 导入 Oracle 签名密钥

```bash
wget -qO- https://www.virtualbox.org/download/oracle_vbox_2016.asc | sudo gpg --dearmor -o /usr/share/keyrings/oracle-virtualbox-2016.gpg
```

**说明**：这一步确保从官方仓库安装的软件能被验证。

---

## 2) 添加 VirtualBox 官方仓库

```bash
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/oracle-virtualbox-2016.gpg] https://download.virtualbox.org/virtualbox/debian trixie contrib" | sudo tee /etc/apt/sources.list.d/virtualbox.list
```

---

## 3) 更新包索引并安装 VirtualBox

```bash
sudo apt update
sudo apt install virtualbox-7.2
```

**如果想安装其它次版本**，把 `virtualbox-7.2` 换成对应包名（例如 `virtualbox-7.1`）。

---

## 4) （可选）安装 Extension Pack

官方扩展包提供 USB 3、VRDP 等功能。先查询已安装版本：

```bash
vboxmanage -v
```

然后从 https://www.virtualbox.org/download 下载匹配版本的 Extension Pack，安装：

```bash
sudo VBoxManage extpack install --replace /path/to/Oracle_VM_VirtualBox_Extension_Pack-<version>.vbox-extpack
```

---

## 5) 添加用户到 vboxusers（允许访问 USB 等）

```bash
sudo usermod -aG vboxusers $USER
```

添加后注销或重启以生效。

---

## 6) 如果遇到内核模块问题

安装构建工具与头文件：

```bash
sudo apt install build-essential linux-headers-$(uname -r) dkms
sudo /sbin/vboxconfig
```

若报错，贴出具体错误我可以帮排查。

---

水平线分隔（已包含）。如需针对 ARM（Apple Silicon）、Debian 非 amd64 或用 Debian 官方 Fast Track / Debian 仓库方式安装，请说明你的平台与偏好。
