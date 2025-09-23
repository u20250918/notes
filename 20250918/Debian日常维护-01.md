## 日常维护总览（每天/每周/每月可执行的步骤）

---

### 每日（或每次登录后）

- **更新包信息并查看可更新项**：
  - sudo apt update
- **检查是否有重要安全更新并升级**（若不在自动升级环境）：
  - sudo apt upgrade   或按需 sudo apt full-upgrade
- **查看系统日志异常（快速）**：
  - sudo journalctl -p err -b --no-pager
- **检查磁盘空间**：
  - df -h
- **检查根目录/重要分区 inode 使用**：
  - df -i
- **检查系统负载与运行状况**：
  - top 或 htop
- **检查服务状态（关键服务）**：
  - sudo systemctl status 服务名

---

### 每周

- **清理无用包与缓存**：
  - sudo apt autoremove
  - sudo apt autoclean
- **检查自动更新是否正常（unattended-upgrades）**：
  - sudo systemctl status unattended-upgrades
  - 查看 /var/log/unattended-upgrades/ 与 /var/log/dpkg.log
- **检查并轮转日志配置（验证 logrotate）**：
  - sudo logrotate -d /etc/logrotate.conf  （模拟运行）
- **检查磁盘 SMART 健康（针对物理服务器/磁盘）**：
  - sudo apt install smartmontools
  - sudo smartctl -H /dev/sdX
- **备份检测**（确保备份任务成功）：
  - 检查备份日志并做一次恢复演练

---

### 每月

- **完整系统升级并重启（如内核有更新）**：
  - sudo apt update && sudo apt full-upgrade
  - sudo reboot （如内核或关键服务更新）
- **文件系统一致性检查（需要单用户或维护窗口）**：
  - sudo touch /forcefsck && sudo reboot  （Debian 会在启动时检查）
  - 或线下使用 fsck /dev/sdXN
- **审计已安装软件与安全配置**：
  - 列出手动安装包： aptitude search '~i !~M' （或 dpkg --get-selections）
  - 检查开放端口与监听服务： ss -tuln 或 sudo netstat -tulnp
- **检查用户与权限、SSH 配置审计**：
  - sudo cat /etc/ssh/sshd_config （确认禁止 root 登录、使用密钥等）
  - lastlog 与 last 检查登录历史
- **更新并测试备份策略，轮换旧备份**

---

### 季度或按需（更深入）

- **进行发行版升级前的准备与测试**：
  - 参考 /usr/share/doc/apt/… 和 Debian Release Notes
- **安全扫描与漏洞评估**：
  - 使用 Lynis、OpenVAS 等工具扫描
- **硬件固件/BIOS 更新（按厂商指示）**
- **性能基线和容量规划**（日志、磁盘、内存趋势）

---

## 推荐工具与配置（简短清单）

- 自动更新：**unattended-upgrades**
- 更新通知：**apt-listchanges**
- 日志管理：**rsyslog / journalctl & logrotate**
- 监控告警：**Prometheus + Alertmanager、Grafana** 或轻量的 netdata
- 备份：**rsync / borg / restic**（并定期验证）
- 磁盘健康：**smartmontools**
- 安全审计：**Lynis、fail2ban、ufw/iptables**

---

## 常用命令速查

```
sudo apt update
sudo apt upgrade
sudo apt full-upgrade
sudo apt autoremove
sudo apt autoclean
sudo systemctl status unattended-upgrades
sudo journalctl -p err -b --no-pager
df -h
df -i
ss -tuln
sudo smartctl -H /dev/sdX
```

---

如果你需要，我可以把上面步骤整理成可执行的脚本或 systemd timer / cron 作业，或按“桌面/小型服务器/生产服务器”场景给出更具体的自动化配置。
