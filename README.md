# 智码软件官网

智码软件 / Smart Zhima 的静态官网。

## 本地文件

- `index.html`: 官网页面
- `assets/`: favicon、二维码等静态资源
- `Dockerfile`: Nginx 静态站镜像
- `nginx.conf`: 容器内 Nginx 配置
- `docker-compose.yml`: 服务器部署配置

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

## 服务器端口

腾讯云安全组需要放行：

- `22`: SSH
- `80`: HTTP
- `443`: HTTPS，后续配置证书时使用

当前 Docker 配置只监听 `80`。备案通过并完成域名解析后，可以先访问：

- `http://smartzhima.com`
- `http://www.smartzhima.com`

HTTPS 证书建议后续用宿主机 Nginx / Caddy 统一处理，再反向代理到当前容器。
