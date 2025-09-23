要把日常维护自动化并在系统启动后自运行，我会给出以下内容：

- 一个完整的 systemd service + timer（或 timer + oneshot service）实现周期性任务（每日、每周、每月）；
- 各类维护任务的可配置脚本（更新、清理、日志检查、SMART、备份验证、安全扫描）；
- 集成 unattended-upgrades、fail2ban（或“安全狗”若指中国厂商安全软件需说明兼容性），并在脚本中包含安全日志/策略检查；
- 日志记录与轮换、错误通知（可通过邮件或自定义 webhook）；
- 配置示例、权限与 SELinux/AppArmor 注意事项、测试步骤与回滚方法。

在继续前我需要两个确认：

1) 你说的“安全狗”是指阿里云/腾讯/安全狗（安全软件厂商）中的哪一个具体产品？还是泛指防护/入侵检测（如 fail2ban / AIDE / Wazuh）？不同产品安装/集成方式不同，我需要明确目标软件。
2) 目标机器类型：桌面、家用小服务器、还是生产服务器（有严格维护窗口与变更控制）？以及 Debian 版本（例如 Debian 12 bookworm / 11 bullseye）？

确认后我会生成：

- 完整可复制的脚本文件（bash），放到 /usr/local/bin，带日志和退出码；
- systemd 单元和 timer 文件（每日/每周/每月不同频率），并说明如何启用与测试；
- 配置说明（包含邮件/webhook 通知、备份验证、与“安全狗”的集成步骤）；
- 回滚与故障排查要点。


---



下面给出一套面向 Debian 13 桌面（GNOME）、用于个人电脑的自启动自动化维护解决方案。包含一个主维护脚本（/usr/local/bin/debian-maintenance.sh），以及 systemd timer+service 实现每日/每周任务。脚本侧重安全、更新、清理、日志检查、SMART 健康检查、备份验证与通知（邮件或 webhook 可选）。假设“安全狗”指泛指入侵防护，脚本集成 fail2ban 检查；若你要特定第三方“安全狗”产品，请另行说明。

重要提示：在运行前请先阅读并按需修改脚本中的 EMAIL、WEBHOOK、BACKUP_DIR、EXCLUDE_DIRS 等变量，测试环境先手动运行一次以确认无误。

将以下内容以 root 权限保存并设置可执行。

1) 主维护脚本 /usr/local/bin/debian-maintenance.sh

