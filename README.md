# 📦 DGDrop (DageDrop)

> 零信任加密投递箱 | 建立在 Nostr 协议之上的端到端加密（E2EE）物理粉碎网盘

[![Deploy to GitHub Pages](https://github.com/dgdrop/dgdrop.github.io/actions/workflows/pages/pages-build-deployment/badge.svg)](https://dgdrop.github.io)
[![Protocol](https://img.shields.io/badge/Protocol-Nostr-purple.svg)](https://nostr.com/)
[![Encryption](https://img.shields.io/badge/Encryption-AES--GCM--256-blue.svg)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

🌐 **Live Demo (纯静态零信任客户端):** [https://dgdrop.github.io](https://dgdrop.github.io)

---

## 💡 核心理念：这不是一个传统的“云盘”

在传统的云盘中，服务器拥有至高无上的权力，它们知道你存了什么、文件名是什么、甚至可以通过计算 Hash 来进行“秒传”和“审查”。

**DGDrop 彻底颠覆了这种模式。**
这套系统采用了严格的 **“厚客户端 + 物理致盲服务端”** 零信任架构：
1. **服务器对数据一无所知**：所有的加密、分块、压缩都在浏览器（客户端）本地完成。服务器硬盘里存放的，只是一堆毫无逻辑的 AES-GCM 二进制密文碎片。
2. **拒绝“分享链接”**：摒弃了极不安全的“生成分享外链+提取码”模式。DGDrop 基于 Nostr 的椭圆曲线非对称加密（NIP-44/04）进行密钥交换。除了指定的接收人，没有任何人（包括网盘管理员和黑客）能解开文件。

---

## 🛡️ 核心特性 (Features)

### 1. 绝对的物理致盲 (Zero-Knowledge)
所有的文件在离开你的设备前，都会被前端 JSZip 打包并使用 `WebCrypto API` 生成的随机 256 位密钥进行 `AES-GCM` 加密。服务端永远无法获取文件类型、大小、名称或内容。

### 2. 军工级不可逆粉碎 (Atomic Shredding)
文件拥有“授权寿命”与“物理寿命”双轨制约。一旦用户主动删除，或达到设定寿命（如 30 分钟），服务端将直接在底层硬盘执行原子级实体抹除（`os.remove`），不留任何数字痕迹。

### 3. NIP-98 防重放鉴权 (Anti-Replay Auth)
客户端与后端的每一次 HTTP 交互（包括二进制碎片的上传/下载），均受 Nostr NIP-98 签名协议保护。完美抵御中间人攻击 (MITM) 与重放攻击。

### 4. 极端场景护城河 (Security Moats)
- **本地私钥死锁**：你的 Nostr 私钥 (`nsec`) 仅留存本地，且被 PBKDF2 (60万次迭代) + AES 强加密。
- **内存防嗅探**：一旦闲置超时、刷新页面或点击注销，内存中的密钥变量会被瞬间剥离销毁。
- **防 OOM 并发流**：100% 纯前端实现的“锯齿状内存调度”，支持在浏览器内对数百 MB 的加密碎片进行流水线解密、打包与下载，即使在低端手机上也不会崩溃。

---

## 🚀 快速开始 (Quick Start)

客户端是 **100% 纯静态的 HTML/JS**，没有任何后端代码，你可以直接通过 GitHub Pages 访问并使用：

👉 **打开终端：[https://dgdrop.github.io](https://dgdrop.github.io)**

### 如何使用？
1. **生成/导入账号**：在登录界面点击“生成全新账号”（系统会在本地为你生成一套 Nostr 密钥对）。
2. **设置本地安全密码**：这是用来在你的浏览器本地加密保护私钥的，请务必牢记，遗失无法找回。
3. **开始投递**：拖拽照片、视频或任意文件进入页面，设定“保留寿命”，文件将被加密并碎裂上传至 Relay 中继服务器。
4. **安全分享**：在“通讯录”中添加好友的 Nostr 公钥（`npub...`）。选中文件点击分享，系统会在本地用目标好友的公钥加密 AES 密钥，建立不可嗅探的共享通道。

---

## 🏗️ 架构解析 (Architecture)

本仓库仅包含 **前端静态客户端**。DGDrop 完整架构由两部分组成：

1. **Client (本仓库)**: 纯前端。负责生成 AES 密钥、分块加密、组装 Nostr 30400/30401 事件，以及向服务端发送 HTTP 请求。
2. **Relay Backend**: 这是一个轻量级的 Python (FastAPI + SQLite) 服务端。负责 WebSocket 信令中转、二进制碎片的落盘存储、定时垃圾回收（GC）以及用户 Quota 配额管理。（*如果您希望自行部署私有中继节点，请获取 Relay 源码*）。

### 交互协议简述：
* **`Kind 30400`**: 文件的元数据（包含加密后的 AES 密钥、IV、文件名），只对上传者自己加密。
* **`Kind 30401`**: 分发授权凭证。当把文件分享给 B 时，单独用 B 的公钥将 AES 密钥加密并打上 B 的 `p` 标签。
* **`Kind 5`**: 撤回指令。发送此事件即可物理删除硬盘上的文件，或销毁特定的 30401 授权。

---

## 🔐 隐私声明与安全性 (Disclaimer)

* **纯静态应用**：`dgdrop.github.io` 是由 GitHub 托管的纯静态页面，我们无法、也没有代码逻辑去收集您的私钥或明文数据。
* **安全性保证**：只要您保证“本地安全密码”足够复杂且未被泄露，即使服务器管理员拔下硬盘，或者黑客拖走了完整的 SQLite 数据库与存储目录，他们也只能得到一堆废弃的随机字节。
* **免责条款**：本项目仅供密码学与去中心化技术研究使用。请遵循您所在国家或地区的法律法规，严禁利用本工具传输违法违规内容。

---

## 📜 开源协议 (License)

本项目采用 [MIT License](LICENSE) 开源协议。欢迎提交 PR、Fork 以及探讨 Nostr 协议在文件加密共享领域的更多可能。

> 📖 **普通用户必读**：如果你想知道为什么现在的网盘不安全，以及 DGDrop 到底解决了什么痛点，请阅读我们的 👉 [**产品宣言 (WHY.md)**](./WHY.md)
