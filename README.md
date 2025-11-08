# Clash Converter

一个功能完善的订阅转换系统，支持多种代理协议格式的转换。

## 项目简介

本项目提供了一套完整的订阅转换解决方案，包含前端界面和后端转换服务。用户可以通过 Web 界面或 API 接口将各种代理订阅格式相互转换，支持 Clash、Surge、Quantumult X、Loon、V2Ray、Shadowsocks 等主流代理客户端。

## 主要特性

- **多格式支持**: Clash, Surge, Quantumult X, Loon, V2Ray, SS/SSR, Singbox 等
- **订阅合并**: 支持合并多个订阅源
- **规则定制**: 内置 ACL4SSR 规则集，支持自定义配置
- **Docker 部署**: 一键部署，开箱即用
- **前后端分离**: Vue.js 前端 + C++ 高性能后端
- **模板系统**: 灵活的配置模板，适配不同客户端需求

## 快速开始

### 使用 Docker Compose (推荐)

```bash
# 克隆仓库及子模块
git clone --recursive https://github.com/yourusername/clash-converter.git
cd clash-converter

# 一键启动
docker-compose up -d --build

# 访问服务
# 前端界面: http://localhost:8080
# 后端 API: http://localhost:25500
```

详细的 Docker 部署说明请参考 [DEPLOY.md](DEPLOY.md)

### 手动部署

#### 前端 (sub-web)

```bash
cd sub-web

# 安装依赖
yarn install

# 开发模式
yarn serve

# 生产构建
yarn build
```

#### 后端 (subconverter)

```bash
cd subconverter

# 编译
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
cmake --build .

# 运行
./subconverter
```

## 项目架构

### 组件说明

- **sub-web**: Vue.js 前端界面 (端口 8080)
  - 提供用户友好的订阅转换界面
  - 基础模式和进阶模式两种操作方式

- **subconverter**: C++ 后端服务 (端口 25500)
  - 高性能订阅格式转换引擎
  - 支持多种代理协议的解析和生成

- **ACL4SSR**: 配置规则仓库
  - 预配置的分流规则集
  - 支持多种场景的规则模板

### 技术栈

- **前端**: Vue 2.6 + Element UI + Webpack
- **后端**: C++20 + CMake + httplib
- **部署**: Docker + Docker Compose

## API 使用

### 基本格式

```
http://127.0.0.1:25500/sub?target=<TARGET>&url=<URL>&config=<CONFIG>
```

### 参数说明

- `target`: 目标格式 (clash, surge, quanx, loon, v2ray, ss, ssr, singbox 等)
- `url`: URL 编码的订阅链接，多个订阅使用 `|` 分隔
- `config`: (可选) 外部配置文件路径或 URL

### 使用示例

```bash
# 转换为 Clash 格式
curl "http://127.0.0.1:25500/sub?target=clash&url=https%3A%2F%2Fexample.com%2Fsub"

# 合并多个订阅
curl "http://127.0.0.1:25500/sub?target=clash&url=订阅1|订阅2"

# 使用自定义配置
curl "http://127.0.0.1:25500/sub?target=clash&url=订阅链接&config=配置URL"
```

## 配置说明

### 后端配置 (subconverter/base/pref.ini)

首次使用需要从示例配置创建：

```bash
cd subconverter/base
cp pref.example.ini pref.ini
```

关键配置项：

- `api_mode`: 生产环境设为 `true`
- `api_access_token`: 修改默认密码
- `default_external_config`: 默认外部配置
- `exclude_remarks`: 过滤节点的正则表达式

### 前端配置 (sub-web/.env)

- `VUE_APP_SUBCONVERTER_DEFAULT_BACKEND`: 后端 API 地址
- `VUE_APP_MYURLS_API`: 短链接服务
- `VUE_APP_CONFIG_UPLOAD_API`: 配置文件托管服务

## 常用命令

```bash
# Docker 操作
docker-compose up -d          # 启动服务
docker-compose down           # 停止服务
docker-compose logs -f        # 查看日志
docker-compose ps             # 查看状态

# 子模块更新
git submodule update --remote --recursive

# 版本检查
curl http://localhost:25500/version
```

## 生产环境部署建议

1. 修改 `subconverter/base/pref.ini` 中的 `api_access_token`
2. 设置 `api_mode=true` 启用 API 模式
3. 使用 Nginx/Caddy 配置 HTTPS 反向代理
4. 配置防火墙规则限制访问
5. 定期备份配置文件和规则

## 文档

- [Docker 部署指南](DEPLOY.md) - 详细的 Docker Compose 使用说明
- [开发者文档](CLAUDE.md) - 项目架构和开发指南

## 许可证

本项目采用 [MIT License](LICENSE)

## 致谢

本项目使用了以下开源项目：

- [sub-web](https://github.com/duzefu/sub-web) - 前端界面
- [subconverter](https://github.com/duzefu/subconverter) - 后端转换引擎
- [ACL4SSR](https://github.com/duzefu/ACL4SSR) - 规则配置

## 问题反馈

如遇到问题，请提交 Issue 或 Pull Request。