```bash
#!/usr/bin/env bash
# debian-maintenance.sh
# 适用于 Debian 13 (GNOME) 个人电脑的日常维护自动化脚本
# 放置 /usr/local/bin/debian-maintenance.sh，并 chmod +x

set -euo pipefail
IFS=$'\n\t'

# ------------------------
# 配置区（按需修改）
# ------------------------
EMAIL="you@example.com"          # 若为空则不发邮件
WEBHOOK_URL=""                   # 若使用 webhook（例如 Slack/IFTTT），否则留空
LOGDIR="/var/log/debian-maintenance"
LOGFILE="${LOGDIR}/maintenance-$(date +%F).log"
KEEP_LOG_DAYS=30

BACKUP_DIR="/home/youruser/backups"    # 如果有备份，脚本会检查最新备份时间
BACKUP_MAX_AGE_DAYS=7

SMART_DEVICES="/dev/sda"         # 用空隔分隔多个设备，例如 "/dev/sda /dev/sdb"
EXCLUDE_DIRS="/proc /sys /run /dev /tmp"

APT_LOCK_TIMEOUT=300             # apt 被锁定时等待秒数

# 任务开关
DO_UPDATE=true
DO_AUTOREMOVE=true
DO_AUTOCLEAN=true
DO_SMART_CHECK=true
DO_LOG_CHECK=true
DO_BACKUP_CHECK=true
DO_FAIL2BAN_CHECK=true

# ------------------------
# 初始化
# ------------------------
mkdir -p "${LOGDIR}"
touch "${LOGFILE}"
exec 3>&1 1>>"${LOGFILE}" 2>&1

timestamp() { date '+%F %T'; }
log() { echo "$(timestamp) $*"; }

# ------------------------
# 帮助与轻量邮件/通知
# ------------------------
notify() {
  local subject="$1"; shift
  local body="$*"
  # 控制台输出（也写入日志）
  echo "NOTIFY: ${subject} -- ${body}"
  # 邮件（如果设置 EMAIL 且 mail 命令可用）
  if [[ -n "${EMAIL}" ]] && command -v mail >/dev/null 2>&1; then
    echo "${body}" | mail -s "${subject}" "${EMAIL}"
  fi
  # Webhook
  if [[ -n "${WEBHOOK_URL}" ]]; then
    if command -v curl >/dev/null 2>&1; then
      curl -fsS -m 10 -X POST -H "Content-Type: application/json" \
        -d "$(printf '{"text":"%s: %s"}' "${subject}" "${body}" | sed 's/"/\\"/g')" \
        "${WEBHOOK_URL}" >/dev/null 2>&1 || true
    fi
  fi
}

# ------------------------
# 等待 APT 锁（避免在自动脚本中失败）
# ------------------------
wait_for_apt() {
  local waited=0
  while fuser /var/lib/dpkg/lock >/dev/null 2>&1 || fuser /var/lib/apt/lists/lock >/dev/null 2>&1; do
    if (( waited >= APT_LOCK_TIMEOUT )); then
      log "APT lock timeout after ${waited}s"
      return 1
    fi
    log "Waiting for apt lock..."
    sleep 3
    waited=$((waited+3))
  done
  return 0
}

# ------------------------
# 子任务：更新与升级
# ------------------------
do_update() {
  if ! $DO_UPDATE; then
    log "Update disabled"
    return 0
  fi
  log "=== APT update/upgrade start ==="
  if ! wait_for_apt; then
    notify "Maintenance: APT lock" "Could not acquire apt lock"
    return 1
  fi
  apt update -y
  # 仅升级普通软件包，不自动删除包；若需要移除旧内核可使用 autoremove
  apt -y upgrade
  # 若需要完全升级（处理依赖变更），可启用 full-upgrade:
  # apt -y full-upgrade
  log "=== APT update/upgrade done ==="
}

# ------------------------
# 子任务：清理
# ------------------------
do_cleanup() {
  log "=== Autoremove/autoclean start ==="
  if $DO_AUTOREMOVE; then
    apt -y autoremove
  fi
  if $DO_AUTOCLEAN; then
    apt -y autoclean
  fi
  log "=== Autoremove/autoclean done ==="
}

# ------------------------
# 子任务：SMART 检查
# ------------------------
do_smart() {
  if ! $DO_SMART_CHECK; then
    log "SMART check disabled"
    return 0
  fi
  if ! command -v smartctl >/dev/null 2>&1; then
    log "smartctl not found — installing smartmontools"
    apt -y install smartmontools
  fi
  for dev in ${SMART_DEVICES}; do
    if [[ -b "${dev}" ]]; then
      log "SMART test for ${dev}"
      smartctl -H "${dev}" || {
        notify "SMART warning" "smartctl reported problems on ${dev}"
      }
    else
      log "Device ${dev} not found or not a block device"
    fi
  done
}

# ------------------------
# 子任务：日志错误检查（systemd/journalctl）
# ------------------------
do_log_check() {
  if ! $DO_LOG_CHECK; then
    log "Log check disabled"
    return 0
  fi
  log "=== Journal error check (since boot) ==="
  local err_count
  err_count=$(journalctl -p err -b --no-pager | wc -l || true)
  if (( err_count > 0 )); then
    local summary
    summary=$(journalctl -p err -b --no-pager | head -n 200)
    notify "Journal errors: ${err_count}" "${summary}"
  else
    log "No journal errors since boot"
  fi
  # 检查 Xorg / GNOME 崩溃日志
  if [[ -d "/var/crash" ]]; then
    local crash_count
    crash_count=$(find /var/crash -type f | wc -l || true)
    if (( crash_count > 0 )); then
      notify "Crash reports" "Found ${crash_count} files in /var/crash"
    fi
  fi
}

# ------------------------
# 子任务：备份检查（基于时间戳）
# ------------------------
do_backup_check() {
  if ! $DO_BACKUP_CHECK; then
    log "Backup check disabled"
    return 0
  fi
  if [[ -z "${BACKUP_DIR}" || ! -d "${BACKUP_DIR}" ]]; then
    log "Backup dir not configured or missing: ${BACKUP_DIR}"
    return 0
  fi
  local newest
  newest=$(find "${BACKUP_DIR}" -type f -printf '%T@ %p\n' 2>/dev/null | sort -n | tail -n1 | cut -d' ' -f2- || true)
  if [[ -z "${newest}" ]]; then
    notify "Backup missing" "No backup files found in ${BACKUP_DIR}"
    return 1
  fi
  local newest_age
  newest_age=$(( ( $(date +%s) - $(stat -c %Y "${newest}") ) / 86400 ))
  if (( newest_age > BACKUP_MAX_AGE_DAYS )); then
    notify "Backup stale" "Newest backup ${newest} is ${newest_age} days old"
  else
    log "Backup OK: ${newest} (${newest_age} days)"
  fi
}

# ------------------------
# 子任务：fail2ban / 入侵检测简单检查
# ------------------------
do_fail2ban_check() {
  if ! $DO_FAIL2BAN_CHECK; then
    log "Fail2ban check disabled"
    return 0
  fi
  if ! command -v fail2ban-client >/dev/null 2>&1; then
    log "fail2ban not installed — skipping"
    return 0
  fi
  # 列出最近被禁止的 IP（示例）
  if fail2ban-client status >/dev/null 2>&1; then
    local banned
    banned=$(fail2ban-client status | sed -n 's/.*Banned IP list:\s*//p')
    log "Fail2ban banned IPs: ${banned:-none}"
  fi
}

# ------------------------
# 子任务：日志轮换与过期日志清理（维护脚本自身）
# ------------------------
rotate_logs() {
  find "${LOGDIR}" -type f -name 'maintenance-*.log' -mtime +"${KEEP_LOG_DAYS}" -delete || true
}

# ------------------------
# 主流程
# ------------------------
main() {
  log "Maintenance run start"

  # 检查 root 权限（某些操作需要）
  if (( EUID != 0 )); then
    log "Not running as root — some tasks will be skipped. Re-run as root for full functionality."
  fi

  # 更新与清理
  do_update || log "Update step failed"
  do_cleanup || log "Cleanup step failed"

  # SMART
  do_smart || log "SMART step failed"

  # 日志检查
  do_log_check || log "Log check failed"

  # 备份检查
  do_backup_check || log "Backup check failed"

  # fail2ban 检查
  do_fail2ban_check || log "Fail2ban check failed"

  # 自身日志轮换
  rotate_logs

  log "Maintenance run end"
  # 复制日志到控制台（保留 fd3）
  exec 1>&3 3>&- 
  echo "Maintenance completed. Log: ${LOGFILE}"
}

main "$@"
```

