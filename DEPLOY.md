# Cloudflare Pages 构建说明

## 失败原因

原构建日志的关键错误：

```
[build] Running: npx bundle exec jekyll build
[build] npm error could not determine executable to run
✘ [ERROR] Running custom build `npx bundle exec jekyll build` failed.
```

`npx` 是 Node 包执行器，它会把 `bundle` 当成一个 **npm 包名** 去查找并安装。
而 `bundle`（Bundler）是 Ruby 工具，npm 上不存在这个可执行包，所以 npx 在真正
调用 Jekyll 之前就直接退出了（exit code 1）。这不是 Jekyll 配置问题，是构建命令前缀写错了。

另外仓库缺少 `Gemfile`（`.gitignore` 里把 `Gemfile` 和 `Gemfile.lock` 忽略了），
导致 Cloudflare 无法安装 Jekyll 依赖。

## 需要在 Cloudflare Pages 控制台修改的设置

进入项目 → Settings → Build & deployments：

| 项目 | 原值（错误） | 改为 |
| --- | --- | --- |
| Framework preset | Static | **Jekyll** |
| Build command | `npx bundle exec jekyll build` | `bundle exec jekyll build` |
| Build output directory | `_site` | `_site`（保持不变） |

`_site` 是对的，不需要改。

## 已修复的仓库文件

1. **新增 `Gemfile`** — 声明 `jekyll ~> 4.3.4` 及 `jekyll-sitemap`、`jekyll-feed`，
   并固定 `ruby "3.1.6"`，与 Cloudflare 构建镜像保持一致。
2. **新增 `.ruby-version`** — 内容 `3.1.6`，Cloudflare 会按此选择 Ruby 版本。
3. **`.gitignore`** — 去掉对 `Gemfile` / `Gemfile.lock` 的忽略，改忽略 `vendor/`、`.wrangler`。
   注意：首次成功构建后请把生成的 `Gemfile.lock` 一起提交，锁定依赖版本。
4. **`_config.yml`**
   - `gems:` 改为 `plugins:`（Jekyll 4 已移除 `gems:` 键，继续用会告警且插件不加载）。
   - 补上 `url: "https://xuxuecheng.github.io"`，否则 sitemap 与 feed 里的绝对链接不完整。
   - `exclude` 增加 `.ruby-version`、`node_modules`、`vendor`。

## 本地验证方式

本机需要 Ruby 3.1+（当前这台机器未安装 Ruby，因此未能在本地实际跑通构建）：

```bash
gem install bundler
bundle install
bundle exec jekyll build   # 产物在 _site/
bundle exec jekyll serve   # 本地预览 http://127.0.0.1:4000
```

## 关于 npx wrangler（可选了解）

日志里 `Executing user deploy command: npx wrangler deploy` 是 Cloudflare Workers/Pages
的新部署通道，它会自动探测框架并生成 `wrangler.jsonc`（输出目录 `_site`）。
它探测到的 Build Command 就是那条错误的 `npx bundle exec jekyll build`。
因此只要把控制台里的构建命令改正，这条链路即可正常跑。