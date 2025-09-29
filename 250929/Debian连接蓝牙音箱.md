在 Debian 13 使用 i3 窗口管理器（i3wm）管理蓝牙的步骤如下：

### 安装蓝牙工具

首先，确保安装了必要的蓝牙工具。打开终端并运行以下命令：

```bash
sudo apt update
sudo apt install pulseaudio pulseaudio-module-bluetooth pavucontrol bluez blueman
```

- `pulseaudio` 和 `pulseaudio-module-bluetooth` 用于音频管理。
- `bluez` 是蓝牙协议栈，`blueman` 是图形化的蓝牙管理工具。

### 启动蓝牙服务

确保蓝牙服务正在运行：

```bash
sudo systemctl start bluetooth
sudo systemctl enable bluetooth
```

### 使用 Blueman 连接蓝牙音箱

1. **启动 Blueman**：在终端中输入 `blueman-manager` 启动 Blueman 界面。
2. **打开蓝牙**：确保 Bluetooth 设备处于开启状态，在 Blueman 界面中点击“设备”菜单，选择“搜索”来查找可用的蓝牙设备。
3. **配对音箱**：

   - 找到你的蓝牙音箱，右键点击并选择“配对”。
   - 根据提示完成配对过程（可能需要输入配对码）。
4. **连接音箱**：

   - 配对成功后，右键点击音箱设备，选择“连接”。

### 设置音频输出

1. **打开音量控制**：运行 `pavucontrol`，打开 PulseAudio 音量控制。
2. **选择音频设备**：
   - 在“播放”选项卡中，确保音频流通过蓝牙音箱播放。
   - 在“输出设备”选项卡中，选择你的蓝牙音箱作为输出设备。

### 其他命令行管理

如果你更喜欢使用命令行，可以使用以下命令：

- **列出蓝牙设备**：

  ```bash
  bluetoothctl
  ```
- **打开蓝牙**：

  ```bash
  power on
  ```
- **扫描设备**：

  ```bash
  scan on
  ```
- **配对设备**：

  ```bash
  pair <device_mac_address>
  ```
- **连接设备**：

  ```bash
  connect <device_mac_address>
  ```

### 解决问题

如果遇到任何问题，确保你的蓝牙设备在配对模式，并检查系统设置中的蓝牙状态。

按照以上步骤，你应该能够在 Debian 13 的 i3wm 环境中顺利连接和管理蓝牙音箱。
