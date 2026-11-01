# Hachile API 部署交接

状态：用户已选择保留现有服务器，并购买 hachapi.top；用户报告实名已完成，待提交 ICP 备案。容器保留，公网服务尚未上线。更新时间：2026-11-01。

## 1. 当前目标

部署自用 Hachile API，用户已在阿里云中国站购买新域名 hachapi.top，用于解决旧 api.hachile.org 的备案条件问题。继续使用用户阿里云 2 核、2GB 1Panel 服务器；固定镜像与 SQLite 单实例已由用户在面板启动。当前先办理 ICP 备案，暂停重复申请 HTTP 证书。Codex 未操作服务器，不涉及其他站点正文，不能操作既有 qqbot 编排；未获授权代提交身份材料。项目目录和 GitHub 仓库继续沿用 api.hachile.org，不重新初始化、不更换数据卷。

## 2. 完成条件

HTTPS 域名可访问并完成管理员初始化；注册关闭；合法上游渠道的普通及流式调用通过；资源占用可接受且无异常重启；持久数据有备份。上线后按用户约定将新网站记录同步到 home.hachile.org 与 site.hachile.org，并增加 Site 目录入口，使用一致版本号，新站使 b 增加 1。服务部署到 1Panel，GitHub Pages 仅能用于静态内容，不能运行本服务。

## 3. 已确认事实

OpenResty 1.21.4.3-3-3-focal 已启动，host 网络，HTTP 80、HTTPS 443；API 编排已创建 hachile-api-data 卷并启动。Google 公共 DoH 确认 api.hachile.org A 记录为 47.101.65.25。HTTP /api/status 曾返回 200、success=true、v1.0.0-rc.47、New API，Server=openresty，这是历史结果。随后用户 HTTP-01 日志显示验证文件返回 403；本轮独立 GET 复现首页、/api/status、相同 challenge 路径全为 403，Server=Beaver，页面标题 Non-compliance ICP Filing，包含 aliyun.com/beian/beian-block。该证据确认阿里云备案层拦截，不是单独 challenge 路由错误。HTTPS 此前为 TLS unexpected EOF，没有证书已配置的证据。资源截图为旧读数，真实当前占用待核对。

11 月 1 日用户明确表示 hachapi.top 已在阿里云购买且实名认证完成，购买成功截图与域名一致。截图是支付完成页，仍带实名未完成提示，可能为旧状态；未直接读取账户，域名控制台的实名审核通过状态仍待核对。用户没有报告 ICP 备案已提交或通过。已通过 agent-reach 的网页读取路径核对阿里云官方备案准备文档：域名实名主体需匹配备案主体，服务器资质由备案系统核验。

## 4. 相关文件与命令

项目根目录 D:\Desktop\hachile-workspace\api.hachile.org，远程 https://github.com/GPDdev/api.hachile.org，分支 main；本轮开始时工作区干净。本轮只更新此交接，未修改服务器或应用配置。compose.yaml 使用 env_file: [1panel.env]，配套 1panel.env.example，私密文件被 gitignore 排除；应用限 512MB，SQLite 持久卷，端口仅监听回环。SESSION_COOKIE_TRUSTED_URL 仍是 https://api.hachile.org，上新域名前必须更新并重建应用容器，保留卷。README 仍记录旧 .org 部署受阻的历史状态，后续新站实际配置时一并更新。历史 Actions 41098810211 成功不能当作当前运行状态证据；本轮未重新探测服务器。

## 5. 已否决方案

普通 Worker 无法直接部署原版 Go 服务与本地 SQLite；小内存暂不用完整 PostgreSQL/Redis 栈。OpenResty host 模式无需外部共享网络。1Panel 变量须通过 env_file 注入，不能保留覆盖它的 SESSION_SECRET 插值。HTTP 备案拦截时反复签发、调整 challenge 路径无效；DNS-01 证书本身不解决网站备案拦截。电脑控制多次无法识别 Edge URL 并要求停止，不得绕过限制读取会话或操作面板。

## 6. 风险与待确认项

当前需要域名实名状态核对、备案基础信息/服务器资质核验、真实网站与服务内容提交审核；不能承诺购买支持后缀就会自动获批，不能将 API 网关伪报为其他网站类型。若提示实名信息未同步，按备案系统提示等待后重试，阿里云资料说明常有 2～3 天同步时间。不得代填或保存证件信息。初始化与模型调用未完成；预算非容量测试，信任代理为 none；旧截图含会话密钥，不能复制，初始化前用户需换新值。Home/Site 同步和最终验收待备案及实际部署完成。

## 7. 下一步动作

用户登录购买服务器的阿里云账号，打开 https://beian.aliyun.com/ 的备案系统，若从未备案则开始首次备案，域名填 hachapi.top，主体按本人真实情况填写，产品选择现有轻量应用服务器（公网 IP 此前为 47.101.65.25）。先完成基础信息校验；如显示已有主体或要求其他备案类型，按实际状态选择，不能盲选首次备案。收集不含证件号码等敏感信息的核验结果截图，再指导后续服务信息填写。备案通过后再配置新域名、受信任 HTTPS Origin、证书与实际调用。
