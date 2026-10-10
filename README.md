# beszel-lzcapp

[Beszel](https://github.com/henrygd/beszel)（轻量服务器监控 hub）的懒猫微服打包：**只发喵喵商店，镜像模式**（`delivery.mode: mirror`）。

- **镜像**：`henrygd/beszel:0.21.0`（scratch 镜像，容器内只有 `/beszel` 一个静态二进制，监听 **8090**，数据在 `/beszel_data`）。镜像模式只做只读校验，交付时把 manifest 的镜像行改写成 `docker.1ms.run/henrygd/beszel:0.21.0`，不搬镜像到懒猫镜像源。
- **版本跟踪**：`channel: stable` + `sort: semver` + `tag_regex: '^[0-9]+\.[0-9]+\.[0-9]+$'`（排除 `0.21` 两段别名和 `latest`）。
- **健康检查**：镜像里没有 shell/curl/wget，但 `/beszel` 自带 `health` 子命令——`CMD ["/beszel", "health", "http://127.0.0.1:8090/api/health"]`（和 magpie 的 `CMD /magpie healthcheck` 一个路数）。
- **环境**：`APP_URL=https://{{ .S.AppDomain }}`（UI 链接、WebSocket、通知里的地址全靠它）。
- **路由**：`/` → `beszel:8090`，`public_path: [/]`（入口放行，应用自带账号：首次打开页面注册管理员）。
- **连接方向**：**hub 主动连 agent**（agent 监听 45876），所以微服侧只需要出网，不用开任何入站端口，被监控的机器也不用公网。
- **持久化**：`/lzcapp/var/beszel:/beszel_data`（SQLite、密钥与历史数据）。`user: root`（scratch 镜像默认 root，平台建的 var 目录属主也是 root）。
- **图标**：上游 `internal/site/public/static/icon.png`。

## 商店状态

- **喵喵商店**：`cloud.lazycat.app.beszel` 由 Action 发布（`stores.private`）。
- **官方商店**：镜像模式不满足官方商店「镜像必须来自 `registry.lazycat.cloud`」的要求，只发喵喵商店。
- 本仓库只打包 **hub**；被监控的机器需要自己装 `beszel-agent`（版本与 hub 匹配）。
