# VPS Board

一个面向公开展示的 VPS 实时状态监控面板。

本项目适合部署十几台 VPS，采用：

- Cloudflare Pages：部署前端
- Go：后端 API 和 VPS Agent
- SQLite：数据存储
- Caddy：HTTPS 和反向代理
- SSE：实时推送机器状态
- IPinfo Widget API：识别服务器位置、ISP、ASN 和网络风险
- Leaflet：世界地图
- CARTO：地图底图
- Devicon / Simple Icons：Linux 系统图标
- FlagCDN：国家国旗图标

---

## 项目声明

本项目由 OpenAI ChatGPT 根据实际需求协助设计、编写、重构和整理。

项目中的以下内容均由 OpenAI ChatGPT 协助制作：

- 前端页面结构
- 前端 CSS 样式
- Go 后端 API
- Go VPS Agent
- SQLite 数据结构
- IPinfo 查询逻辑
- IDC / ISP 判断
- IPinfo 风险值计算
- 世界地图
- 服务器详情页
- SLA 统计
- 历史数据清理
- 部署流程和文档

使用者应根据自己的服务器环境、安全要求和第三方服务条款，自行检查代码后再用于生产环境。
---

# 一、重要目录约定

本项目统一使用以下目录名称。

## GitHub 项目目录

```text
vps-board
```

## 后端服务器源码目录

```text
/opt/vps-board
```

## SQLite 数据目录

```text
/var/lib/vps-board
```

## 后端配置目录

```text
/etc/vps-board
```

## 后端可执行文件

```text
/usr/local/bin/vps-board-server
```

## Agent 可执行文件

```text
/usr/local/bin/vps-board-agent
```

之后所有命令都使用：

```bash
cd /opt/vps-board
```

---

# 二、功能

## 1. 首页

首页提供：

- VPS 总数量
- 当前在线数量
- 世界地图
- 服务器地理分布
- 国家国旗
- 城市和地区
- 服务器名称
- 在线状态
- CPU 使用率
- 内存使用率
- 硬盘使用率
- 当前上行速度
- 当前下行速度
- 中文 / English 切换
- 全局设置入口

卡片布局：

- 电脑端：每行两个服务器卡片
- 手机端：每行一个服务器卡片

没有添加机器时显示：

```text
没有 VPS，请添加
```

---

## 2. 首页服务器卡片

未进入详情页时，卡片只显示核心信息：

- 服务器名称
- 国家国旗
- 城市或地区
- 在线状态
- CPU 使用率
- 内存使用率
- 硬盘使用率
- 当前下行速度
- 当前上行速度

点击卡片进入详情页：

```text
/detail.html?id=机器ID
```

URL 中只使用内部机器 ID，不包含真实 IP。

---

## 3. 服务器详情页

详情页显示：

- 服务器名称
- 在线状态
- 国家和国旗
- 国家、地区、城市
- 机器提供商
- ISP
- ASN
- IDC / ISP / Business 等网络类型
- IPinfo 风险值
- VPN 状态
- Proxy 状态
- Tor 状态
- Relay 状态
- Linux 发行版
- Linux 版本
- CPU 架构
- CPU 核数
- 总内存
- 总硬盘
- 当前系统运行时间
- 累计在线时间
- 30 天 SLA
- CPU 24 小时趋势
- 内存和硬盘 24 小时趋势
- 上行和下行 24 小时趋势
- 最后一次上报时间

详情页地址：

```text
https://你的前端域名/detail.html?id=机器ID
```

---

## 4. VPS Agent

每台被监控 VPS 安装一个 Go Agent。

Agent 默认每 5 秒上报：

- Linux 发行版
- Linux 版本
- CPU 架构
- CPU 核数
- CPU 使用率
- 内存总量
- 内存使用量
- 硬盘总量
- 硬盘使用量
- 当前上行速度
- 当前下行速度
- 系统运行时间

Agent 不在 JSON 请求正文中主动提交公网 IP。

---

# 三、项目架构

```text
Cloudflare Pages
        │
        │ 前端网页
        ▼
浏览器
        │
        │ HTTPS API / SSE
        ▼
你的后端 API 域名
        │
      Caddy
        │
        ▼
Go Backend + SQLite
        │
        ├── 接收 VPS Agent 上报
        ├── 推送实时机器状态
        ├── 记录性能历史
        ├── 记录在线历史
        ├── 计算 SLA
        └── 请求 IPinfo
                   ▲
                   │
              每台 VPS 一个 Agent
```

本文档中使用以下占位符：

```text
API_DOMAIN=api.example.com
PAGES_DOMAIN=your-project.pages.dev
FRONTEND_DOMAIN=panel.example.com
```

部署时请替换为你自己的域名。

请不要把真实域名、Token、密钥写入 GitHub README。

---

# 四、项目结构

