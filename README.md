# 智码软件官网

智码软件 / Smart Zhima 的静态官网。

## 当前线上信息

- 域名：`smartzhima.com`
- ICP 备案号：`陕ICP备2026012300号`
- 备案审核通过日期：`2026-05-22`
- 页脚备案号链接至：`https://beian.miit.gov.cn/`

## 本地文件

- `index.html`: 官网页面
- `assets/`: favicon、二维码等静态资源
- `Dockerfile`: Nginx 静态站镜像
- `nginx.conf`: 容器内 Nginx 配置
- `docker-compose.yml`: 服务器部署配置
- `nginx.https.conf`: HTTPS Nginx 配置，证书申请成功后使用
- `docker-compose.https.yml`: HTTPS 部署覆盖配置，证书申请成功后使用

## 首次推送到 Git

在本地创建远程仓库后，把仓库地址替换到下面命令里：

```bash
git remote add origin git@github.com:YOUR_NAME/YOUR_REPO.git
git branch -M main
git push -u origin main
```

如果使用 Gitee、GitLab 或 Coding，把远程地址替换成对应平台提供的 SSH/HTTPS 地址即可。

## 服务器首次部署

以下命令假设服务器是 Ubuntu，项目放在 `/opt/smartzhima`。

```bash
# 安装 Docker
curl -fsSL https://get.docker.com | sh

# 拉代码
mkdir -p /opt
cd /opt
git clone git@github.com:YOUR_NAME/YOUR_REPO.git smartzhima
cd /opt/smartzhima

# 启动网站
docker compose up -d --build

# 查看状态
docker compose ps
```

如果服务器还没有配置 Git SSH key，也可以先用 HTTPS 地址：

```bash
git clone https://github.com/YOUR_NAME/YOUR_REPO.git smartzhima
```

## 后续更新部署

本地修改后：

```bash
git add .
git commit -m "Update website"
git push
```

服务器更新：

```bash
cd /opt/smartzhima
git pull
docker compose up -d --build
```

如果已经启用 HTTPS，用下面命令更新：

```bash
cd /opt/smartzhima
git pull
docker compose -f docker-compose.yml -f docker-compose.https.yml up -d --build
```

## 服务器端口

腾讯云安全组需要放行：

- `22`: SSH
- `80`: HTTP
- `443`: HTTPS，后续配置证书时使用

如果服务器启用了系统防火墙，也要放行 `80` 和 `443`。Ubuntu 常见是 `ufw`：

```bash
sudo ufw status
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

检查端口是否正在监听：

```bash
sudo ss -lntp | grep -E ':80|:443'
```

默认 Docker 配置只监听 `80`。完成域名解析后，可以先访问：

- `http://smartzhima.com`
- `http://www.smartzhima.com`

## 免费 Wildcard HTTPS 证书

Wildcard 证书需要 DNS 验证。证书需要同时包含根域名和通配符域名：

- `smartzhima.com`
- `*.smartzhima.com`

下面用 `acme.sh + 腾讯云 DNSPod API` 申请 Let's Encrypt 免费证书。先在腾讯云创建 API 密钥，然后在服务器执行：

```bash
# 安装 acme.sh
curl https://get.acme.sh | sh -s email=你的邮箱
source ~/.bashrc

# 避免 acme.sh 默认使用 ZeroSSL
~/.acme.sh/acme.sh --set-default-ca --server letsencrypt

# 腾讯云/DNSPod API 密钥
export Tencent_SecretId="你的 SecretId"
export Tencent_SecretKey="你的 SecretKey"

# 申请 wildcard 证书
~/.acme.sh/acme.sh --issue \
  --server letsencrypt \
  --dns dns_tencent \
  --dnssleep 120 \
  -d smartzhima.com \
  -d '*.smartzhima.com'

# 安装证书到固定目录
sudo mkdir -p /opt/certs/smartzhima
sudo chown -R "$USER":"$USER" /opt/certs/smartzhima

~/.acme.sh/acme.sh --install-cert -d smartzhima.com --ecc \
  --key-file /opt/certs/smartzhima/privkey.pem \
  --fullchain-file /opt/certs/smartzhima/fullchain.pem \
  --reloadcmd "docker exec smartzhima-web nginx -s reload"
```

证书安装成功后，启用 HTTPS 配置：

```bash
cd /opt/smartzhima
docker compose -f docker-compose.yml -f docker-compose.https.yml up -d --build
```

检查：

```bash
curl -I https://smartzhima.com
curl -I https://www.smartzhima.com
```

续期由 `acme.sh` 的定时任务自动处理。可以用下面命令查看：

```bash
crontab -l
```
