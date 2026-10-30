# Hachile API 部署交接

状态：用户截图确认 OpenResty 已启动、API 编排成功启动，待健康状态、域名和 HTTPS 验证。更新时间：2026-10-31。

## 1. 当前目标

在用户的阿里云 2 核、2GB 1Panel 服务器部署自用 Hachile API，域名 api.hachile.org，使用 QuantumNous/new-api。已准备固定镜像、SQLite 单实例配置及导入说明。用户已在面板完成容器启动，Codex 未操作服务器。不涉及其他站点正文，也不能操作面板中的既有 qqbot 编排。

## 2. 完成条件

HTTPS 域名可访问并完成管理员初始化；注册关闭；合法上游渠道的普通及流式调用通过；资源占用可接受且无异常重启；持久数据有备份。上线后按用户约定将新网站记录同步到 home.hachile.org 与 site.hachile.org，并增加 Site 目录入口，使用一致版本号，新站使 b 增加 1。服务部署到 1Panel，GitHub Pages 仅能用于静态内容，不能运行本服务。

## 3. 已确认事实

10 月 30 日截图显示内存 1.04GB/1.83GB，磁盘 10.52GB/39.01GB，CPU 10%；不是当前实时资源占用。10 月 31 日截图确认 OpenResty 1.21.4.3-3-3-focal 已启动，host 网络，HTTP 80、HTTPS 443。最新截图有 Container hachile-api Started 与 docker-compose up successful，已创建卷 hachile-api-data，但没有 healthy 证据。面板还显示 qqbot 编排。10 月 30 日曾核对官方 v1.0.0-rc.47 发布和镜像标签；不是当前最新版本检查。

## 4. 相关文件与命令

项目根目录 D:\Desktop\hachile-workspace\api.hachile.org，远程 https://github.com/GPDdev/api.hachile.org，分支 main。本次修改前本地与远程均为 96fe9da，工作区干净。compose.yaml 使用 env_file: [1panel.env]，配套 1panel.env.example；该私密文件已被 gitignore 排除。移除 SESSION_SECRET 插值，避免 1Panel 保存变量后仍在 Compose 插值时报错；用户按相同修改启动成功。应用限内存 512MB，SQLite 使用持久卷，主机端口仅监听回环。README.md 为部署说明；CI 新增容器最终环境必须从 1panel.env 取得测试值的检查。无密钥保存在仓库。当前电脑无 Docker；旧版 Actions 41093080012 成功不覆盖新环境文件处理，需检查本次提交的 CI。

## 5. 已否决方案

普通 Cloudflare Worker 直接运行原版 Go 应用及本地 SQLite 不适用；完整 PostgreSQL 与 Redis 栈暂不用于小内存自用服务器。OpenResty 为 host 模式，无需共享外部 1panel-network。1Panel 环境变量编辑器保存到 1panel.env，不能以默认 .env 插值读取；用 env_file 直接注入，且 environment 不可保留覆盖它的 SESSION_SECRET 行。电脑控制多次因无法识别 Edge URL 而停止；不得绕过限制读取会话或操作面板。

## 6. 风险与待确认项

容器启动不代表应用健康；尚未验证源站证书、初始化及模型调用。2026-10-31 Google 公共 DoH 查询返回 api.hachile.org CNAME gpddev.github.io，须改为服务器的 A 记录；本机 198.18.0.127 是代理保留地址，不能用作真实公共解析证据。OpenResty 256MB 与 API 512MB 均是资源预算。API 信任代理初始为 none，真实代理来源待确认。截图含密钥，不得复制到文件、仓库或交接记录；初始化前应由用户换成新的随机值。面板控制受工具阻断，后续操作暂需用户完成。Home/Site 同步与部署验收均待真实服务上线。

## 7. 下一步动作

用户在 1Panel 的“网站 → 创建网站”选择反向代理，域名 api.hachile.org，上游 http://127.0.0.1:3000；随后配置域名解析与 HTTPS。收集创建页字段截图，不能假定站点或证书已创建；以容器 healthy 和 HTTPS /api/status 成功为验证条件。