```text
vps-board/
├── README.md
├── LICENSE                         # 可选
├── .gitignore
├── go.mod
├── go.sum
│
├── cmd/
│   ├── server/
│   │   └── main.go                 # Go 后端
│   │
│   └── agent/
│       └── main.go                 # VPS 监控 Agent
│
└── frontend/
    ├── index.html                  # 首页
    ├── detail.html                 # 详情页
    ├── style.css                  # 全部 CSS
    ├── config.js                  # 前端 API 地址
    ├── app.js                     # 首页逻辑
    ├── detail.js                  # 详情页逻辑
    └── _headers                   # Cloudflare Pages 响应头
```

---

# 五、技术栈

| 组件 | 用途 |
|---|---|
| Go | 后端和 Agent |
| SQLite | 数据库 |
| Caddy | HTTPS 和反向代理 |
| Cloudflare Pages | 前端部署 |
| SSE | 实时状态推送 |
| IPinfo Widget API | 位置、ISP、ASN、网络风险 |
| Leaflet | 世界地图 |
| CARTO | 地图底图 |
| Devicon | Linux 图标 |
| Simple Icons | Linux 图标 |
| FlagCDN | 国家国旗 |

---

# 六、隐私设计

## 1. 前端不显示真实 IP

公共接口不会返回：

- IPv4
- IPv6
- IP 哈希
- Agent Token
- 管理员 Token

前端只会收到类似信息：

```json
{
  "network": {
    "isp": "Example ISP",
    "city": "Tokyo",
    "region": "Tokyo",
    "countryCode": "JP",
    "riskScore": 15,
    "riskSource": "IPinfo",
    "type": "IDC"
  }
}
```

## 2. 数据库不保存原始 IP

数据库只保存：

```text
ip_hash
```

这个值由以下方式生成：

```text
HMAC-SHA256(IP_HASH_KEY, VPS_IP)
```

用于：

- 判断 VPS IP 是否变化
- 查询网络信息缓存
- 避免重复请求 IPinfo

数据库不保存原始 IP。

## 3. 必须注意

为了查询 ISP、位置和风险，后端必须在收到 Agent 请求时临时取得来源 IP，并将被查询的 IP 发送给 IPinfo。

因此：

- IPinfo 会接收到被查询的 IP
- 后端在 Agent 请求期间会临时读取来源 IP
- 原始 IP 不写入 SQLite
- 原始 IP 不返回前端
- 原始 IP 不显示在网页
- Agent JSON 不主动提交 IP

---

# 七、IPinfo 位置和风险

当前使用：

```text
https://ipinfo.io/widget/demo/{IP}
```

例如：

```text
https://ipinfo.io/widget/demo/121.8.215.106
```

接口返回：

- 国家
- 城市
- 地区
- 经纬度
- ASN
- ISP
- Organization
- `hosting`
- `vpn`
- `proxy`
- `tor`
- `relay`

## 1. IDC / ISP 判断

后端按照以下规则判断：

```text
privacy.hosting = true
或 asn.type = hosting
或 company.type = hosting
→ IDC
```

```text
asn.type = isp
或 company.type = isp
→ ISP
```

其他类型可能显示：

```text
BUSINESS
EDUCATION
GOVERNMENT
UNKNOWN
```

## 2. IPinfo 风险值

当前显示的是项目根据 IPinfo 字段计算的自定义风险值，不是 Cloudflare 官方风险评分。

计算规则：

| 条件 | 分值 |
|---|---:|
| `hosting=true` | +15 |
| `relay=true` | +20 |
| `proxy=true` | +30 |
| `vpn=true` | +35 |
| `tor=true` | +60 |
| 最大值 | 100 |

示例：

```text
hosting=true
→ 风险值 15
```

```text
hosting=true + proxy=true
→ 风险值 45
```

页面显示：

```text
IP 风险 15
```

英文显示：

```text
IP Risk 15
```

---

# 八、历史数据和 SLA

## 1. 实时数据

Agent 每 5 秒上报一次。

实时数据用于：

- 首页实时状态
- 首页 CPU、内存、硬盘
- 首页上下行速度
- SSE 实时推送
- 详情页当前信息

实时上报不会每 5 秒写一条历史记录。

## 2. 性能历史

后端每分钟保存一条性能历史：

- CPU 使用率
- 内存使用率
- 硬盘使用率
- 上行速度
- 下行速度
- 在线状态

性能历史只保留：

```text
最近 24 小时
```

每台机器最多约：

```text
24 × 60 = 1440 条
```

超过 24 小时的数据自动删除。

## 3. SLA 历史

后端每分钟记录在线状态：

```text
online = 1
offline = 0
```

SLA 历史保留：

```text
最近 30 天
```

SLA 计算方式：

```text
在线分钟数 ÷ 已观测分钟数 × 100%
```

Agent 安装之前的时间不会被计算为离线。

如果后端没有记录到某一分钟，则属于：

```text
unknown
```

