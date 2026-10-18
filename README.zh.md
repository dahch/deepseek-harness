# DeepSeek Harness

[English](README.md) | 中文

DeepSeek Harness（`dsh`）是由 [DeepSeek AI](https://deepseek.com) 开发的开源 agent harness（智能体框架）。

它构建于**一切皆插件**的架构之上，由 [Cordis](https://github.com/cordiverse/cordis) 驱动，其设计参见论文 [_A Programming Paradigm for Spatiotemporal Composability_](https://arxiv.org/abs/2608.25512)。

文档：[https://deepseek-harness.github.io/deepseek-harness/](https://deepseek-harness.github.io/deepseek-harness/)

## 开发者预览

DeepSeek Harness 处于 _开发者预览_ 阶段，正在快速迭代。**未来将出现破坏兼容性的变更。**

运行本项目前，请阅读[安全说明](SAFETY.zh.md)。

## Linux 构建（非官方）

> **非官方构建。** 本分支是个人 fork，与 DeepSeek **没有任何隶属、赞助或背书关系**。这里没有官方 Linux Desktop 支持，没有支持承诺，也不保证这些产物适合生产环境或会持续维护。上游打包指南仍写着 “Linux is not a supported Desktop release target”。

本分支新增本地、未签名的 `linux-x64` Desktop 构建，使用与 macOS/Windows 相同的打包流水线生成 RPM 和 AppImage，并包含下文所述的 Linux 专属 sharp 规避方案。

### 测试环境

| 项目 | 值 |
|---|---|
| 发行版 | Fedora Linux 44（KDE Plasma Desktop Edition） |
| 桌面 | KDE Plasma 6.7.5 / KWin 6.7.5 |
| 会话 | Wayland（`XDG_SESSION_TYPE=wayland`） |
| 架构 | x86_64 |
| 内核 | `7.2.11-200.fc44.x86_64` |
| 打包用 Node | 24.22.0（打包运行记录） |
| 打包用 pnpm | 11.28.5（仓库锁定版本；本地日志显示实际使用 11.28.5） |

仅测试了上述环境。X11、其他发行版、ARM64 和其他 GPU 厂商均未测试。

### 前置要求

#### 运行时

- **Fedora 44 x86_64**（已测试目标）。其他 glibc 发行版可能可用，但未测试。
- **`libxcrypt-compat`**：

  ```sh
  sudo dnf install libxcrypt-compat
  ```

  Fedora 44 默认不安装 `libcrypt.so.1`（`libxcrypt-compat` 是独立软件包）。内置 Python 的 `_crypt` 扩展链接它，因此缺少兼容包时 `import crypt` 会失败。在测试机上，即使系统未安装该包，Desktop GUI 本身也能启动，所以这主要是内置 Python 的要求；安装它是为了与测试环境一致。

- **约 1.1 GB 可用磁盘**，用于安装 RPM 载荷（RPM 报告的安装大小为 1,096,554,466 字节）。构建目录和 AppImage 需要更多空间。

#### 构建

- **git**
- **Node.js**，需满足仓库 `engines` 字段（`^22.19.0 || >=24.0.0`）；测试构建使用 Node 24.22.0。
- **pnpm 11.28.5**，即根 `package.json` 锁定的版本（`"packageManager": "pnpm@11.28.5"`）。不要假设更新的全局 pnpm 等价；仓库锁定 11.28.5，构建期间本地环境报告使用 11.28.5。使用 Corepack：

  ```sh
  corepack enable
  corepack prepare pnpm@11.28.5 --activate
  ```

  如果使用独立安装，请安装该精确版本，并在构建前检查 `pnpm -v`。

- **`rpm-build`**（提供 `rpmbuild`），用于 RPM 目标：

  ```sh
  sudo dnf install rpm-build
  ```

- 首次构建需要网络连接：流水线会下载 Electron 和锁定的 runtime 载荷。

### 构建

```sh
git clone https://github.com/dahch/deepseek-harness.git
cd deepseek-harness
git checkout linux-desktop

corepack enable
corepack prepare pnpm@11.28.5 --activate
pnpm install --frozen-lockfile

pnpm run package:desktop:linux:x64
```

构建输出到：

```
apps/desktop/.desktop-build/targets/linux-x64/unsigned-artifacts/
├── deepseek-harness-0.2.1-rc.2-linux-x86_64-unsigned.AppImage
├── deepseek-harness-0.2.1-rc.2-linux-x86_64-unsigned.rpm
└── linux-unpacked/
```

构建耗时较长，首次使用会下载依赖。如果失败，打包运行会在 `apps/desktop/.desktop-build/packaging-runs/` 下记录各阶段日志。

### 安装

#### RPM

```sh
sudo dnf install ./apps/desktop/.desktop-build/targets/linux-x64/unsigned-artifacts/deepseek-harness-0.2.1-rc.2-linux-x86_64-unsigned.rpm
```

从应用菜单启动，或运行：

```sh
"/opt/DeepSeek Harness/deepseek-harness"
```

卸载：

```sh
sudo dnf remove deepseek-harness
```

#### AppImage

```sh
chmod +x ./apps/desktop/.desktop-build/targets/linux-x64/unsigned-artifacts/deepseek-harness-0.2.1-rc.2-linux-x86_64-unsigned.AppImage
./apps/desktop/.desktop-build/targets/linux-x64/unsigned-artifacts/deepseek-harness-0.2.1-rc.2-linux-x86_64-unsigned.AppImage
```

electron-builder 生成的 AppImage desktop entry 使用 `AppRun --no-sandbox %U`。如果系统无法直接运行 AppImage（例如没有 FUSE），请使用发行版提供的解包运行方式；该路径未在此测试。

### 校验 SHA-256

以下哈希对应 2026-10-18 从提交 `2df4a588` 构建的产物。重新构建会产生不同字节，请与所下载构建一同发布的哈希比较。

```text
1e938cd7f87dbe76dc668678dec002a90148e27e8a000e8a2c7687d290d8f64f  deepseek-harness-0.2.1-rc.2-linux-x86_64-unsigned.AppImage
5664d1fffc02d20982481408865557855c7743fafa427e2f20645751ea4ba338  deepseek-harness-0.2.1-rc.2-linux-x86_64-unsigned.rpm
```

将上面两行保存为产物旁的 `SHA256SUMS`，然后运行：

```sh
sha256sum -c SHA256SUMS
```

在 Fedora 上还可以检查 RPM 的签名状态：

```sh
rpm -qpi deepseek-harness-0.2.1-rc.2-linux-x86_64-unsigned.rpm | grep -E 'Signature|Build Date|Size'
```

本构建显示 `Signature: (none)`。哈希校验是这些产物唯一可用的完整性校验。

### 已验证内容

- 未解包的 Linux 应用在上述环境启动，并打开 `dsh-app://` 窗口。
- Desktop Host 进程运行在内置独立 Node（`resources/runtime/primary-runtime/dependencies/node/bin/node`）下，而不是 Electron Node 模式。
- 渲染进程使用 Wayland（`--ozone-platform=wayland`）。
- 第二次启动立即以退出码 0 结束。
- 关闭主窗口会退出所启动的应用进程。
- AppImage 已解包并检查其载荷（未启动）。
- RPM 已检查（未安装）。
- sharp 在内置 Node 下正常，在 Electron Node 模式下崩溃；分支的规避方案见下文。

本次未测试：安装 RPM、启动 AppImage、X11、其他发行版、ARM64、签名、自动更新，或从干净 clone 执行完整打包流水线。

### 为什么 Linux 上 Host 不使用 Electron 的 Node

Electron 的 Linux Node 模式会把系统 glib 符号泄漏进 libvips，当 Host 处理图片附件时，sharp 的图像解码会段错误（退出码 139）。sharp 将此记录为已知的 Electron/Linux 冲突，并指向 [electron#46323](https://github.com/electron/electron/issues/46323)。一段很小的 sharp 解码在 `ELECTRON_RUN_AS_NODE=1` 下崩溃，在内置独立 Node 下成功。

规避方案保留 Electron 作为 shell，用内置 Node 运行 Host。独立 Node 无法读取 ASAR，因此准备好的 dsh 树以真实文件形式放在 `resources/dsh`（约 418 MB），而不是放进 `app.asar`。打包 runtime smoke 使用同一个解释器。macOS 和 Windows 仍使用 Electron Node 模式。

这是规避方案，不是对底层 Electron/sharp 冲突的修复。它还意味着 Linux 载荷更大且不受 ASAR 保护，任何假设 `resources/app.asar/dsh` 的代码都需要更新。

### 已知限制

- **未签名。** RPM 显示 `Signature: (none)`；AppImage 未签名。没有 GPG 签名或代码签名。请校验上面的 SHA-256。
- **候选版本。** 构建为 `0.2.1-rc.2`，不是稳定版。
- **无自动更新。** 未签名的 Linux 目标不解析更新源（`publish` 为 `null`）；upload、retention 和 release-record 路径会拒绝 `linux-x64`。Linux 更新与发布不在本分支范围内。不要指望更新菜单可用。
- **非官方。** 上游不支持 Linux Desktop。本分支不修改上游文档或 CI；上游文档仍描述 `resources/app.asar/dsh` 和 macOS/Windows 流程。
- **仅测试一种配置。** 仅 Fedora 44 KDE Plasma 6.7.5 / Wayland / x86_64。X11、GNOME、其他发行版、ARM64 和 GPU 相关问题均未测试。
- **观察到 GPU 进程崩溃。** 测试机日志反复出现 `GPU process exited unexpectedly: exit_code=139` 和 `drmGetDevices2() has not found any devices`，但窗口仍能打开并加载页面。原因未调查；不要认为 GPU 路径已经验证。
- **Linux CLI 启动器已过时。** `apps/desktop/cli/dsh-linux` 仍指向 `resources/app.asar/dsh/node_modules/@deepseek-ai/dsh-desktop-host/lib/cli.js`，但本分支已将树移到 `resources/dsh`。`app.asar` 中没有 `dsh` 目录。该启动器还使用 `ELECTRON_RUN_AS_NODE=1`，也就是会让 sharp 崩溃的模式。在修复并测试前，不要依赖内置的 `dsh-linux` 启动器。
- **AppImage 生成的 desktop entry 使用 `--no-sandbox`。** 它启动 `AppRun --no-sandbox %U`；未修改，也未做安全审查。
- **内置 Python `crypt` 需要 `libxcrypt-compat`。** 见上方前置要求。
- **产品分析默认开启。** 见下一小节。

### 产品分析

**Linux 构建默认发送 Desktop 产品分析数据。** 我用本地 OTLP collector 验证了这一点，而不仅是查看日志。

- Desktop 组合以 `enabled: true` 挂载 `@deepseek-ai/dsh-host-product-telemetry-otel` 和 `@deepseek-ai/dsh-client-product-analytics`，并以 `desktop` profile 为条件。Linux 构建使用该 profile。
- collector 端点默认为 `https://dsh-otel-collector.deepseeksvc.com/v1/logs`；服务名为 `deepseek-harness-desktop`。
- 将 `DSH_PRODUCT_ANALYTICS_OTLP_URL` 指向本地 sink 后，打包的 Linux 构建发送了 `desktop_app_launch` 和 `auth_page_view`。载荷包含 `os_name=linux`、`os_version=linux 7.2.11-200.fc44.x86_64`、`app_arch=x64`、CPU 型号与逻辑核数、总内存和 `app_version=0.2.1-rc.2`。登录账户时会包含身份字段；该次运行未登录。
- Linux 移植没有新增 Linux 专属元数据：`os_arch`、`os_build`、`os_version_full` 和 `cpu_physical_core_count` 在 `missing_fields` 中报告为 `unsupported`。
- 此构建**没有面向用户的分析开关**。`DSH_PRODUCT_ANALYTICS_OTLP_URL` 只重定向导出目的地，不是禁用开关。我没有找到或验证受支持的关闭采集方式。如果无法接受，请不要运行此构建，或在网络层屏蔽 `dsh-otel-collector.deepseeksvc.com`（此处未测试）。
- 我验证的是发送到本地 sink，而不是生产 collector。

这是继承的 Desktop 行为，不是本 Linux 分支新增的。之所以特别说明，是因为非官方 Linux 构建不应悄悄发送遥测数据。

### 免责声明

这是个人、非官方移植。它与 DeepSeek 没有隶属、背书或支持关系。“DeepSeek”和“DeepSeek Harness”仅用于描述本分支修改的软件。使用风险自负，请校验哈希，不要将这些产物作为官方发布展示。

<a id="run"></a>

## 运行

### 通过 `npm` 运行

安装 `Node.js`，然后运行：

```sh
npx @deepseek-ai/dsh web
```

该命令默认会在 `http://127.0.0.1:3080` 启动 Web UI，本机启动时还会用默认浏览器打开页面。通过 SSH 启动时只打印宿主机 URL，因为本地转发地址由 SSH 客户端或编辑器持有。传入 `--no-open` 可仅运行服务器而不打开浏览器。详见 [Web UI 指南](docs/user/guide/index.zh.md)。

<a id="run-from-source"></a>

### 从源码运行

如需从仓库源码运行：

```sh
git clone https://github.com/deepseek-ai/deepseek-harness.git
cd deepseek-harness
pnpm install
pnpm run build
pnpm dsh web
```

`pnpm run build` 会准备仓库产物。`pnpm dsh web` 会直接使用这些已构建产物，不会重新构建。

## 社区与支持

- 通过 [GitHub Discussions](https://github.com/deepseek-ai/deepseek-harness/discussions) 提交反馈或 bug 报告。
- 为你的插件仓库添加 [`dsh-plugin`](https://github.com/topics/dsh-plugin) 话题，便于被发现。
- 欢迎加入 DeepSeek Harness 企微群！扫描下方二维码填写入群问卷，小助手会定期发送入群邀请。

<table>
  <thead>
    <tr>
      <th align="center">入群问卷</th>
      <th align="center">微信公众号</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td align="center"><a href="https://trtgsjkv6r.feishu.cn/share/base/form/shrcnIt5twSVdLGD52KJBckGCgg"><img src="https://cdn.deepseek.com/harness/readme/community-wecom-survey.png" alt="DeepSeek Harness 入群问卷二维码" width="180" height="180"></a></td>
      <td align="center"><img src="https://cdn.deepseek.com/harness/readme/community-wechat-official-account.png" alt="DeepSeek Harness 团队微信公众号二维码" width="180" height="180"></td>
    </tr>
  </tbody>
</table>

## 参与贡献

参见 [CONTRIBUTING.md](CONTRIBUTING.zh.md)。

## 开发

请先阅读[开发指南](docs/development.zh.md)与[架构文档](docs/architecture.zh.md)。

`pnpm run dev:web` 会在一个终端里完成构建、启动，并在源码修改时重建 client bundle；`make help` 列出 Web 与 Desktop 对应的 Make target。完整表格见开发指南的「应用命令」一节。

面向 agent：请遵循 [AGENTS.md](AGENTS.md)。

## 引用

```bibtex
@misc{deepseek-harness2026,
  title={DeepSeek Harness: Everything is a Plugin},
  author={DeepSeek-AI},
  year={2026},
  publisher={GitHub},
  howpublished={\url{https://github.com/deepseek-ai/deepseek-harness}},
}
```

## 许可证

[MIT](LICENSE)

第三方依赖及其许可证见 [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)。
