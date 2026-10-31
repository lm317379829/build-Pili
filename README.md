# build-Pili

这是一个用于自动同步并构建 `PiliPlus` Android APK 的仓库。它不直接维护完整的上游应用源码，而是通过 GitHub Actions 自动：

- 监控上游 `bggRGjQaUbCoE/PiliPlus` 的最新提交
- 拉取最新源码
- 应用必要的补丁与自定义配置
- 构建 Android Release APK
- 使用固定签名证书进行重签名
- 发布到 GitHub Releases
- 保留最近若干个版本并清理旧构建记录

本仓库的目标是让用户能持续获取一个“带自定义图标、固定应用名、更新检查指向本仓库”的 PiliPlus APK 版本。

## 项目定位

这个项目属于“自动化打包仓库”，本质上是上游 `PiliPlus` 的持续构建与定制封装。其核心用途是：

- 跟踪上游项目更新
- 自动编译新版本 APK
- 替换应用图标和启动界面
- 修改应用名称为“哔哩哔哩”
- 让更新检查接口指向当前仓库，而不是上游仓库
- 生成可直接分发的安装包

## 工作原理

GitHub Actions 会在以下条件下执行构建：

- 每天定时检查上游更新
- 也支持手动触发

流程大致如下：

1. 读取上游 `PiliPlus` 最新 `main` 分支提交 SHA
2. 与本仓库记录的 `.last_build_sha` 对比
3. 若有更新，则拉取上游源码到 `pilipalus-source`
4. 安装 Java 17、Flutter SDK
5. 执行 `flutter pub get` 获取依赖
6. 应用上游补丁脚本 `patch.ps1`
7. 替换应用图标、启动背景和名称
8. 修改更新检查 URL，指向当前仓库
9. 注入编译时间和提交哈希
10. 使用本地固定的 `release.jks` 进行 APK 重签名
11. 构建 `arm64-v8a` 版 APK
12. 创建 GitHub Release 并上传 APK
13. 清理旧 release 与旧 workflow run
14. 更新 `.last_build_sha` 记录

## 仓库结构

```text
build-Pili/
├── .github/
│   └── workflows/
│       └── main.yml
├── .last_build_sha
├── README.md
├── assets/
│   └── icon.png
├── release.jks
└── (自动构建时生成的 APK / 临时目录)
```

### 关键文件说明

- `.github/workflows/main.yml`：自动构建流水线
- `.last_build_sha`：记录上一次已构建的上游提交 SHA
- `assets/icon.png`：自定义应用图标
- `release.jks`：固定签名证书，用于稳定重签名 APK

## 功能特性

### 1. 自动跟踪上游更新

工作流会向 GitHub API 请求 `PiliPlus` 最新提交，并与 `.last_build_sha` 比较。如果没有新提交，则跳过构建；若有新提交，则触发编译流程。

### 2. 自动构建 APK

当前构建目标为：

- 平台：Android
- 架构：arm64-v8a
- 产物：Release APK

GitHub Actions 会在虚拟环境中安装 Flutter、Java 等依赖，并完成编译。

### 3. 自定义界面和品牌

构建过程会对应用做以下定制：

- 使用自定义图标：`assets/icon.png`
- 设置启动白底界面
- 修改应用名称为：`哔哩哔哩`
- 跳过系统默认黑底启动界面问题
- 调整自适应图标相关资源

### 4. 固定签名证书

为了保证 APK 具有稳定签名，仓库中提供 `release.jks`。如果不存在，流程会自动生成并提交回仓库，确保后续构建复用同一证书。

### 5. 更新检查重定向

这个项目会修改 `PiliPlus` 原本的更新检查 API，指向本仓库的 Releases，确保本仓库中的版本可以正常检查更新。

### 6. 版本清理

构建成功后会：

- 发布对应 GitHub Release
- 保留最近 `KEEP_COUNT` 个版本
- 删除旧的 Release 和过期 Action Run

## GitHub Actions 说明

工作流入口：

- `.github/workflows/main.yml`

主要触发方式：

- 定时任务（每天检查一次）
- 手动执行（`workflow_dispatch`）

流水线的权限包括：

- `contents: write`
- `actions: write`

用于：

- 提交 `.last_build_sha`
- 发布 Release
- 删除旧构建记录

## 使用方式

### 1. 直接下载 Release

访问当前仓库的 Releases 页面，下载最新构建的 APK 即可。

### 2. 手动触发构建

1. 进入 GitHub Actions 页面
2. 选择 `Build PiliPlus APK`
3. 点击 `Run workflow`
4. 等待编译完成并发布新版本

## 版本管理

当前流程会生成带版本号的 APK，例如：

```text
哔哩哔哩_vX.Y.Z_arm64-v8a.apk
```

并自动使用 GitHub Release 发布，标签名通常形如：

```text
vX.Y.Z
```

## 注意事项

- 此仓库依赖上游 `PiliPlus` 的持续更新，构建内容会随着上游提交变化而更新。
- 构建过程中使用了固定签名证书，建议不要随意替换 `release.jks`。
- 本仓库的 APK 产物是按 `arm64-v8a` 架构构建的，适合大多数现代 Android 设备。
- 如果你需要指定其他架构或额外功能，需自行修改 workflow。

## 免责声明

本项目仅用于自动化构建与分发，生产内容来自上游 `PiliPlus` 项目。请遵守上游项目的许可协议、使用条款和相关要求。

## 参考来源

- 上游仓库：`bggRGjQaUbCoE/PiliPlus`
- 工作流文件：`.github/workflows/main.yml`
- 图标资源：`assets/icon.png`
- 固定签名证书：`release.jks`

如果你需要，我还可以继续把这个 README 改成更偏“GitHub 首页展示版”或“部署/使用说明版”，让它看起来更像正式项目文档。