不会错误地计算为离线。

## 4. 累计在线时间

后端根据连续 Agent 上报计算累计在线时间。

如果两次上报之间的时间间隔不超过：

```ini
OFFLINE_AFTER_SECONDS=20
```

则将这段时间计入累计在线时间。

Agent 默认每 5 秒上报一次，因此正常情况下累计在线时间会持续增加。

---

# 九、数据库结构

主要数据表：

```text
machines
network_cache
availability_history
metrics_history
```

## 1. machines

保存：

- 机器名称
- 提供商
- Agent Token 哈希
- IP HMAC 哈希
- ISP
- 城市
- 地区
- 国家代码
- 经纬度
- IPinfo 风险值
- Linux 信息
- CPU、内存、硬盘
- 网络速率
- 当前运行时间
- 累计在线时间
- 首次监控时间
- 最后上报时间

## 2. network_cache

按照 IP 哈希缓存 IPinfo 查询结果：

- ISP
- Organization
- ASN
- 城市
- 地区
- 国家
- 经纬度
- 风险值
- IDC / ISP
- VPN / Proxy / Tor / Relay

默认缓存：

```text
24 小时
```

## 3. availability_history

保存最近 30 天的在线状态。

逻辑字段：

```text
machine_id
bucket_at
online
```

## 4. metrics_history

保存最近 24 小时的性能数据。

逻辑字段：

```text
machine_id
bucket_at
online
cpu_usage
memory_usage
disk_usage
network_download_bps
network_upload_bps
```

---

# 十、数据库自动迁移

升级旧版本时，不要删除：

```text
/var/lib/vps-board/board.db
```

后端启动时会自动：

- 创建新表
- 增加缺少字段
- 创建索引
- 失效旧网络缓存
- 保留现有机器
- 保留 Agent Token 哈希
- 保留管理员配置

升级前建议备份：

```bash
mkdir -p /var/backups/vps-board

sqlite3 \
  /var/lib/vps-board/board.db \
  ".backup '/var/backups/vps-board/before-upgrade-$(date +%F-%H%M%S).db'"
```

查看表：

```bash
sqlite3 \
  /var/lib/vps-board/board.db \
  ".tables"
```

应该包含：

```text
availability_history
metrics_history
machines
network_cache
```

---

# 十一、从零部署后端

## 1. 登录服务器

```bash
ssh root@你的后端服务器
```

## 2. 安装基础软件

```bash
apt update

apt install -y \
  curl \
  ca-certificates \
  git \
  sqlite3 \
  openssl \
  unzip \
  zip \
  file \
  dnsutils \
  ufw
```

## 3. 创建项目目录

```bash
mkdir -p /opt/vps-board
mkdir -p /var/lib/vps-board
mkdir -p /etc/vps-board
```

创建系统用户：

```bash
id vps-board >/dev/null 2>&1 || useradd \
  --system \
  --no-create-home \
  --shell /usr/sbin/nologin \
  vps-board
```

设置数据库目录：

```bash
chown \
  vps-board:vps-board \
  /var/lib/vps-board

chmod 700 \
  /var/lib/vps-board
```

## 4. 上传源码

从本地电脑执行：

```bash
scp -r \
  vps-board \
  root@你的后端服务器:/opt/
```

最终源码目录必须是：

```text
/opt/vps-board
```

检查：

```bash
find \
  /opt/vps-board \
  -maxdepth 3 \
  -type f
```

## 5. 生成密钥

管理员密钥：

```bash
openssl rand -hex 32
```

IP 哈希密钥：

```bash
openssl rand -hex 32
```

两个值必须不同。

## 6. 创建环境文件

```bash
nano /etc/vps-board/backend.env
```

内容：

```ini
LISTEN_ADDRESS=127.0.0.1:8080
DATABASE_PATH=/var/lib/vps-board/board.db

ADMIN_TOKEN=替换为真实管理员密钥
IP_HASH_KEY=替换为另一段随机密钥

IPINFO_API_BASE=https://ipinfo.io/widget/demo

ALLOWED_ORIGINS=https://your-project.pages.dev

NETWORK_TTL_SECONDS=86400
NETWORK_FAILURE_COOLDOWN_SECONDS=600
IPINFO_REQUEST_INTERVAL_MS=1200
OFFLINE_AFTER_SECONDS=20
```

如果还没有前端域名：

```ini
ALLOWED_ORIGINS=
```

前端部署完成后再填写。

保护：

```bash
chown root:root \
  /etc/vps-board/backend.env

chmod 600 \
  /etc/vps-board/backend.env
```

## 7. 编译后端

```bash
cd /opt/vps-board
```

格式化：

```bash
gofmt -w \
  cmd/server/main.go \
  cmd/agent/main.go
```

整理依赖：

```bash
go mod tidy
```

编译：

```bash
CGO_ENABLED=0 go build \
  -trimpath \
  -ldflags="-s -w" \
  -o vps-board-server \
  ./cmd/server
```

