在 Debian 13 中关闭蓝牙的设置可以通过以下几种方法实现：

### 1. 使用 `bluetoothctl` 命令行工具

1. 打开终端。
2. 输入以下命令启动蓝牙控制工具：

   ```bash
   bluetoothctl
   ```
3. 在进入 `bluetoothctl` 后，输入以下命令关闭蓝牙：

   ```bash
   power off
   ```
4. 输入 `exit` 退出 `bluetoothctl`。

### 2. 使用 Systemd 服务

你也可以通过 Systemd 停止蓝牙服务：

```bash
sudo systemctl stop bluetooth
```

如果你想在下次启动时禁用蓝牙服务，可以执行：

```bash
sudo systemctl disable bluetooth
```

### 3. 使用 Blueman 图形界面

如果你已经安装了 Blueman，可以通过图形界面关闭蓝牙：

1. 打开 Blueman（在终端中输入 `blueman-manager`）。
2. 在窗口中，点击“蓝牙”菜单。
3. 选择“关闭蓝牙”选项。

### 4. 使用设置菜单（如果有）

如果你使用的是带有 GNOME 或其他桌面环境的系统，通常可以在设置中找到蓝牙选项，直接关闭它。

选择你觉得最方便的方法来关闭蓝牙设置。
