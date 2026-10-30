# Hachile API

基于 [New API](https://github.com/QuantumNous/new-api) 的自用模型网关部署配置，计划使用 https://api.hachile.org 。2026-10-31 用户截图确认 OpenResty 已启动、API 编排成功启动；健康检查、域名 HTTPS 和真实上游调用仍待验证。

使用官方预构建镜像 `calciumion/new-api:v1.0.0-rc.47`，版本名称带有 rc 标记。部署采用单实例与 SQLite，数据保存在 Docker 卷 `hachile-api-data`，适合先进行小规模自用测试。

## 当前资源判断

2026-10-30 的 1Panel 截图显示：2 核 CPU，系统可见内存 1.83GB，已用 1.04GB；磁盘总量 39.01GB，已用 10.52GB。概览显示网站、数据库和已安装应用均为 0，但尚未检查容器列表。内存读数可能包含缓存，不能据此确定 Linux 的实际 available 内存。

应用内存硬上限为 512MB，Go 软内存目标为 384MiB，CPU 上限为 1 核。这些限制用于控制资源占用，不代表已验证的容量；超过限制可能导致应用被终止。请先测试单人文本调用，观察内存和重启次数。

## 在 1Panel 部署

1. 在应用商店安装 OpenResty。2026-10-31 的安装截图明确显示使用 host 网络模式；保留 HTTP 80、HTTPS 443 和自动容器名称。CPU 可设为 1 核，内存先设为 256MB，保留拉取镜像，使用应用默认 compose。这个内存值是自用起步预算，安装后需要观察是否有异常重启。
2. 打开“容器 → 编排”，创建名为 `hachile-api` 的编排，将 [compose.yaml](compose.yaml) 内容填入编辑器。
3. 在该编排的环境变量中填写 `SESSION_SECRET=服务器生成的密钥`，该编辑器保存为 `1panel.env`。Compose 已通过 `env_file` 将此文件加载到容器，请保留该引用。命令行部署时，也在 Compose 同目录创建 `1panel.env`，参考 [1panel.env.example](1panel.env.example)。会话密钥需要在服务器生成一次并保存，可用 `openssl rand -hex 32`；它与面板登录密码无关。勿将该文件提交到 Git。
4. 启动编排。确认容器 `hachile-api` 正常运行，健康检查变为 healthy。应用使用 Compose 自动创建的 bridge 网络，无需外部网络；若拉取镜像失败，查看镜像拉取日志。
5. 在 1Panel 的“网站”中创建 `api.hachile.org` 的反向代理站点。host 模式的 OpenResty 与服务器共用网络，代理目标填写 `http://127.0.0.1:3000`，通过主机回环端口访问应用。此方案无需将 OpenResty 加入应用的 Docker 网络。
6. 在 Cloudflare 为 `api` 添加指向服务器公网 IP 的 A 记录。先完成源站 HTTPS 证书配置，再启用访问；如果启用 Cloudflare 代理，SSL 模式使用 Full (strict)。
7. 为反向代理启用 WebSocket 支持，并关闭响应缓冲，以便流式输出。下方提供 location 内应具备的配置项；1Panel 已生成的项目应修改现有项，避免重复添加。
8. 通过 `https://api.hachile.org` 完成 New API 初始化，设置名称为 `Hachile API`。保留上游署名及项目链接。自用场景关闭公共注册，再添加已授权的上游渠道、模型与配额，在控制台测试后创建自己的调用令牌。上游密钥只填在服务端控制台。

编排的 3000 端口只绑定服务器回环地址，保持此设置；对外访问通过 OpenResty 的 80/443 端口。首次初始化也通过 HTTPS 域名完成；配置启用了 Secure Cookie 和 HTTPS Origin 校验，直接用公网 IP 加 HTTP 地址登录不能作为部署成功的验证。若 80/443 无法访问，检查阿里云安全组及服务器防火墙的对应规则。

### 1Panel 环境变量报错

若旧版配置报告 `SESSION_SECRET is missing a value`，即使面板环境变量框已有值，也可能是 Compose 插值阶段未读取 `1panel.env`。当前版本已改为直接使用 `env_file: [1panel.env]`，并移除了 `environment` 中的 `SESSION_SECRET` 插值。不要同时保留旧插值行；不要将密钥粘进公开的 Compose 或截图。文件存在仍不足以保证密钥有效，创建时须在环境变量框填写非空随机值并长期保存。

反向代理的 location 配置参考：

```nginx
proxy_http_version 1.1;
proxy_set_header Host $host;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
proxy_buffering off;
proxy_cache off;
proxy_read_timeout 300s;
proxy_send_timeout 300s;
```

`TRUSTED_PROXIES` 初始为 `none`，应用不会信任转发的来源 IP。确认应用实际看到的代理来源地址后，可将该值设为该地址；经过 Docker 端口转发时，不能仅凭 OpenResty 使用 host 模式就假定来源是 127.0.0.1。这会影响来源 IP 记录与按 IP 限流。

## 验收与备份

- `https://api.hachile.org/api/status` 返回成功状态，控制台可通过 HTTPS 登录。
- 配置一条有效上游渠道，验证一次普通请求和一次流式请求。
- 客户端 Base URL 使用 `https://api.hachile.org/v1`，使用 New API 签发的令牌。
- 在 1Panel 观察应用内存、容器重启次数与服务器可用内存；目前未执行负载测试，不提供并发容量承诺。
- 用 1Panel 备份数据卷 `hachile-api-data`。需要一致的 SQLite 文件备份时，短暂停止应用容器，备份完整卷后再启动。保留会话密钥，备份中包含敏感业务数据。
- 升级前先备份数据，再更换固定镜像标签。若新版本已经迁移数据库，恢复旧镜像前也需恢复与旧版本对应的备份。

## 托管和源码

GitHub 仓库保存部署配置，服务运行于阿里云的 1Panel 服务器。GitHub Pages 无法承载这个 Go 服务及持久化数据库。正式验证上线后再同步 Home 更新日志和 Site 站点目录，避免将未运行的地址列为可用站点。

这里没有修改或复制 New API 的应用源码。上游使用 AGPLv3，并要求修改后的界面保留署名与原项目链接。参见该版本的 [README](https://github.com/QuantumNous/new-api/tree/v1.0.0-rc.47#license)、[LICENSE](https://github.com/QuantumNous/new-api/blob/v1.0.0-rc.47/LICENSE) 和 [环境变量示例](https://github.com/QuantumNous/new-api/blob/v1.0.0-rc.47/.env.example)。
