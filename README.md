# gpt-image-canvas-docker

这个仓库用于给上游 [`mrslimslim/gpt-image-canvas`](https://github.com/mrslimslim/gpt-image-canvas) 做 Docker 包装发布，不复制上游源码，而是通过 GitHub Actions 每天拉取上游 `main` 分支并构建镜像，推送到 `ghcr.io`。

## 包含内容

- `docker-compose.yml`：直接部署 GHCR 镜像。
- `.env.example`：运行时环境变量示例。
- `.github/workflows/build-gpt-image-canvas.yml`：每日构建并推送多架构镜像。

## 镜像发布策略

工作流会从上游仓库直接检出源码，使用上游 `Dockerfile` 构建 `linux/amd64` 和 `linux/arm64` 镜像，并推送到：

```text
ghcr.io/<github-owner>/gpt-image-canvas
```

默认会生成这些标签：

- `latest`
- 上游版本号，例如 `0.2.0`
- 日期标签，例如 `20260509`
- 提交标签，例如 `sha-abcdef1`

工作流触发方式：

- 每天北京时间 `09:00`（UTC `01:00`）
- 手动触发 `workflow_dispatch`

另外，工作流里显式覆盖了上游 Dockerfile 的构建参数，改用 Debian 和 npm 官方源，避免 GitHub Actions 访问中国镜像源时构建不稳定。

## 使用前准备

1. 把当前目录推送到你自己的 GitHub 仓库。
2. 确认仓库已启用 GitHub Actions。
3. 首次构建完成后，到 GHCR 把包可见性改成 `public`（如果你希望匿名拉取镜像）。

如果你的默认分支不是 `main`，需要调整 `.github/workflows/build-gpt-image-canvas.yml` 里的 `SOURCE_REF` 或工作流所在分支策略。

## 手动触发构建

仓库推送到 GitHub 后，可以在 Actions 页面手动运行 `Build and Publish gpt-image-canvas`。

发布完成后，镜像地址通常是：

```text
ghcr.io/<你的 GitHub 用户名>/gpt-image-canvas:latest
```

## 本地部署

1. 复制环境文件：

```powershell
Copy-Item .env.example .env
```

2. 编辑 `.env`，至少填入：

```env
IMAGE_NAME=ghcr.io/your-github-owner/gpt-image-canvas:latest
OPENAI_API_KEY=your_api_key
```

如果你使用官方 OpenAI 接口，`OPENAI_BASE_URL` 留空即可。

3. 拉取并启动：

```powershell
docker compose pull
docker compose up -d
```

4. 查看日志：

```powershell
docker compose logs -f app
```

默认访问地址：

```text
http://localhost:8787
```

运行数据会持久化到当前目录的 `./data`。

## docker-compose 说明

`docker-compose.yml` 做了这些运行时设置：

- 容器名固定为 `gpt-image-canvas`
- 端口默认映射 `8787:8787`
- 数据目录挂载为 `./data:/app/data`
- 默认启用：
  - `SQLITE_JOURNAL_MODE=DELETE`
  - `SQLITE_LOCKING_MODE=EXCLUSIVE`

这两个 SQLite 参数和上游 Compose 保持一致，能降低 Docker Desktop 绑定挂载场景下的 SQLite 共享内存问题。

## 升级

重新拉取最新镜像并重建容器即可：

```powershell
docker compose pull
docker compose up -d
```

## 参考

- 上游项目：<https://github.com/mrslimslim/gpt-image-canvas>
- 参考实现：<https://github.com/aiqinxuancai/nanobot-docker>