安装：

```bash
install \
  -m 0755 \
  vps-board-server \
  /usr/local/bin/vps-board-server
```

检查：

```bash
ls -lh \
  /usr/local/bin/vps-board-server
```

---

# 十二、后端 Systemd 服务

创建：

```bash
cat >/etc/systemd/system/vps-board.service <<'EOF'
[Unit]
Description=VPS Board Go API
After=network-online.target
Wants=network-online.target

[Service]
Type=simple

User=vps-board
Group=vps-board

EnvironmentFile=/etc/vps-board/backend.env

ExecStart=/usr/local/bin/vps-board-server

Restart=always
RestartSec=3

UMask=0077

NoNewPrivileges=true
PrivateTmp=true
ProtectHome=true
ProtectSystem=strict

ReadWritePaths=/var/lib/vps-board

[Install]
WantedBy=multi-user.target
EOF
```

启动：

```bash
systemctl daemon-reload
systemctl enable --now vps-board
```

检查：

```bash
systemctl status \
  vps-board \
  --no-pager
```

测试本机：

```bash
curl -fsS \
  http://127.0.0.1:8080/api/v1/health
```

---

# 十三、Caddy 配置

编辑：

```bash
nano /etc/caddy/Caddyfile
```

使用你的真实 API 域名替换下面的占位符：

```caddy
api.example.com {
    reverse_proxy 127.0.0.1:8080 {
        header_up X-Real-IP {remote_host}
    }
}
```

格式化：

```bash
caddy fmt --overwrite \
  /etc/caddy/Caddyfile
```

验证：

```bash
caddy validate \
  --config /etc/caddy/Caddyfile
```

重启：

```bash
systemctl restart caddy
```

检查：

```bash
systemctl status \
  caddy \
  --no-pager
```

测试：

```bash
curl -fsS \
  https://api.example.com/api/v1/health
```

---

# 十四、Cloudflare Pages 部署

## 1. 修改前端配置

编辑：

```bash
nano /opt/vps-board/frontend/config.js
```

内容：

```js
window.VPS_BOARD_CONFIG = {
  API_BASE: "https://api.example.com"
};
```

注意：

- 必须有 `https://`
- 不要写结尾 `/`
- 不要将真实前端域名写入这里
- 这里填写后端 API 域名

## 2. 检查前端文件

```bash
cd /opt/vps-board/frontend
```

```bash
for file in \
  index.html \
  detail.html \
  style.css \
  config.js \
  app.js \
  detail.js \
  _headers
do
  if [ -s "$file" ]; then
    echo "正常：$file"
  else
    echo "错误：$file 缺失或为空"
  fi
done
```

## 3. 打包

```bash
rm -f /tmp/vps-board-frontend.zip

zip -r \
  /tmp/vps-board-frontend.zip \
  index.html \
  detail.html \
  style.css \
  config.js \
  app.js \
  detail.js \
  _headers
```

检查：

```bash
unzip -l \
  /tmp/vps-board-frontend.zip
```

压缩包根目录必须直接包含：

```text
index.html
detail.html
style.css
config.js
app.js
detail.js
_headers
```

不能多一层：

```text
frontend/index.html
```

## 4. 上传 Cloudflare Pages

进入：

```text
Cloudflare Dashboard
→ Workers & Pages
→ 你的 Pages 项目
→ Deployments
→ Create deployment
```

上传压缩包或解压后的根目录。

确认部署状态：

```text
Production
Success
```

---

# 十五、CORS 配置

假设 Cloudflare Pages 默认域名为：

```text
your-project.pages.dev
```

后端环境文件：

```ini
ALLOWED_ORIGINS=https://your-project.pages.dev
```

如果使用自定义前端域名：

```text
panel.example.com
```

则：

```ini
ALLOWED_ORIGINS=https://your-project.pages.dev,https://panel.example.com
```

修改后：

```bash
systemctl restart vps-board
```

测试：

```bash
curl -i \
  -H 'Origin: https://your-project.pages.dev' \
  https://api.example.com/api/v1/public/machines
```

响应中应有：

```text
Access-Control-Allow-Origin: https://your-project.pages.dev
```

注意：

- `ALLOWED_ORIGINS` 填前端地址
- 不填 API 地址
- 必须包含 `https://`
- 不能有结尾 `/`
- 多个域名用英文逗号隔开

---

# 十六、添加机器

打开前端页面：

```text
https://your-project.pages.dev
```

点击：

```text
设置
```

输入管理员密钥。

选择：

```text
添加新机器
```

填写：

- 机器名称
- 机器提供商，可选

当前版本不需要填写：

- 带宽
- 到期时间

保存后，页面会显示一次性 Agent Token。

Agent Token：

- 每台机器不同
- 后端只保存哈希
- 只显示一次
- 丢失后需要重置
- 不要提交到 GitHub
- 不要写进 README
- 不要发到公开聊天

