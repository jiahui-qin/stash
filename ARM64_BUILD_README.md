# Stash ARM64 (RK3568) Docker Build

本配置用于构建支持 ROCKCHIP RK3568 的 Stash Docker 镜像。

## 文件说明

| 文件 | 说明 |
|------|------|
| `.github/workflows/build-docker-arm64.yml` | GitHub Actions 工作流 |
| `Dockerfile.arm64` | ARM64 多阶段构建 Dockerfile |
| `docker-compose.arm64.yml` | RK3568 设备使用的 docker-compose |

## 在 GitHub 上启用

1. **Fork 仓库** 或将文件推送到你的仓库

2. **确保 GitHub Packages 访问权限**
   - 进入仓库 Settings → Actions → General
   - 确保 "Read and write permissions" 已勾选

3. **触发构建**
   - 推送到 main 分支自动构建
   - 创建 Tag 自动构建: `git tag v1.0.0 && git push --tags`
   - 手动触发: Actions → Build and Publish ARM64 Docker Image → Run workflow

## 使用镜像

### Docker Hub / GHCR

```bash
# 登录到 GitHub Container Registry
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin

# 拉取镜像
docker pull ghcr.io/jiahui-qin/stash:latest

# 运行
docker run -d \
  --name stash \
  -p 9999:9999 \
  -v /path/to/config:/root/.stash \
  -v /path/to/data:/data \
  ghcr.io/jiahui-qin/stash:latest
```

### 使用 docker-compose

```bash
# 编辑 docker-compose.arm64.yml 中的镜像地址为你的仓库
# 然后运行
docker-compose -f docker-compose.arm64.yml up -d
```

## RK3568 部署说明

RK3568 是一款 ARM64 芯片，广泛用于 NAS 和 SBC 设备：

- 绿联 DX4600 Pro / DX4600
- 飞牛私有云 NAS
- OrangePi 5 / 5B
- 贝壳物联 / 其他 RK3568 设备

### 性能注意事项

1. **内存**: 推荐至少 4GB RAM
2. **存储**: 建议使用 SSD 存储 metadata 和 cache
3. **FFmpeg**: RK3568 支持硬件编解码，容器内 FFmpeg 可利用

### GPU 加速 (可选)

如果你的 RK3568 设备支持 GPU 加速，修改 Dockerfile:

```dockerfile
# 在 Final Image 中添加
RUN apk add --no-cache mesa-dri-gallium mesa-gbm
# 并在运行时添加设备映射
# devices:
#   - /dev/dri:/dev/dri
```

## 镜像标签说明

| 标签 | 说明 |
|------|------|
| `latest` | 最新构建版本 |
| `vX.Y.Z` | 对应 Git Tag 版本 |
| `arm64-latest` | ARM64 最新版本 |

## 本地构建 (可选)

如果需要在本地构建 ARM64 镜像 (需要 Docker Buildx):

```bash
# 启用 buildx
docker buildx create --name mybuilder
docker buildx use mybuilder

# 构建
docker buildx build --platform linux/arm64 -t stash:arm64 -f Dockerfile.arm64 .

# 或者使用 Makefile (需要在 ARM64 机器上)
make build-cc-linux-arm64v8
```
