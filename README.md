# JMGLink HA 3D Home 安装使用教程

JMGLink HA 3D Home 是 Home Assistant 的 3D 户型图可视化控制中心，安装后可通过侧边栏「3D Home」面板查看房间布局、设备状态并直接控制。

<br />

现在AI越来越厉害了，大家可以用AI自己来做这种应用，根据自己家的情况可以做的还原度更高，我这个项目只是个通用的工具，目的是为了展示设备状态，不是为了还原家里家具的布局。(当然需要用聪明一点的大模型哈)

***

## 一、安装加载项

1. 打开 Home Assistant，进入 **设置 → 加载项 → 加载项商店**
2. 点击右上角菜单，选择「仓库」，添加 JMGLink 加载项仓库地址
3. 刷新加载项商店，找到「JMGLink HA 3D Home」并点击安装
4. 安装完成后启动加载项，建议开启「在侧边栏显示」方便快速访问

***

## 二、启动与访问

启动加载项后，直接点击左侧侧边栏的「3D Home」即可进入。

如果侧边栏未显示面板：

1. 确认加载项正常运行
2. 在加载项详情页验证 Ingress 页面可正常打开
3. 刷新浏览器/重新登录 HA，仍无法显示则重启 HA 前端或重装加载项

***

## 三、快速配置流程

1. 打开 3D Home 面板，进入户型编辑器
2. 创建/导入户型，设置楼层、墙体、房间结构
3. 添加家具、门窗和设备点位，将点位绑定 HA 实体
4. 保存项目后返回 3D 视图即可查看和控制设备

***

## 四、常用配置说明

### 核心配置项

| 配置项               | 默认值     | 说明                         |
| ----------------- | ------- | -------------------------- |
| `log_level`       | `info`  | 排查问题时可临时改为 `debug` 输出详细日志  |
| `external_access` | `false` | 普通Ingress场景保持默认即可，需外部访问时开启 |

修改配置后需保存并重启加载项生效。

### 数据备份

加载项自动保存项目数据到 HA 可写目录，大幅修改户型前建议手动导出备份项目文件。

***

## 五、Docker 独立版本部署

Docker 独立版本适用于 NAS、Linux 小主机、Windows Docker Desktop 等非 HAOS Add-on 环境。该模式不会自动出现在 Home Assistant 侧边栏，需要通过浏览器访问独立服务地址。

### 适用场景

- 不使用 Home Assistant OS / Supervisor Add-on
- 希望在独立 Docker 主机上运行 3D Home
- Home Assistant 与 3D Home 分别部署，但网络可互相访问

### 准备工作

1. 安装 Docker 或 Docker Compose
2. 确认 Docker 主机可以访问 Home Assistant 地址，例如 `http://homeassistant.local:8123` 或 `http://HA主机IP:8123`
3. 在 Home Assistant 中进入 **用户头像 → 安全 → 长期访问令牌**，创建一个长期访问令牌备用

### 使用 Docker Compose 启动（推荐）

创建 `docker-compose.yml`：

```yaml
services:
  jmglink-ha-3d-home:
    image: docker.jmglink.cn/jmglink-ha-3d:latest
    container_name: jmglink-ha-3d-home
    restart: unless-stopped
    ports:
      - "8099:8099"
    volumes:
      - ./data:/data
      - ./assets:/assets
    environment:
      HA3D_MODE: standalone
      PORT: 8099
      HA3D_DATA_DIR: /data
      HA3D_ASSET_DIR: /assets
```

然后执行：

```bash
# 查看日志
docker logs -f jmglink-ha-3d-home

# 重启服务
docker restart jmglink-ha-3d-home

# 更新镜像并重建容器
docker compose pull
docker compose up -d
```

### 使用 docker run 启动

Linux / macOS 示例：

```bash
mkdir -p data assets

docker run -d \
  --name jmglink-ha-3d-home \
  --restart unless-stopped \
  -p 8099:8099 \
  -e HA3D_MODE=standalone \
  -e PORT=8099 \
  -e HA3D_DATA_DIR=/data \
  -e HA3D_ASSET_DIR=/assets \
  -v "$PWD/data:/data" \
  -v "$PWD/assets:/assets" \
  docker.jmglink.cn/jmglink-ha-3d:latest
```