Token 泄露后：

```text
设置
→ 选择对应机器
→ 重置 Token
```

---

# 十七、编译 Agent

进入项目目录：

```bash
cd /opt/vps-board
mkdir -p dist
```

## AMD64

```bash
CGO_ENABLED=0 \
GOOS=linux \
GOARCH=amd64 \
go build \
  -trimpath \
  -ldflags="-s -w" \
  -o dist/vps-board-agent-amd64 \
  ./cmd/agent
```

## ARM64

```bash
CGO_ENABLED=0 \
GOOS=linux \
GOARCH=arm64 \
go build \
  -trimpath \
  -ldflags="-s -w" \
  -o dist/vps-board-agent-arm64 \
  ./cmd/agent
```

查看目标 VPS 架构：

```bash
uname -m
```

对应关系：

| 架构 | 文件 |
|---|---|
| `x86_64` | `vps-board-agent-amd64` |
| `aarch64` | `vps-board-agent-arm64` |
| `arm64` | `vps-board-agent-arm64` |

---

# 十八、部署 Agent

## 1. 上传

AMD64：

```bash
scp \
  /opt/vps-board/dist/vps-board-agent-amd64 \
  root@目标VPS:/tmp/vps-board-agent
```

ARM64：

```bash
scp \
  /opt/vps-board/dist/vps-board-agent-arm64 \
  root@目标VPS:/tmp/vps-board-agent
```

## 2. 安装

在目标 VPS：

```bash
install \
  -m 0755 \
  /tmp/vps-board-agent \
  /usr/local/bin/vps-board-agent
```

删除临时文件：

```bash
rm -f /tmp/vps-board-agent
```

创建用户：

```bash
id vps-board-agent >/dev/null 2>&1 || useradd \
  --system \
  --no-create-home \
  --shell /usr/sbin/nologin \
  vps-board-agent
```

## 3. 创建配置

```bash
nano /etc/vps-board-agent.env
```

内容：

```ini
API_URL=https://api.example.com
AGENT_TOKEN=这台机器专属的AgentToken

REPORT_INTERVAL=5
REQUEST_TIMEOUT=10
ROOT_FILESYSTEM=/
```

如果需要手动指定网卡：

```ini
NETWORK_INTERFACE=eth0
```

查看默认网卡：

```bash
ip route show default
```

保护：

```bash
chown root:root \
  /etc/vps-board-agent.env

chmod 600 \
  /etc/vps-board-agent.env
```

## 4. 创建服务

```bash
cat >/etc/systemd/system/vps-board-agent.service <<'EOF'
[Unit]
Description=VPS Board Go Monitoring Agent
After=network-online.target
Wants=network-online.target

[Service]
Type=simple

User=vps-board-agent
Group=vps-board-agent

EnvironmentFile=/etc/vps-board-agent.env

ExecStart=/usr/local/bin/vps-board-agent

Restart=always
RestartSec=5

NoNewPrivileges=true
PrivateTmp=true
ProtectHome=true
ProtectSystem=strict

[Install]
WantedBy=multi-user.target
EOF
```

启动：

```bash
systemctl daemon-reload

systemctl enable --now \
  vps-board-agent
```

检查：

```bash
systemctl status \
  vps-board-agent \
  --no-pager
```

---

# 十九、验证监控数据

等待 10～20 秒：

```bash
sleep 15
```

查询：

```bash
curl -fsS \
  https://api.example.com/api/v1/public/machines \
  | jq '.machines[] | {
      name,
      status,
      system,
      metrics
    }'
```

正常应看到：

```json
{
  "name": "Example VPS",
  "status": "online",
  "system": {
    "osId": "ubuntu",
    "osName": "Ubuntu",
    "osVersion": "24.04",
    "architecture": "amd64",
    "uptimeSeconds": 123456
  },
  "metrics": {
    "cpuCores": 2,
    "cpuUsage": 1.2,
    "memoryUsage": 30.5,
    "diskUsage": 20.1,
    "networkDownloadBps": 0,
    "networkUploadBps": 0,
    "totalOnlineSeconds": 60
  }
}
```

如果离线：

```bash
systemctl status \
  vps-board-agent \
  --no-pager
```

```bash
journalctl \
  -u vps-board-agent \
  -n 100 \
  --no-pager
```

常见原因：

- Token 粘贴不完整
- Token 属于其他机器
- Token 已被重置
- API URL 写错
- Agent 架构错误
- 目标 VPS 无法访问 API
- 系统时间不正确

---

# 二十、位置和网络信息验证

测试 IPinfo：

```bash
curl -fsSL \
  https://ipinfo.io/widget/demo/121.8.215.106 \
  | jq '.data | {
      city,
      region,
      country,
      loc,
      asn,
      company,
      privacy
    }'
```

查询面板结果：

