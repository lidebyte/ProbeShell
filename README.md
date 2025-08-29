# ProbeShell - 服务器监控探针部署脚本

## 📖 项目简介

ProbeShell 是一个完整的服务器监控探针自动化部署脚本，用于在 VPS 上快速部署监控组件，将监控数据发送到 VictoriaMetrics 和 Loki 系统。

## ✨ 主要特性

- 🚀 **一键部署**: 自动化安装所有监控组件
- 🏗️ **架构支持**: 支持 x86_64 和 ARM64 架构
- 🔧 **灵活配置**: 支持命令行参数和交互式配置
- 📊 **全面监控**: 系统指标、网络探针、日志收集
- 🏷️ **标签丰富**: 自动添加地理位置、运营商、分组等标签
- 🔄 **服务管理**: 自动配置 systemd 服务并设置开机自启

## 🧩 监控组件

| 组件 | 端口 | 功能描述 |
|------|------|----------|
| **Blackbox Exporter** | 9115 | 网络可达性监控 (ICMP/TCP) |
| **Node Exporter** | 9100 | 系统指标收集 (CPU/内存/磁盘/网络) |
| **VMagent** | - | 数据收集和转发到 VictoriaMetrics |
| **Promtail** | - | 日志收集和转发到 Loki |

## 📋 系统要求

- **操作系统**: Ubuntu/Debian/CentOS/RHEL
- **架构**: x86_64 或 ARM64
- **权限**: root 权限或 sudo 权限
- **网络**: 能够访问 GitHub 和外部监控服务器

## 🚀 快速开始

### 1. 下载脚本

```bash
git clone https://github.com/lidebyte/ProbeShell.git
cd ProbeShell
chmod +x agent.sh
```

### 2. 安装监控组件

#### 方式一：命令行安装（推荐）

```bash
./agent.sh --install \
  --victoria "192.168.1.100:8428" \
  --loki "192.168.1.100:3100" \
  --name "HK-Frontend-01" \
  --group "Frontend" \
  --location "HongKong" \
  --vm-user "admin" \
  --vm-pass "password123"
```

#### 方式二：交互式安装

```bash
./agent.sh
# 选择选项 1，然后按提示输入参数
```

## 📝 参数说明

| 参数 | 别名 | 说明 | 是否必填 | 示例值 |
|------|------|------|----------|---------|
| `--install` | - | 安装所有组件 | 是 | - |
| `--victoria` | `--vm` | VictoriaMetrics 写入地址 | 是 | `192.168.1.100:8428` |
| `--loki` | - | Loki 写入地址 | 是 | `192.168.1.100:3100` |
| `--name` | `--instance` | VPS 实例名称 | 是 | `HK-Frontend-01` |
| `--group` | - | 分组类型 | 是 | `Frontend`、`Backend`、`Service` |
| `--location` | - | 地理位置 | 是 | `HongKong`、`Japan`、`Singapore` |
| `--vm-user` | `--username` | VictoriaMetrics 用户名 | 是 | `admin` |
| `--vm-pass` | `--password` | VictoriaMetrics 密码 | 是 | `password123` |

## 🔧 配置说明

### 监控目标配置

#### TCP 连接测试 (`endpoint.yml`)
- 湖北电信/联通/移动
- 山东电信/联通/移动
- 使用 `zstaticcdn.com` 的测试节点

#### ICMP 测试 (`icmp.yml`)
- 同样覆盖湖北和山东的三大运营商
- 使用 `ddnstoday.xyz` 的测试节点

### 系统日志收集

Promtail 自动收集以下系统日志：
- `/var/log/syslog` - 系统日志
- `/var/log/auth.log` - 认证日志
- `/var/log/kern.log` - 内核日志
- `/var/log/cron.log` - 定时任务日志
- `/var/log/user.log` - 用户日志
- `/var/log/fail2ban.log` - 安全日志

## 📊 监控数据标签

安装完成后，所有监控数据会自动添加以下标签：

- **instance**: VPS 实例名称
- **group**: 分组类型 (Frontend/Backend/Service 等)
- **location**: 地理位置 (HongKong/Japan/Singapore 等)
- **name**: 测试目标名称
- **code**: 地区代码
- **city**: 城市名称
- **isp**: 运营商 (China Telecom/China Unicom/China Mobile)
- **ip**: IP 版本 (IPv4/IPv6)
- **domestic**: 是否国内 (true/false)

## 🛠️ 管理操作

### 查看服务状态

```bash
systemctl status blackbox
systemctl status node_exporter
systemctl status vmagent
systemctl status promtail
```

### 查看日志

```bash
journalctl -u blackbox -f
journalctl -u node_exporter -f
journalctl -u vmagent -f
journalctl -u promtail -f
```

### 重启服务

```bash
systemctl restart blackbox
systemctl restart node_exporter
systemctl restart vmagent
systemctl restart promtail
```

## 🗑️ 卸载组件

```bash
./agent.sh --uninstall --delete-logs
```

**参数说明**:
- `--delete-logs`: 删除系统日志（可选）

## 📁 项目结构

```
ProbeShell/
├── agent.sh              # 主安装脚本
├── README.md             # 项目说明文档
├── .gitignore            # Git 忽略文件
├── blackbox/             # Blackbox Exporter 配置
│   └── blackbox.yml      # 探针模块配置
├── vmagent/              # VMagent 配置
│   ├── prometheus.yml    # 主配置文件
│   ├── endpoint.yml      # TCP 连接测试目标
│   └── icmp.yml          # ICMP 测试目标
├── promtail/             # Promtail 配置
│   └── promtail.yml      # 日志收集配置
└── service/              # Systemd 服务文件
    ├── blackbox.service
    ├── node_exporter.service
    ├── vmagent.service
    └── promtail.service
```

## 🔍 故障排除

### 常见问题

1. **服务启动失败**
   - 检查配置文件语法
   - 查看服务日志: `journalctl -u <service_name> -f`
   - 确认端口未被占用

2. **监控数据无法发送**
   - 检查网络连接
   - 验证 VictoriaMetrics 地址和认证信息
   - 确认防火墙设置

3. **日志收集失败**
   - 检查 Loki 服务器地址
   - 确认日志文件权限
   - 查看 Promtail 日志

### 日志位置

- **Blackbox**: `/var/log/blackbox_exporter.log`
- **Node Exporter**: `/var/log/node_exporter.log`
- **VMagent**: `/var/log/vmagent.log`
- **Promtail**: `/var/log/promtail.log`

## 🤝 贡献指南

欢迎提交 Issue 和 Pull Request 来改进这个项目！

### 开发环境

1. Fork 本仓库
2. 创建特性分支: `git checkout -b feature/AmazingFeature`
3. 提交更改: `git commit -m 'Add some AmazingFeature'`
4. 推送分支: `git push origin feature/AmazingFeature`
5. 提交 Pull Request

## 📄 许可证

本项目采用 MIT 许可证 - 查看 [LICENSE](LICENSE) 文件了解详情。

## 🙏 致谢

感谢以下开源项目：
- [Prometheus](https://prometheus.io/) - 监控系统
- [VictoriaMetrics](https://victoriametrics.com/) - 时序数据库
- [Loki](https://grafana.com/oss/loki/) - 日志聚合系统
- [Blackbox Exporter](https://github.com/prometheus/blackbox_exporter) - 网络探针

## 📞 联系方式

- **项目地址**: https://github.com/lidebyte/ProbeShell
- **问题反馈**: https://github.com/lidebyte/ProbeShell/issues

---

⭐ 如果这个项目对你有帮助，请给它一个星标！
