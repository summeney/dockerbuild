# Chromadb Docker 部署指南

本文档介绍如何使用仓库中的 `Dockerfile` 构建并运行一个在容器中托管 Chromadb 的 HTTP 服务。

## 先决条件
- 已安装 Docker（24.x 及以上版本更佳）
- 能访问互联网以下载 Python 基础镜像和 Chromadb 依赖
- 如果要持久化数据，需要准备一块宿主机磁盘或网络存储路径

## 构建镜像
```bash
docker build -t chromadb-server:latest .
```

关键点：
- 基础镜像为 `python:3.11-slim`
- 镜像内安装了 `chromadb[server]==0.5.3`
- 默认对外开放 8000 端口

## 运行容器
最简单的运行方式：
```bash
docker run --rm -p 8000:8000 chromadb-server:latest
```

生产环境推荐映射数据卷并调整资源限制：
```bash
docker run -d \
  --name chromadb \
  -p 8000:8000 \
  -v /srv/chroma-data:/data/chroma \
  -e CHROMA_SERVER_PORT=8000 \
  chromadb-server:latest
```

容器启动后可通过 `http://<宿主机IP>:8000/api/v1/heartbeat` 验证服务状态。

## 环境变量
| 变量 | 默认值 | 作用 |
| --- | --- | --- |
| `CHROMA_DB_PATH` | `/data/chroma` | 持久化存储路径 |
| `CHROMA_SERVER_HOST` | `0.0.0.0` | Chromadb 服务监听地址 |
| `CHROMA_SERVER_PORT` | `8000` | Chromadb 服务端口 |

## 升级与备份
- 构建新的镜像版本前，可修改 `Dockerfile` 中的 `chromadb` 版本号
- 数据均保存在 `/data/chroma`，只需备份对应宿主机目录即可完成快照

## 常见问题
1. **端口冲突**：确保宿主机 8000 端口未被占用，或在 `docker run -p <host_port>:8000` 中调整映射
2. **性能瓶颈**：可通过 `docker run` 指定 `--cpus`、`--memory` 等参数，或将数据目录挂载到更快的磁盘
3. **健康检查失败**：查看容器日志 `docker logs chromadb`，确认 HTTP 接口是否能访问 `/api/v1/heartbeat`