```bash
curl -fsS \
  https://api.example.com/api/v1/public/machines \
  | jq '.machines[] | {
      name,
      isp: .network.isp,
      city: .network.city,
      region: .network.region,
      countryCode: .network.countryCode,
      latitude: .network.latitude,
      longitude: .network.longitude,
      riskScore: .network.riskScore,
      riskSource: .network.riskSource,
      type: .network.type
    }'
```

---

# 二十一、历史 API

获取机器 ID：

```bash
MACHINE_ID="$(
  curl -fsS \
    https://api.example.com/api/v1/public/machines \
  | jq -r '.machines[0].id'
)"
```

## 1. 单台机器详情

```bash
curl -fsS \
  "https://api.example.com/api/v1/public/machines/${MACHINE_ID}" \
  | jq
```

## 2. 24 小时历史

```bash
curl -fsS \
  "https://api.example.com/api/v1/public/machines/${MACHINE_ID}/history" \
  | jq
```

返回：

```json
{
  "rangeHours": 24,
  "intervalSeconds": 60,
  "points": [
    {
      "time": 1785787200,
      "online": true,
      "cpuUsage": 10.2,
      "memoryUsage": 35.1,
      "diskUsage": 22.5,
      "networkDownloadBps": 1024000,
      "networkUploadBps": 512000
    }
  ]
}
```

## 3. 30 天 SLA

```bash
curl -fsS \
  "https://api.example.com/api/v1/public/machines/${MACHINE_ID}/sla" \
  | jq
```

返回：

```json
{
  "days": 30,
  "onlineMinutes": 43025,
  "offlineMinutes": 175,
  "observedMinutes": 43200,
  "unknownMinutes": 0,
  "sla": 99.5949
}
```

刚部署时历史记录较少是正常的。

---

# 二十二、数据库备份

创建目录：

```bash
mkdir -p /var/backups/vps-board
```

手动备份：

```bash
sqlite3 \
  /var/lib/vps-board/board.db \
  ".backup '/var/backups/vps-board/board-$(date +%F-%H%M%S).db'"
```

每天凌晨 3 点备份：

```bash
crontab -e
```

加入：

```cron
0 3 * * * sqlite3 /var/lib/vps-board/board.db ".backup '/var/backups/vps-board/board-$(date +\%F).db'"
30 3 * * * find /var/backups/vps-board -type f -name '*.db' -mtime +14 -delete
```

建议保留：

- SQLite 数据库备份
- `/etc/vps-board/backend.env`
- Caddyfile
- 每台 VPS 的 Agent 配置
- 管理员密钥
- IP 哈希密钥

---

# 二十三、更新后端

更新前先备份：

```bash
BACKUP_DIR="/root/vps-board-backup-$(date +%F-%H%M%S)"

mkdir -p "$BACKUP_DIR"

cp -a \
  /usr/local/bin/vps-board-server \
  "$BACKUP_DIR/"

sqlite3 \
  /var/lib/vps-board/board.db \
  ".backup '$BACKUP_DIR/board.db'"
```

编译新版本：

```bash
cd /opt/vps-board

gofmt -w \
  cmd/server/main.go \
  cmd/agent/main.go

go mod tidy

CGO_ENABLED=0 go build \
  -trimpath \
  -ldflags="-s -w" \
  -o vps-board-server-new \
  ./cmd/server
```

替换：

```bash
systemctl stop vps-board

install \
  -m 0755 \
  vps-board-server-new \
  /usr/local/bin/vps-board-server

systemctl start vps-board
```

检查：

```bash
systemctl is-active vps-board
```

```bash
curl -fsS \
  https://api.example.com/api/v1/health \
  | jq
```

---

# 二十四、更新 Agent

重新编译对应架构后上传。

目标 VPS 执行：

```bash
systemctl stop vps-board-agent

install \
  -m 0755 \
  /tmp/vps-board-agent \
  /usr/local/bin/vps-board-agent

systemctl start vps-board-agent
```

如果 Agent Token 没有被重置，原来的：

```text
/etc/vps-board-agent.env
```

不需要修改。

如果 Token 已经被重置，需要同步修改：

```ini
AGENT_TOKEN=新的Token
```

---

# 二十五、更新前端

```bash
cd /opt/vps-board/frontend
```

打包：

```bash
rm -f /tmp/vps-board-frontend-new.zip

zip -r \
  /tmp/vps-board-frontend-new.zip \
  index.html \
  detail.html \
  style.css \
  config.js \
  app.js \
  detail.js \
  _headers
```

检查：

```bash
unzip -l \
  /tmp/vps-board-frontend-new.zip
```

然后将压缩包上传到 Cloudflare Pages 的 Production 部署。

---

# 二十六、回滚

查看备份：

```bash
ls -d /root/vps-board-backup-*
```

停止服务：

```bash
systemctl stop vps-board
```

恢复后端程序：

