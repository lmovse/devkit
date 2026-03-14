# DevKit

开发工具集

## 镜像

### 快速开始

本项目包含各种开发环境镜像，定义在 `images/` 目录下。

#### 版本管理

每个镜像的版本号记录在 `images/version.json` 中：

```json
{
  "sandbox": "v1.0.0"
}
```

当版本号发生变更时，GitHub Actions 会自动构建并推送到 ghcr.io。

#### 可用镜像

- **sandbox**: 通用开发环境

#### 手动构建

```bash
# 构建特定镜像
docker build -t my-image:latest ./images/sandbox

# 多平台构建
docker buildx build --platform linux/amd64,linux/arm64 -t my-image:latest ./images/sandbox
```

## License

MIT License - see [LICENSE](LICENSE) file for details.
