# Hachile API 部署交接

状态：容器已启动，但公网域名现被阿里云备案拦截；待用户选择符合部署条件的托管地域或域名。更新时间：2026-10-31。

## 1. 当前目标

部署自用 Hachile API，首选域名 api.hachile.org，使用 QuantumNous/new-api。原计划使用用户阿里云 2 核、2GB 1Panel 服务器；固定镜像与 SQLite 单实例已由用户在面板启动。当前必须先解决托管前提，暂停继续申请 HTTP 证书。Codex 未操作服务器，不涉及其他站点正文，也不能操作既有 qqbot 编排。没有授权购买新服务器或代提交备案。

## 2. 完成条件

HTTPS 域名可访问并完成管理员初始化；注册关闭；合法上游渠道的普通及流式调用通过；资源占用可接受且无异常重启；持久数据有备份。上线后按用户约定将新网站记录同步到 home.hachile.org 与 site.hachile.org，并增加 Site 目录入口，使用一致版本号，新站使 b 增加 1。服务部署到 1Panel，GitHub Pages 仅能用于静态内容，不能运行本服务。

## 3. 已确认事实

OpenResty 1.21.4.3-3-3-focal 已启动，host 网络，HTTP 80、HTTPS 443；API 编排已创建 hachile-api-data 卷并启动。Google 公共 DoH 确认 api.hachile.org A 记录为 47.101.65.25。HTTP /api/status 曾返回 200、success=true、v1.0.0-rc.47、New API，Server=openresty，这是历史结果。随后用户 HTTP-01 日志显示验证文件返回 403；本轮独立 GET 复现首页、/api/status、相同 challenge 路径全为 403，Server=Beaver，页面标题 Non-compliance ICP Filing，包含 aliyun.com/beian/beian-block。该证据确认阿里云备案层拦截，不是单独 challenge 路由错误。HTTPS 此前为 TLS unexpected EOF，没有证书已配置的证据。资源截图为旧读数，真实当前占用待核对。

## 4. 相关文件与命令

项目根目录 D:\Desktop\hachile-workspace\api.hachile.org，远程 https://github.com/GPDdev/api.hachile.org，分支 main；本轮修改前本地/远程均为 1d90eb6，工作区干净。本轮仅更新 README.md 和此交接，保留容器配置。compose.yaml 使用 env_file: [1panel.env]，配套 1panel.env.example，私密文件被 gitignore 排除；应用限 512MB，SQLite 持久卷，端口仅监听回环。仓库无密钥。历史 Actions 41098810211 对 aa8a260 的 Compose schema 和环境加载检查 success，不能当作本轮网站运行正常的证据。当前 403 通过 Invoke-WebRequest -SkipHttpErrorCheck 实际读取确认。

## 5. 已否决方案

普通 Worker 无法直接部署原版 Go 服务与本地 SQLite；小内存暂不用完整 PostgreSQL/Redis 栈。OpenResty host 模式无需外部共享网络。1Panel 变量须通过 env_file 注入，不能保留覆盖它的 SESSION_SECRET 插值。HTTP 备案拦截时反复签发、调整 challenge 路径无效；DNS-01 证书本身不解决网站备案拦截。电脑控制多次无法识别 Edge URL 并要求停止，不得绕过限制读取会话或操作面板。

## 6. 风险与待确认项

阿里云官方当前说明未备案/未接入域名在内地服务器被阻断：https://help.aliyun.com/zh/icp-filing/not-for-the-record-dns-can-access-to-different-areas 。.org 注册管理机构许可限制与备案要求见 https://help.aliyun.com/zh/dws/product-overview/limits 。保留 .org 时可选择非中国内地服务器；保留内地服务器时需满足域名与备案条件，具体流程待用户决定。不得自行购买或提交用户身份材料。初始化与真实模型调用未完成；资源预算非容量测试，信任代理为 none；截图含密钥，不能复制，初始化前用户需换新值。Home/Site 同步和最终验收待解决托管条件并实际上线。

## 7. 下一步动作

先取得用户对托管方案的选择：保留 api.hachile.org 并迁到非中国内地服务器，或使用符合备案条件的域名与现有内地服务器。确认后再核对目标服务器和域名条件，复用现有部署文件，保留 hachile-api-data，不能删除卷。解决条件后再验证 HTTPS /api/status 和真实调用；不继续在拦截状态下反复申请 HTTP-01。