```bash
cp -a \
  /root/vps-board-backup-时间目录/vps-board-server \
  /usr/local/bin/vps-board-server
```

恢复数据库：

```bash
cp -a \
  /root/vps-board-backup-时间目录/board.db \
  /var/lib/vps-board/board.db
```

修复权限：

```bash
chown \
  vps-board:vps-board \
  /var/lib/vps-board/board.db
```

启动：

```bash
systemctl start vps-board
```

检查：

```bash
systemctl status \
  vps-board \
  --no-pager
```

---

# 二十七、常见踩坑

## 1. `gofmt: command not found`

原因：

- Go 没加入 PATH
- 没创建 `gofmt` 软链接

解决：

```bash
export PATH=/usr/local/go/bin:$PATH

ln -sf \
  /usr/local/go/bin/gofmt \
  /usr/local/bin/gofmt
```

不要执行：

```bash
apt install gofmt
```

`gofmt` 是 Go 工具链的一部分。

---

## 2. `file: command not found`

安装：

```bash
apt install -y file
```

这只是缺少检查工具，不代表编译失败。

---

## 3. `curl -I` 返回 405

`curl -I` 会发送 HEAD 请求。

正确测试健康接口：

```bash
curl -i \
  https://api.example.com/api/v1/health
```

---

## 4. Caddy reload 失败

如果 Caddy 原本没有运行，使用：

```bash
systemctl restart caddy
```

而不是：

```bash
systemctl reload caddy
```

---

## 5. 443 端口被 Shadowsocks 占用

检查：

```bash
ss -lntup | grep -E ':(80|443)\b'
```

确认占用程序后停止对应服务：

```bash
systemctl disable --now shadowsocks-rust.service
```

然后：

```bash
systemctl restart caddy
```

---

## 6. `Unexpected token '<'`

错误：

```text
Unexpected token '<', "<!DOCTYPE "... is not valid JSON
```

原因通常是：

- `config.js` 地址错误
- API 请求到了 Pages 页面
- API 返回 HTML 而不是 JSON

检查：

```bash
curl -fsS \
  https://your-project.pages.dev/config.js
```

正确：

```js
window.VPS_BOARD_CONFIG = {
  API_BASE: "https://api.example.com"
};
```

---

## 7. 网页提示管理员密钥错误

浏览器可能保存了旧密钥。

在浏览器控制台执行：

```js
sessionStorage.removeItem("vps-admin-token");
location.reload();
```

然后重新输入管理员密钥。

---

## 8. 自定义前端域名报错

自定义域名会改变浏览器的 `Origin`。

例如默认域名：

```text
https://your-project.pages.dev
```

自定义域名：

```text
https://panel.example.com
```

必须配置：

```ini
ALLOWED_ORIGINS=https://your-project.pages.dev,https://panel.example.com
```

然后：

```bash
systemctl restart vps-board
```

---

## 9. CSS 没加载

检查：

```bash
curl -sSI \
  https://your-project.pages.dev/style.css
```

必须返回：

```text
HTTP/2 200
content-type: text/css
```

检查内容：

```bash
curl -fsS \
  https://your-project.pages.dev/style.css \
  | head
```

正确开头：

```css
:root {
```

如果返回：

```html
<!DOCTYPE html>
```

说明 Pages 上传目录结构错误。

正确结构：

```text
index.html
style.css
config.js
app.js
```

错误结构：

```text
frontend/index.html
frontend/style.css
```

---

## 10. Agent 一直离线

检查：

```bash
systemctl status \
  vps-board-agent \
  --no-pager
```

查看日志：

```bash
journalctl \
  -u vps-board-agent \
  -n 100 \
  --no-pager
```

常见原因：

- Token 不完整
- 使用了其他机器的 Token
- Token 已被重置
- API URL 错误
- Agent 架构错误
- VPS 无法访问 API
- 系统时间不正确

---

## 11. ISP 或位置为空

测试 IPinfo：

```bash
curl -fsSL \
  https://ipinfo.io/widget/demo/121.8.215.106 \
  | jq '.data'
```

检查后端日志：

```bash
journalctl \
  -u vps-board \
  -n 100 \
  --no-pager
```

也可以在网页中：

```text
设置
→ 选择机器
→ 刷新网络信息
```

---

## 12. 上行和下行一直是 0

查看默认网卡：

```bash
ip route show default
```

查看接口统计：

```bash
cat /proc/net/dev
```

在 Agent 配置中指定：

```ini
NETWORK_INTERFACE=eth0
```

然后重启：

```bash
systemctl restart vps-board-agent
```

制造一些下载流量：

```bash
curl -o /dev/null \
  "https://speed.cloudflare.com/__down?bytes=20000000"
```

---

# 二十八、外部资源

## IPinfo

```text
https://ipinfo.io/widget/demo
```

用途：

- 城市
- 地区
- 国家
- 经纬度
- ISP
- ASN
- IDC / ISP
- VPN
- Proxy
- Tor
- Relay

