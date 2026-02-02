# dvm

用目录来管理 Dockerfile，并通过 GitHub Actions 自动构建并推送到 Docker Hub。
如果手工编译, 使用: `docker build -t aidev -f images/aidev/Dockerfile images/aidev/`

## 目录规范
- 一个镜像 = 一个目录
- 目录内必须包含 `Dockerfile`
- 同目录放需要 `COPY` 的配置文件

当前结构示例：
```
.
├── images
│   └── aidev
│       ├── Dockerfile
│       ├── .bashrcx
│       ├── .tmux.conf
│       └── base.Brewfile
└── .github/workflows/docker.yml
```

## 镜像列表
- `images/aidev`
  - 基于 `ubuntu:24.04`
  - 系统级工具用 Homebrew 安装（base）
  - 项目级工具建议用 `mise`
  - 内置：uv/uvx、node/npm/npx/pnpm（优先 node@24，缺失则回退到 node）、bun/bunx、rustup
  - 复制 `.bashrcx`、`.tmux.conf`

## GitHub Actions 构建与推送
工作流会自动扫描仓库内所有 `Dockerfile`，为每个目录构建并推送镜像。

### 需要配置的 Secrets
在 GitHub 仓库中添加：
- `DOCKERHUB_USERNAME`
- `DOCKERHUB_TOKEN`

### 镜像命名规则
```
<DOCKERHUB_USERNAME>/dvm-<目录名>
```
例如：
```
mr00325ml/dvm-aidev:latest
```

### 触发条件
- push 到 `main` 分支
- 手动触发 `workflow_dispatch`

## 本地构建
```
docker build -t dvm-aidev ./images/aidev
```

## 运行示例
```
docker run \
  --detach-keys="ctrl-@" \
  --name unlimit-ai \
  -v /Users/mr00325ml:/Users/mr00325ml \
  -e LANG=en_US.UTF-8 \
  -e LC_ALL=en_US.UTF-8 \
  -t -i \
  dvm-aidev \
  bash

docker start --detach-keys="ctrl-@" -ai unlimit-ai
```

## 挂载建议（可选）
如果需要在容器中使用本机的 AI 配置，可以挂载：
```
/Users/mr00325ml/Docker/.claude
/Users/mr00325ml/Docker/.codex
/Users/mr00325ml/Docker/.claude.json
```

示例（按需增加）：
```
docker run \
  --detach-keys="ctrl-@" \
  --name unlimit-ai \
  -v /Users/mr00325ml:/Users/mr00325ml \
  -v /Users/mr00325ml/Docker/.claude:/root/.claude \
  -v /Users/mr00325ml/Docker/.codex:/root/.codex \
  -v /Users/mr00325ml/Docker/.claude.json:/root/.claude.json \
  -e LANG=en_US.UTF-8 \
  -e LC_ALL=en_US.UTF-8 \
  -t -i \
  dvm-aidev \
  bash
```

## 约定与说明
- 系统级工具：使用 Homebrew
  - base：常用、稳定、不常更新（`base.Brewfile`）
- 项目级工具：使用 `mise`（不同项目不同版本）
- AI 相关 CLI：已在镜像中安装（如 `claude`、`@openai/codex`）
- 容器是可随意破坏的环境，不挂载额外目录时损坏可直接重建

## 添加新镜像
1. 新建目录：`images/<name>/`
2. 放入 `Dockerfile` 与配置文件
3. 推送到 `main`，GitHub Actions 会自动构建并推送

## 常用配置文件
### `.bashrcx`
已内置基础按键、历史搜索、git alias 与 locale 配置。

### `.tmux.conf`
已内置快捷键、窗口操作与基础主题设置。

## TODO
- 注入更多常用配置（如 SSH、Git 相关）
