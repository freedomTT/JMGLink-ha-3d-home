# HA 3D Home 加载项

此加载项运行与独立部署相同的 HA 3D Home 服务端和 React 构建产物。

## 发布

此加载项配置为使用预构建镜像：

```yaml
image: docker.ysf.cn/jmglink-ha-3d
```

安装加载项前，请先发布匹配版本的镜像标签。对于 `version: 0.1.0`，请发布：

```text
docker.ysf.cn/jmglink-ha-3d:0.1.0
```

这样可以避免在用户的 HAOS 主机上构建镜像，并防止因本地 Docker Hub 或 npm registry 超时导致安装失败。

## 仓库包

在源码仓库根目录运行以下命令，生成仅包含镜像加载项元数据的仓库包：

```bash
npm run package:addon
```

该命令只会将 Home Assistant 加载项元数据复制到 `dist-addon-release/`。运行时代码不会存放在加载项仓库中，而是只存在于预构建 Docker 镜像内。

预期打包结构如下：

```text
dist-addon-release/
  repository.yaml
  jmglink-ha-3d-home/
    config.yaml
    DOCS.md
    CHANGELOG.md
    translations/
```

发布此加载项仓库前，请先在源码仓库根目录使用 `Dockerfile.addon` 构建并推送预构建镜像。

推荐构建命令：

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -f Dockerfile.addon \
  --build-arg BUILD_VERSION=0.1.1 \
  -t docker.ysf.cn/jmglink-ha-3d:0.1.1 \
  -t docker.ysf.cn/jmglink-ha-3d:latest \
  --push .
```

## 运行时

加载项通过 `SUPERVISOR_TOKEN` 和 `homeassistant_api: true` 经由 Supervisor 调用 Home Assistant Core。前端仍只通过相对路径 `/api/*` 和 `/ws` 调用此服务，因此可以在 Ingress 后正常工作。

如果连接面板仍显示 “local development”，说明 Home Assistant 正在运行旧镜像或旧的加载项元数据。请刷新加载项仓库、升级或重新安装加载项，并确认加载项日志中包含 `Runtime configuration loaded`，且其中 `mode: "addon"` 与 `hasSupervisorToken: true`。