# lutinlens_server 进度与部署

> LutinLens 的 AI 服务端（基于 NVIDIA NeMo Agent Toolkit）。App 仓库见 `https://github.com/HSDX2/LutinLens.git`（那里的 `PROGRESS.md` 有完整项目进度）。

## 已完成的改动

AI 已从「阿里云百炼 / 通义千问」切换为 **DeepSeek**：

- `base_url`：`https://api.deepseek.com`
- 文本模型：`deepseek-v4-flash`（ReAct Agent + LUT 选择）
- 视觉模型：`deepseek-v4-flash-vision-exp`（图片内容/亮度识别 + 多图构图建议）
- 环境变量：`DEEPSEEK_API_KEY`（原来是 `DASHSCOPE_API_KEY`）

改动文件：
- `agents/lut_advisor.yml`
- `tools/content_identifier/src/content_identifier/content_identifier_function.py`
- `tools/lut_finder/src/lut_finder/lut_finder_function.py`
- `tools/framing_advisor/src/framing_advisor/framing_advisor_function.py`
- `tools/content_identifier/example_config.yaml`

## 三个工作流（对应 App 的三个请求）

1. `lut_advisor` —— LUT 推荐（文本 `deepseek-v4-flash` + 视觉 `deepseek-v4-flash-vision-exp`）
2. `framing_advisor` —— 构图建议（**多图**，务必用 `deepseek-v4-flash-vision-exp`）
3. `s3` —— 图床上传（配 MinIO 或 S3 对象存储）

## 部署步骤

环境：Python 3.11~3.12 + NeMo Agent Toolkit。**不需要 GPU**（走 DeepSeek 云端 API）。Windows 需 WSL2 或 Docker。

```bash
git clone https://github.com/HSDX2/lutinlens_server.git
cd lutinlens_server
# 安装 NeMo Agent Toolkit（见根 README）
pip install ./tools/*
export DEEPSEEK_API_KEY="sk-你的key"
nat serve --workflow lut_advisor          # LUT 推荐
nat serve --workflow framing_advisor       # 构图建议
# 图床：配 s3/s3.yml 后单独起 s3 工作流
```

## 待办

- 部署并跑通三个服务，拿到实际端口
- App 设置里填上三个服务地址后联调
- 重点实测 `framing_advisor` 多图 + 严格 JSON 输出的稳定性
- S3 图床联调