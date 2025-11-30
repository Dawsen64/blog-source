---
title: Obsidian自动化图床
date: 2025-11-30
updated: 2025-11-30
---



> **版本**: 1.0 (Auditor Approved)
> **目标**: 实现 "Obsidian 粘贴 -> PicGo 自动接管 -> 上传 GitHub -> 返回 jsDelivr 加速链接" 的全自动闭环。
> **风险等级**: 中 (依赖第三方服务，需注意隐私)

---

## 🛑 核心前置原则 (Read First)
1.  **隐私红线**：严禁上传包含个人隐私（身份证、密码、住址）的截图。GitHub 仓库是 **Public（公开）** 的。
2.  **数据备份**：建议定期将 GitHub 仓库 `git clone` 到本地备份，防止账号封禁导致图片丢失。
3.  **网络依赖**：此方案依赖 GitHub API 可访问性。

---

## 阶段一：基础设施搭建 (GitHub)

### 1. 创建仓库
- [ ] 登录 GitHub，新建仓库（例如命名为 `blog-assets`）。
- [ ] **权限设置**：必须设为 **Public** (公开)，否则 CDN 无法抓取图片。

### 2. 获取访问令牌 (Token) ⚠️ 高危易错点
*请严格按照此路径，不要凭直觉点击*
1.  点击右上角头像 -> **Settings**。
2.  左侧栏滑到底部 -> **Developer settings**。
3.  左侧点击 **Personal access tokens** -> **Tokens (classic)**。
4.  点击 **Generate new token (classic)**。
    - **Note**: `PicGo-Blog`
    - **Expiration**: `No expiration` (建议永不过期，避免频繁更换)
    - **Scopes**: 仅勾选 **`repo`** (Full control of private repositories)
5.  **复制 Token**：以 `ghp_` 开头的字符串。*(只显示一次，必须立即保存)*

---

## 阶段二：中间件配置 (PicGo)

### 1. 基础设置
- [ ] 下载并安装 PicGo (建议 2.3.1+ 版本)。
- [ ] **图床设置** -> **GitHub 图床**：
    - **仓库名**: `你的用户名/仓库名` (例: `audit/blog-assets`)
    - **分支**: `main` (或 `master`，看你仓库实际情况)
    - **Token**: 填入阶段一获取的 `ghp_...`
    - **自定义域名 (CDN加速核心)**: 
      ```
      [https://cdn.jsdelivr.net/gh/你的用户名/仓库名@main](https://cdn.jsdelivr.net/gh/你的用户名/仓库名@main)
      ```
      *(注意：不填此项在国内无法加载图片，务必严格检查格式)*

### 2. 自动化优化
- [ ] **PicGo 设置** -> 开启 **时间戳重命名** (防止文件名冲突)。
- [ ] **PicGo 设置** -> 开启 **Server** (确保监听端口 `36677`，默认通常已开启)。
- [ ] **PicGo 设置** -> 开启 **开机自启** (推荐方案，最稳定)。

---

## 阶段三：前端集成 (Obsidian)

### 1. 安装核心插件
- [ ] 关闭安全模式，进入 **社区插件 (Community plugins)** 市场。
- [ ] 搜索并安装 **Image Auto Upload Plugin** (作者: Taotao)。
- [ ] 启用插件。

### 2. 插件配置
- [ ] 进入插件设置页：
    - **Default uploader**: 选择 `PicGo(app)`
    - **PicGo server**: 保持默认 `http://127.0.0.1:36677/upload`
    - **Delete original file after upload**: **开启** (保持本地文件夹整洁)

---

## 阶段四：联动自动化 (可选优化)
*如果你不想让 PicGo 开机自启，希望“打开 Obsidian 时自动唤醒 PicGo”，请配置此步。*

1.  安装 **Shell Commands** 插件。
2.  **Settings -> Shell Commands -> New command**:
    - **Windows**: `start "" "C:\你的安装路径\PicGo.exe"` (需替换真实路径)
    - **macOS**: `open -a "PicGo"`
3.  点击命令旁的齿轮图标 -> **Events** -> 开启 **Startup**。
4.  **警告**：Obsidian 启动后请等待 **5-8秒** 让 PicGo 完成初始化，否则立即粘贴会失败。

---

## ✅ 最终验收清单 (Checklist)
1.  [ ] PicGo 正在运行（状态栏可见）。
2.  [ ] 在 Obsidian 随便截张图，按下 `Ctrl+V`。
3.  [ ] 图片显示 "Uploading..." 随后变为 Markdown 链接。
4.  [ ] 链接格式为 `https://cdn.jsdelivr.net/...`。
5.  [ ] 在浏览器打开该链接，图片能正常显示。

## 🆘 故障排除
* **上传失败 (Failed)**:
    * 检查 PicGo 日志 (PicGo 设置 -> 设置日志文件 -> 打开)。
    * 如果是 `403`：Token 权限不对，或者仓库不是 Public。
    * 如果是 `404`：仓库名写错，或者分支名写错 (`main` vs `master`)。
    * 如果是 `Network Error`：端口 `36677` 没开（PicGo 没启动）。