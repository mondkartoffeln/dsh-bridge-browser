> # ⛔ 本仓库已废弃（2026-10-02）
>
> **不要再用这份适配版。** 上游原作者**已完成官方适配**：
> **https://github.com/omdsh-dev/dsh-browser**（`packages/browser/bridge-browser`，**v0.0.7**，peer 已钉 `0.2.0-rc.2`，MIT，持续维护）
>
> **实测对比（同一台机器、同一令牌）**：
> - 本仓库的补丁只是"事件流崩了不掐线"——**推送流是断的**，握手后宿主立刻 `close(1011)`
> - **上游是在 host 适配层真修的**：握手后**无错误帧、连接保持、事件流可用**
>
> **改用上游的步骤**（桌面端插件面板，路径必须带盘符）：
> 1. `git clone https://github.com/omdsh-dev/dsh-browser.git`
> 2. `pnpm install` → `pnpm --filter @yuxianglin/dsh-bridge-browser run build`（上游**不提交 `lib/`**，必须自行构建）
> 3. 面板里卸载本插件，改为从本地路径安装：
>    `D:/<你的克隆路径>/dsh-browser-upstream/packages/browser/bridge-browser`
>
> 留着本仓库只为**记录当时的排查过程**，不再更新。
# dsh-bridge-browser (DSH 0.2.0-rc.2 compatible fork)

上游：[`@yuxianglin/dsh-bridge-browser`](https://github.com/) v0.0.5（MIT，作者 Yuxiang Lin）。
本仓库是**为 DSH 桌面端 0.2.0-rc.2 做的兼容适配版**，行为与上游一致，只改了下面三处。

## ⚠️ 免责与致谢

这是**非官方适配版**，唯一目的是让 `@yuxianglin/dsh-bridge-browser` 能在 **DSH 0.2.0-rc.2** 下继续可用，属于**临时过渡**。
**核心实现、设计与版权均属原作者 Yuxiang Lin（MIT）。** 本仓库只包含兼容性改动。
**原作者如有任何异议，请在本仓库开 issue，我会立即下架或移交。**

## 为什么需要这个 fork

上游把 `peerDependencies` 钉在 `^0.1.5-rc.2`。在 semver 里 `0.x` 的 `^` **只允许同一 minor 内升级**，
所以 DSH 桌面端一升到 `0.2.0-rc.2`，插件面板立刻判为"不兼容"。
实测（扫 `app.asar` 全部字节）确认：**它依赖的 3 个符号 + 5 个服务名在 0.2.0-rc.2 里全部存在**，
没有重命名、没有移除——所以这是**声明问题**，不是真的不能用。

## 改了哪三处

| # | 文件 | 改动 | 原因 |
|---|---|---|---|
| 1 | `package.json` | 13 处 `^0.1.5-rc.2` → `>=0.1.5-rc.2 <0.3.0`（开区间）；版本号 `0.0.5` → `0.0.6` | 避免每升一个 minor 就报不兼容 |
| 2 | `lib/index.js` | 事件流抛错时**不再发 error 帧、不再 `close(1011)`**，改为 `console.warn` | DSH 0.2.0 下 `deps.api.events(signal)` 的参数形状变了，内部 `AbortSignal.any` 抛 `TypeError: signals[0] is not of type AbortSignal`；一抛错就掐线，扩展侧表现为"未连接 dsh" |
| 3 | 全部 JSON | 去掉 BOM | 上游文件本身没问题；是我用 Windows PowerShell 5.1 的 `Set-Content -Encoding UTF8` 改文件时带进去的，Node 的 `JSON.parse` 遇 BOM 会抛异常 |

## 已知上游待办（未根治）

`lib/index.js` 里 `for await (const frame of this.deps.api.events(abort.signal))` 这一行
**仍是按 0.1.5 的签名调用的**。当前补丁只是"崩了但不掐线"，**RPC 通道可用、事件推送流是断的**。
根治需要按 0.2.0 的真实签名改造该调用。

## 安装（DSH desktop profile）

在桌面端插件面板里从**本地路径**安装；或用 CLI（仅限非 desktop profile）：

```
dsh plugin --profile <profile> add <本仓库路径或 git 地址>
```

> ⚠️ 路径识别要求**带盘符的绝对路径**（如 `D:/path/to/repo`）或 `github:` 地址；
> 裸相对路径会被当成 npm 包名去 registry 查。

## 许可

沿用上游 MIT。本仓库只做兼容性改动，**核心实现的版权属原作者 Yuxiang Lin**。