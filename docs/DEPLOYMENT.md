# 部署说明

本仓库使用 [ddquk/wloc](https://github.com/ddquk/wloc) 发布模块和脚本，使用 Cloudflare Worker `ddquk-wloc` 托管网页与解析接口。先使用 Cloudflare 分配的 `workers.dev` 地址即可，不需要先配置自己的域名；后续可为同一个 Worker 绑定域名。

## 准备仓库

使用 Node.js 22 或更新版本。在仓库根目录编辑 `project.config.json`：

```json
{
  "repository": "ddquk/wloc",
  "branch": "main",
  "siteUrl": "https://ddquk-wloc.dddquk.workers.dev/"
}
```

`repository` 和 `branch` 使用实际发布位置；仓库须公开，客户端才能匿名下载 raw 脚本。`siteUrl` 可留空，已有自己的站点时填写 HTTPS 根地址。运行 `npm run configure` 生成模块和源码链接，再运行 `npm run check:release`。不要直接编辑生成的模块；改 `templates/modules/`。

## Cloudflare Workers

```sh
cd worker
npm ci
npm test
npm run build:check
npx wrangler login
npm run deploy
```

`build:check` 是 dry-run，不上传或发布。`deploy` 会真正写入 Cloudflare。`wrangler.jsonc` 已设置 `name: "ddquk-wloc"` 和 `workers_dev: true`；部署前确认账户中没有需要保留的同名 Worker。配置没有绑定 KV、数据库或账户 ID。

从部署输出取得真实的 `workers.dev` 站点 URL，填写根目录 `project.config.json` 的 `siteUrl`，重新运行 `npm run configure` 和 `npm run check:release`，再提交并推送模块到 `ddquk/wloc`。不要用 Worker 名称猜测完整域名；地址还包含 Cloudflare 账户的子域名。

本实例已部署，尚待手机真机验证。选点网页为 [https://ddquk-wloc.dddquk.workers.dev/](https://ddquk-wloc.dddquk.workers.dev/)，解析接口为 `https://ddquk-wloc.dddquk.workers.dev/api/parse`。[WLOC设置位置 ddquk](https://www.icloud.com/shortcuts/e0cd9222aa114385809a312e5861c005) 已配置该接口，直接导入即可。保留旧指令或另行部署实例时，按照[快捷指令迁移说明](shortcut-guide.md#替换旧解析服务)修改解析地址，保留原有查询参数、输入变量，以及 Apple 保存入口 `https://gs-loc.apple.com/wloc-settings/save`。

[WLOC恢复位置 ddquk](https://www.icloud.com/shortcuts/7d10955e5e7e46edb9c6b628586edfeb) 用于清除设备中保存的虚拟坐标，使用本机拦截路径 `https://gs-loc.apple.com/wloc-settings/save?action=clear`，无需配置 Worker 域名。

发布源码应与线上运行版本一致；网页底部提供源码入口。若从发布 tag 部署，可将配置中的 `branch` 设为相应 tag 后生成。

以后切换到自定义域名时，同步更新 `siteUrl`、README 和快捷指令说明中的服务地址，重新生成模块，并重新发布已修改地址的 iCloud 快捷指令分享链接。手机上已安装的快捷指令需重新导入或手动修改。仅修改仓库不会自动更新已安装的指令。

## Cloudflare Pages

```sh
cd worker
npm ci
npm run pages:build
npx wrangler login
npm run pages:deploy
```

`pages:build` 仅检查 Functions 打包，产物在仓库 `build/pages`。`pages:deploy` 使用 `wrangler.pages.jsonc` 以及 `worker/dist` 静态目录，并由 Wrangler 处理 `functions` 目录。首次使用时按提示选择或创建 Pages 项目。

同时修改项目名时也检查 `wrangler.pages.jsonc`；两个配置的 compatibility_date 保持一致。不要把两个配置的 name 当成域名。

## 部署后检查

- 首页正常加载，底部源码入口指向实际发布仓库。
- `/api/parse?u=31.230400,121.473700&format=json` 返回对应 lat/lon，并带 `Cache-Control: no-store`。
- `/api/parse?format=json` 返回 422，说明输入缺失。
- GitHub raw 模块中的两个脚本 URL 和图标可匿名访问。
- 手机上的设置指令使用 `https://ddquk-wloc.dddquk.workers.dev/api/parse`，并保留原有查询参数和输入变量。
- 真机检查保存、查询、清除，以及定位响应是否被拦截；网页成功不能替代这一步。

外部短链接、地图瓦片和搜索服务还需联网测试。Cloudflare 额度以自己的控制台为准，本项目不保证公共实例长期可用。