Windows PowerShell 示例：

```powershell
mkdir data, assets -Force

docker run -d `
  --name jmglink-ha-3d-home `
  --restart unless-stopped `
  -p 8099:8099 `
  -e HA3D_MODE=standalone `
  -e PORT=8099 `
  -e HA3D_DATA_DIR=/data `
  -e HA3D_ASSET_DIR=/assets `
  -v ${PWD}/data:/data `
  -v ${PWD}/assets:/assets `
  docker.jmglink.cn/jmglink-ha-3d:latest
```

启动后访问：

```text
mkdir -p data assets

docker run -d \
  --name jmglink-ha-3d-home \
  --restart unless-stopped \
  -p 8099:8099 \
  -e HA3D_MODE=standalone \
  -e PORT=8099 \
  -e HA3D_DATA_DIR=/data \
  -e HA3D_ASSET_DIR=/assets \
  -v "$PWD/data:/data" \
  -v "$PWD/assets:/assets" \
  docker.jmglink.cn/jmglink-ha-3d:latest
```

首次打开后，在页面的「连接与服务」面板填写 Home Assistant 地址和 API Key。配置会保存到挂载的 `data` 目录中，容器重启后会自动读取。

### 常用维护命令

```bash
# 查看日志
docker logs -f jmglink-ha-3d-home

# 重启服务
docker restart jmglink-ha-3d-home

# 更新镜像并重建容器
docker compose pull
docker compose up -d
```

### 独立版本注意事项

1. Docker 独立版本不依赖 Home Assistant Supervisor，也不会使用 Add-on Ingress
2. 浏览器访问地址是 `http://Docker主机IP:8099`，不是 Home Assistant 侧边栏地址
3. 如容器无法连接 Home Assistant，请优先检查 HA 地址是否能从 Docker 主机访问、长期访问令牌是否正确、HA 防火墙/反向代理是否放行
4. 数据目录 `./data` 和素材目录 `./assets` 请勿随意删除，否则可能导致连接配置、项目和素材丢失

***

## 六、版本更新内容

### 0.1.5

- 对接HA语音助手测试版，还没决定加不加tts、stt，先试试浏览器的api。

### 0.1.4

- 左上数据支持手动选择。
- 数据卡片支持名字自定义。
- 部分模型样式调整。
- 场景交互调整。

### 0.1.3

版本号写错了，跳过了

### 0.1.2

- 窗帘、风扇等模型优化
- 增加天气预报功能，对接ha天气数据
- 增加自定义数据卡片功能
- 修正门开向与 3D 渲染视图不一致。
- 支持一个开关建多个灯模
- 设备模型支持缩放
- 支持彩色灯

### 0.1.1

- 新增 Docker 独立部署版
- 支持者头像展示
- 增加卡通风和奶油风渲染模式
- 开关模型中增加灯具，无智能灯具解决方案
- 去掉编辑视图中半墙展示

### 0.1.0

- 完成 HA 3D Home 初始插件脚手架搭建
- 实现由 Fastify 后端提供服务、支持 Ingress 的 React 用户界面
- 支持通过 `SUPERVISOR_TOKEN` 调用 Home Assistant Supervisor API

***

## 七、常见问题

1. **页面显示「local development」**：刷新加载项仓库，升级/重装加载项，重启后查看日志确认 `mode: "addon"`、`hasSupervisorToken: true`
2. **无法打开页面**：检查加载项运行状态、端口是否占用、Ingress是否正常，清理浏览器缓存
3. **看不到HA实体**：确认加载项已启用 `homeassistant_api` 权限，HA核心运行正常，日志无API调用失败
4. **控制设备无反应**：检查实体绑定是否正确、实体在HA中是否可用，日志无动作调用失败

***

## JMGLink 官网 【作者主营产品，需要大家支持\~】

JMGLink 可用于远程访问 Home Assistant APP 以及其他内网应用，欢迎试用。

官网：
jmglink.cn