注意：Widget Demo 接口可能存在频率限制、字段变化或使用策略变化。

## Leaflet

```text
https://unpkg.com/leaflet@1.9.4/
```

## CARTO 地图

```text
https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png
```

## Linux 图标

Devicon：

```text
https://cdn.jsdelivr.net/gh/devicons/devicon/
```

Simple Icons：

```text
https://cdn.simpleicons.org/
```

## 国旗

```text
https://flagcdn.com/{countryCode}.svg
```

例如：

```text
https://flagcdn.com/jp.svg
```

外部资源可能出现：

- CDN 不可用
- 网络延迟
- 地区访问限制
- 地址变化
- 第三方服务条款变化

如果希望减少第三方依赖，可以下载资源后放入项目中自行托管。

---

# 二十九、当前版本限制

1. IPinfo Widget Demo 接口可能存在未公开限制。
2. 地图位置是城市级近似位置，不是精确机房位置。
3. 地图依赖 Leaflet 和 CARTO 外部资源。
4. Linux 图标依赖 Devicon 或 Simple Icons CDN。
5. 国旗依赖 FlagCDN。
6. 趋势数据需要等待后端采样。
7. 新机器的 SLA 从 Agent 第一次成功上报后开始计算。
8. Agent 每 5 秒上报，但历史数据每分钟保存一次。
9. 性能趋势保留 24 小时。
10. SLA 在线数据保留 30 天。
11. SQLite 适合十几台 VPS，不适合大规模多租户。
12. 每台 VPS 需要单独安装 Agent。
13. 每台 VPS 必须使用独立 Agent Token。
14. 页面不显示真实 IP，但后端和 IPinfo 查询过程仍会临时处理来源 IP。
15. 当前风险值是基于 IPinfo 字段计算的项目自定义值，不是 Cloudflare 官方风险评分。
16. 后端 API 开启 Cloudflare 代理前，需要额外实现可信的 Cloudflare 来源 IP 处理。

---


# 三十、快速验证命令

## 后端健康

```bash
curl -fsS \
  https://api.example.com/api/v1/health \
  | jq
```

## 机器列表

```bash
curl -fsS \
  https://api.example.com/api/v1/public/machines \
  | jq
```

## 总机器数和在线数量

```bash
curl -fsS \
  https://api.example.com/api/v1/public/machines \
  | jq '{
      total: (.machines | length),
      online: (
        [.machines[] | select(.status == "online")]
        | length
      ),
      offline: (
        [.machines[] | select(.status != "online")]
        | length
      )
    }'
```

## 机器详情

```bash
curl -fsS \
  "https://api.example.com/api/v1/public/machines/机器ID" \
  | jq
```

## 24 小时历史

```bash
curl -fsS \
  "https://api.example.com/api/v1/public/machines/机器ID/history" \
  | jq
```

## 30 天 SLA

```bash
curl -fsS \
  "https://api.example.com/api/v1/public/machines/机器ID/sla" \
  | jq
```

## 后端服务

```bash
systemctl status \
  vps-board \
  --no-pager
```

## Agent 服务

```bash
systemctl status \
  vps-board-agent \
  --no-pager
```

## 后端日志

```bash
journalctl \
  -u vps-board \
  -n 100 \
  --no-pager
```

## Agent 日志

```bash
journalctl \
  -u vps-board-agent \
  -n 100 \
  --no-pager
```

---

# 三十一、最终检查清单

- [ ] 项目目录统一为 `vps-board`
- [ ] 服务器源码目录为 `/opt/vps-board`
- [ ] 数据库目录为 `/var/lib/vps-board`
- [ ] 后端配置目录为 `/etc/vps-board`
- [ ] Go 后端可以启动
- [ ] Caddy 正常运行
- [ ] API HTTPS 正常
- [ ] Cloudflare Pages 部署状态为 Production / Success
- [ ] `config.js` 指向真实 API 域名
- [ ] `ALLOWED_ORIGINS` 包含实际前端域名
- [ ] 首页可以访问
- [ ] 首页显示总数量
- [ ] 首页显示在线数量
- [ ] 世界地图正常加载
- [ ] 电脑端一行两个卡片
- [ ] 手机端一行一个卡片
- [ ] 详情页可以访问
- [ ] Agent 状态为 online
- [ ] CPU 数据正常
- [ ] 内存数据正常
- [ ] 硬盘数据正常
- [ ] 上行和下行数据正常
- [ ] ISP 可以识别
- [ ] 国家国旗正常显示
- [ ] IP 风险值正常显示
- [ ] 24 小时历史 API 正常
- [ ] 30 天 SLA API 正常
- [ ] SQLite 自动创建历史表
- [ ] 公共 API 不返回真实 IP
- [ ] GitHub 中没有密钥
- [ ] 已配置 SQLite 定时备份