2) systemd service + timer（每日运行）
   保存 /etc/systemd/system/debian-maintenance.service

```ini
[Unit]
Description=Debian personal maintenance script
After=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/debian-maintenance.sh
Nice=10
StandardOutput=journal
StandardError=journal
```

保存 /etc/systemd/system/debian-maintenance.timer

```ini
[Unit]
Description=Daily timer for Debian personal maintenance

[Timer]
OnCalendar=daily
Persistent=true
WakeSystem=false

[Install]
WantedBy=timers.target
```

如果你也想做每周或每月更深入的任务（例如 full-upgrade、fsck、备份演练），可以新增 debian-maintenance-weekly.service + timer，OnCalendar=weekly，或 OnCalendar=monthly。

3) 安装与启用
   运行（以 root）：

```bash
chmod +x /usr/local/bin/debian-maintenance.sh
systemctl daemon-reload
systemctl enable --now debian-maintenance.timer
# 手动触发一次以测试
systemctl start debian-maintenance.service
journalctl -u debian-maintenance.service --no-pager -n 200
```

4) 关于“安全狗”与桌面环境兼容性说明（简短）

- **如果你指 fail2ban/AIDE/Wazuh 等开源入侵检测，脚本已支持 fail2ban 的基本检查。**
- 第三方闭源“安全狗”产品一般有自己的安装包与守护进程，需依据厂商文档安装，并把脚本中相应检查点（如进程是否在运行、日志位置）加入到 do_fail2ban_check 或新增 do_secu_god_check。

5) 测试与回滚要点

- 先手动运行脚本并检查 /var/log/debian-maintenance/*.log。
- 若自动升级导致问题，保留 apt 日志 /var/log/apt/history.log 与 /var/log/dpkg.log，使用 apt rollback 或手动降级包。
- 如需禁用自动运行： systemctl disable --now debian-maintenance.timer

---

如果需要，我可以：

- 把 weekly/monthly 的脚本模板也写出（包含 fsck/内核重启判断与备份恢复演练）。
- 根据你提供的 BACKUP_DIR 与是否使用邮件/Slack webhook 定制通知格式。
