# 服务端 Windows(WSL2) 部署详细步骤

> 说明：以下基于官方文档整理，**尚未在真实 Windows 环境跑通验证**，遇到问题请按报错调整。

## 0. 前提

- Windows 10 **21H2 及以上**（否则 WSL2 不便）；Windows 11 最稳（NAT 官方验证过的是 Win11 + Ubuntu 24.04）。
- **不需要 GPU**——服务端走 DeepSeek 云端 API，本地仅转发与编解码。

## 1. 启用 WSL2 + 装 Ubuntu

管理员 PowerShell：

```powershell
wsl --install            # 默认装 Ubuntu，会要求重启
wsl --set-default-version 2
```

重启后执行 `wsl` 进入 Ubuntu，首次会要求创建用户名/密码。

## 2. 装依赖（Ubuntu 内）

```bash
sudo apt update
sudo apt install -y python3.12 python3.12-venv git curl
# 可选：装 uv（官方推荐，依赖管理）
curl -LsSf https://astral.sh/uv/install.sh | sh
```

> Python 版本必须在 **3.11~3.13** 之间（NAT 要求）。

## 3. 拉代码 + 建虚拟环境

```bash
git clone https://github.com/HSDX2/lutinlens_server.git
cd lutinlens_server
python3.12 -m venv .venv
source .venv/bin/activate
```

## 4. 安装 NeMo Agent Toolkit + 本地工具

```bash
pip install --upgrade pip
pip install nvidia-nat        # NAT 官方核心包
pip install ./tools/*         # 装 4 个本地工具(content_identifier/lut_finder/framing_advisor/b64_encoder)
```

> 若 `pip install nvidia-nat` 遇到问题，可用仓库内嵌源码安装：`pip install -e .`（仓库根目录就是 NAT 源码）。

## 5. 配置 DeepSeek

```bash
export DEEPSEEK_API_KEY="sk-你的key"
# 建议写入 ~/.bashrc 持久化：
echo 'export DEEPSEEK_API_KEY="sk-你的key"' >> ~/.bashrc
```

## 6. 启动三个服务（各开一个终端）

```bash
nat serve --workflow lut_advisor        # LUT 推荐（默认 8000）
nat serve --workflow framing_advisor     # 构图建议（默认 8001）
nat serve --workflow s3                  # 图床（配 s3/s3.yml）
```

## 7. 图床 MinIO（可选，若不用 s3 工作流可跳过）

lut_advisor 需要「图片 URL」，一般要一个图床。用 Docker 起 MinIO 最简单：

```bash
docker run -d --name minio -p 9000:9000 -p 9001:9001 \
  -e MINIO_ROOT_USER=minioadmin -e MINIO_ROOT_PASSWORD=minioadmin \
  minio/minio server /data --console-address ":9001"
```

然后核对 `s3/s3.yml` 的 endpoint/access_key/secret_key/bucket。

## 8. 让手机连到服务端

- **同一 Wi-Fi**：App 设置里填 `http://<电脑局域网IP>:<端口>`。
- 三个地址分别填：图床上传、LUT(约8000)、取景(约8001)。
- 跨网访问需要内网穿透/公网 IP/端口映射。

## 常见问题

- `nat` 命令找不到 → 确认已 `source .venv/bin/activate`，`which nat`。
- NAT 装不上 → Python 版本必须是 3.11~3.13；Windows 下绝不能原生跑，必须 WSL2。
- 手机连不上 → 检查 Windows 防火墙放行端口；确认端口与 App 设置一致。