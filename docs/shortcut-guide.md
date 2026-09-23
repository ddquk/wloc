# 使用、迁移与排障

## 首次使用

先阅读 README 的兼容性状态，再安装对应客户端模块，启用 MITM 并信任客户端证书。模块匹配的主机包括 `gs-loc.apple.com`、`gs-loc-cn.apple.com`、`gsp-ssl.ls.apple.com` 及两种上游高德备用主机。

用 Safari 打开自己部署的选点网页，选点后点击「储存到设备」。网页查询的「当前生效坐标」是代理本地保存值，不是设备定位服务的独立测量结果。

## 从旧仓库迁移

1. 订阅 [ddquk/wloc](https://github.com/ddquk/wloc) 对应模块，停用旧模块，避免重复脚本执行。
2. 保留客户端持久化键 `wloc_settings`；本分支没有改名。
3. 原网页收藏在旧站点的 localStorage 中，换站点不会自动迁移。请先记录收藏，不要清除旧浏览器数据。
4. 模块默认参数继续沿用上游值；自定义参数要手动核对。

## 快捷指令

### 安装快捷指令

- [WLOC设置位置 xepes0](https://www.icloud.com/shortcuts/0a6465168d554135b78008a8b4bd7c01)
- [wloc 清理恢复位置](https://www.icloud.com/shortcuts/f42632d406504f24a2cd163af4fe012f)

以上链接保留上游 iCloud 分享包及名称，设置位置模板内部仍可能使用上游解析服务。导入模板或复制手机上已正常使用的设置指令后，必须按下文修改解析地址；本仓库未发布已改好地址的新分享包。修改后可重命名为“WLOC设置位置 ddquk”。

在苹果地图选点 → 共享 → 选择“WLOC设置位置 ddquk”；高德地图通过「分享 → 更多」调用。新服务和修改后的指令仍需真机验证。恢复指令用于清除保存值，不保证立即清除系统定位缓存。

### 替换旧解析服务

本仓库的 Cloudflare Worker 名称为 `ddquk-wloc`，选点网页为 https://ddquk-wloc.dddquk.workers.dev/ ，解析接口为 `https://ddquk-wloc.dddquk.workers.dev/api/parse`。可以直接使用 Cloudflare 分配的 `workers.dev` 地址，后续再绑定自己的域名。更换域名时也需同步修改快捷指令。

1. 按[部署说明](DEPLOYMENT.md)部署自己的 Worker，取得 HTTPS 地址。
2. 如果已经装过旧指令，先在「快捷指令」App 中复制一份备份，再打开设置指令的编辑界面。
3. 查找包含 `wloc.xepesw.workers.dev` 或更早的 `wloc-spoofer.wloc.workers.dev` 的 URL / 文本动作，把该服务地址替换为自己的 Worker 地址，保留 `/api/parse` 路径、查询参数及输入变量。导入上方模板也必须完成这一步。
4. 保留 `https://gs-loc.apple.com/wloc-settings/save`。它是客户端拦截的设备保存路径，不是旧公共 Worker。
5. 将修改后的指令重命名为“WLOC设置位置 ddquk”，用地图分享链接检查解析和保存结果，再运行恢复指令确认清理行为。

README 和模块中的新 GitHub 地址不会自动同步到已安装的快捷指令。如果 iCloud 分享失效，仍可使用自部署选点网页；本仓库未恢复可直接导入的 `.shortcut` 文件。

解析接口：`GET https://ddquk-wloc.dddquk.workers.dev/api/parse?format=json&u=<URL编码的地图链接>`，成功返回 `lat`、`lon`、`name`。使用「获取词典值」的快捷指令必须保留 `format=json`，并对输入地图链接进行 URL 编码。站点根地址返回选点网页，不能代替解析接口。不带 `format=json` 时默认返回 `lat=...&lon=...` 纯文本，供旧快捷指令兼容使用。保存接口仍为 `https://gs-loc.apple.com/wloc-settings/save`，它由手机代理脚本拦截，不是 Worker 路由；不要将这个 Apple 地址替换为 Worker 域名。

手动构建快捷指令时：接收分享文本 → URL 编码后请求解析接口 → 检查成功 JSON → 将 lat/lon 传给保存接口。恢复操作使用相同保存路径并附 `?action=clear`。先检查失败响应，避免把空结果写入设备。

## 排障顺序

| 现象 | 检查方向 |
| --- | --- |
| 模块下载失败 | GitHub raw 地址、仓库是否公开、网络可达性 |
| 地图空白 | Leaflet CDN、瓦片服务、浏览器网络 |
| 链接解析失败 | Worker 路由、原链接格式、地图服务响应 |
| 储存失败 | Safari 是否经过代理、模块规则和 MITM 证书 |
| 储存成功但定位不变 | 系统版本、locationd TLS 拒绝、GPS 覆盖、定位缓存 |
| 清除后仍改变位置 | 模块参数中是否另设了自定义经纬度，是否有重复模块 |

上游提出重启可能帮助清除缓存，但重启不能解决 TLS 证书校验限制。不要把反复切换飞行模式当成所有系统版本通用的解决方案。
