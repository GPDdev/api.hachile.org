# Hachile API 部署交接

状态：已按确认的 OpenResty host 网络修正部署配置，服务器部署待完成。更新时间：2026-10-31。

## 1. 当前目标

在用户的阿里云 2 核、2GB 1Panel 服务器部署自用 Hachile API，域名 api.hachile.org，使用 QuantumNous/new-api。已准备固定镜像、SQLite 单实例配置及导入说明。用户正在 OpenResty 安装页，尚未提供安装成功证据；Codex 未操作服务器。不涉及其他站点正文。

## 2. 完成条件

HTTPS 域名可访问并完成管理员初始化；注册关闭；合法上游渠道的普通及流式调用通过；资源占用可接受且无异常重启；持久数据有备份。上线后按用户约定将新网站记录同步到 home.hachile.org 与 site.hachile.org，并增加 Site 目录入口，使用一致版本号，新站使 b 增加 1。服务部署到 1Panel，GitHub Pages 仅能用于静态内容，不能运行本服务。

## 3. 已确认事实

10 月 30 日截图显示内存 1.04GB/1.83GB，磁盘 10.52GB/39.01GB，CPU 10%，已安装应用等概览计数为 0；容器列表未检查。10 月 31 日 OpenResty 安装页明确提示 host 网络模式，HTTP 80、HTTPS 443，版本选项显示 1.21.4.3-3-3-focal。已建议 OpenResty 先用 1 核/256MB 的预算，实际效果待验证。GitHub 官方 API 于 10 月 30 日返回发布 v1.0.0-rc.47，Docker Hub 同名标签存在；这不是 10 月 31 日的最新版本检查。

## 4. 相关文件与命令

项目根目录 D:\Desktop\hachile-workspace\api.hachile.org，远程 https://github.com/GPDdev/api.hachile.org，分支 main。此前已推送部署配置及交接记录，修改前本地与远程均为 0afc067 且工作区干净。本次修改 compose.yaml、.env.example、README.md 和本文件，移除外部网络依赖，host OpenResty 的目标改为 http://127.0.0.1:3000。应用仍限内存 512MB，SQLite 使用持久卷，主机端口仅监听回环。未保存凭据。当前电脑无 Docker；历史 Actions 41085253306 成功，但只验证旧版配置，新版需运行同一 Validate deployment configuration 工作流核对。配置检查不能代替服务器运行验证。

## 5. 已否决方案

普通 Cloudflare Worker 直接运行原版 Go 应用及本地 SQLite 不适用；完整 PostgreSQL 与 Redis 栈暂不用于这台内存较小的自用服务器。已否决 OpenResty 与应用共享外部 1panel-network 的配置：用户的 OpenResty 为 host 模式，使用主机回环端口即可。Windows 电脑控制连续三次报告无法可靠识别 Edge URL，并要求停止；不得绕过该限制读取会话或操作面板。

## 6. 风险与待确认项

尚未在 Docker 运行。需要实际检查 OpenResty 安装是否成功、容器列表、可用内存、域名 DNS 和证书。OpenResty 256MB 与 API 512MB 均是资源预算而非容量测试结果。API 信任代理初始为 none，真实来源 IP 设置需确认。面板控制受工具阻断，服务器导入暂需用户操作或使用以后获准且可用的控制方式。Home/Site 同步与部署验收均待真实服务上线。

## 7. 下一步动作

用户完成当前 OpenResty 安装，并提供“已安装”页的运行状态；确认成功后，在“容器 → 编排”按 README 导入修正后的 compose.yaml，服务器端设置随机 SESSION_SECRET。反向代理目标为 http://127.0.0.1:3000。以容器 healthy 和 HTTPS /api/status 成功为验证条件，随后完成初始化与真实上游调用。
