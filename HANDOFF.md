# Hachile API 部署交接

状态：部署配置已准备，服务器部署待完成。更新时间：2026-10-30。

## 1. 当前目标

在用户的阿里云 2 核、2GB 1Panel 服务器部署自用 Hachile API，域名 api.hachile.org，使用 QuantumNous/new-api。当前已准备固定镜像、SQLite 单实例配置及导入说明。服务器端尚未修改；不涉及其他站点正文。

## 2. 完成条件

HTTPS 域名可访问并完成管理员初始化；注册关闭；合法上游渠道的普通及流式调用通过；资源占用可接受且无异常重启；持久数据有备份。上线后按用户约定将新网站记录同步到 home.hachile.org 与 site.hachile.org，并增加 Site 目录入口，使用一致版本号，新站使 b 增加 1。服务部署到 1Panel，GitHub Pages 仅能用于静态内容，不能运行本服务。

## 3. 已确认事实

截图显示内存 1.04GB/1.83GB，磁盘 10.52GB/39.01GB，CPU 10%，已安装应用等概览计数为 0；容器列表未检查。GitHub 官方 API 于 2026-10-30 返回最新发布 v1.0.0-rc.47，Docker Hub 同名标签存在；已读取对应版本 compose 和环境变量示例。现有本地目录和 GPDdev/api.hachile.org 远程在开始时均为空。

## 4. 相关文件与命令

项目根目录 D:\Desktop\hachile-workspace\api.hachile.org，远程 https://github.com/GPDdev/api.hachile.org，分支 main。部署文件已以 dfe9d87 提交并推送。compose.yaml 限制应用内存为 512MB，SQLite 使用持久卷，主机端口仅监听回环；.env.example 说明服务器端必填值；README.md 是 1Panel 导入、反向代理和验收步骤；GitHub Actions 校验 Compose。未保存任何凭据。Python/PyYAML 已成功解析 YAML，并检查单服务、回环绑定、密钥必填、512MB 预算与卷名。当前电脑无 Docker；GitHub Actions 运行 41085253306 已完成且结论 success，docker compose config --quiet 校验通过。这是配置检查，不能代替服务器上的镜像启动、网络和真实请求验证。

## 5. 已否决方案

普通 Cloudflare Worker 直接运行原版 Go 应用及本地 SQLite 不适用；完整默认 PostgreSQL 与 Redis 栈暂不用于这台内存较小的自用服务器。Windows 电脑控制连续三次报告无法可靠识别 Edge URL，并要求停止；不得绕过该限制读取会话或操作面板。

## 6. 风险与待确认项

尚未在 Docker 运行。需要实际检查 OpenResty 是否安装及所在网络、容器列表、可用内存、域名 DNS 和证书。外部网络名 1panel-network 是示例默认值，须按实际面板校对。512MB 是资源预算而非容量测试结果。面板控制受工具阻断，服务器导入暂需用户操作或使用以后获准且可用的控制方式。Home/Site 同步与部署验收均待真实服务上线。

## 7. 下一步动作

用户在 1Panel 打开“容器 → 网络”与“容器 → 编排”，按 README 确认 OpenResty 网络并导入 compose.yaml，服务器端设置随机 SESSION_SECRET。以容器 healthy 和 HTTPS /api/status 成功为本阶段验证条件；随后完成初始化与真实上游调用。
