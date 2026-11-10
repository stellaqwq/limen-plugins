# limen-plugins

Limen 的插件市场：一份静态清单 + 一堆插件 ZIP，没有任何服务端。客户端读 `market/index.json`，按清单里的地址下载插件包，校验 `size` 与 `sha256` 后交给本机安装器。

## 客户端怎么用

在「全部设置 → 插件市场 → 市场清单地址」填：

```
https://raw.githubusercontent.com/stellaqwq/limen-plugin/main/market/index.json
```

保存后回到「插件市场」点「刷新清单」，就能看到下面这些插件。`允许下载的域名` 留空即可——默认只信任清单所在域名（`raw.githubusercontent.com`），插件包也在同一个域名下，不需要额外放行。

## 目录结构

```
.
├── market/
│   ├── index.json          清单（生成物，不要手改）
│   ├── packages/           插件包 <id>-<版本>.zip（生成物）
│   └── assets.json         可选：FFmpeg 这类运行资源的描述（手工维护）
├── scripts/
│   └── validate-market.mjs 校验清单与包是否一致
└── .github/workflows/
    └── validate-market.yml 推送后自动校验
```

## 清单里的插件

| id | 名称 | 版本 |
| --- | --- | --- |
| `android` | 安卓视频浏览 | 1.0.0 |
| `autonomous-thought` | 自主思考 · 独处日记 | 2.11.0 |
| `background` | 背景壁纸 | 1.0.0 |
| `comfyui` | ComfyUI 生图 | 2.2.0 |
| `douyin-web` | 抖音网页浏览 | 1.0.0 |
| `emotion` | 心绪 · 心理动力情绪 | 1.0.0 |
| `free-activity` | 自由活动 | 1.1.0 |
| `group-chat-plus` | 群聊增强 Chat Plus | 1.1.0 |
| `living-memory` | 流动记忆 Living Memory | 1.0.0 |
| `memes` | 表情包管理器 | 1.0.0 |
| `startup-animation` | 启动动画 | 1.1.0 |
| `video` | 视频理解 | 2.0.0 |
| `voicestudio` | 本地 VoiceStudio | 1.0.0 |

准确版本、字节数与校验值以 `market/index.json` 为准。

## 发布新版本

在 Limen 源码树（`package.json` 所在目录）执行：

```bash
npm run market:publish
```

它会把 `plugins/` 下每个含合法 `plugin.json` 的目录重新打包、算好 `size`/`sha256`，把 `index.json` 里的 URL 指向本仓库的 raw 地址，然后克隆本仓库、替换 `market/`、提交并推送。

```bash
npm run market:publish -- --dry-run          # 只生成到 dist/market，不推送
npm run market:publish -- --only video,memes # 只发布指定的插件
npm run market:publish -- --branch main      # 换分支
```

## 插件包规则

- 包内必须有一个 `plugin.json`，其中 `id`、`kind` 与清单一致，`version` 为 `x.y.z`，`api` 必须是 `1`；
- 单文件不超过 2 MiB，解包后不超过 24 MiB；
- 插件私有数据不要写进插件目录：更新是「先删目录再写文件」，数据会被一起替换；
- 大体积二进制（FFmpeg 之类）不要进插件包，走 `market/assets.json` 的资源通道，安装到 `runtime/` 下。

## 安全边界

清单本身没有签名，安全性依赖 GitHub 的 https 与客户端的域名白名单；客户端下载后先比字节数、再比 SHA-256、最后比对包内 `plugin.json`，全部通过才落盘。
