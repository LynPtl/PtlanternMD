# 使用 Cloudflare R2 + ShareX 搭建个人/团队专属“永久”图床

本指南旨在提供一个从零开始，利用 Cloudflare R2 的免费套餐和 ShareX 的强大功能，搭建一个高性能、高可靠性、几乎零成本且数据完全由自己掌控的图床的完整流程。

**最终效果**：通过一个快捷键截图，图片自动上传到你的专属存储空间，或手动上传自己的图片，自动将 Markdown 格式的链接复制到剪贴板，实现无缝写作。

---

## 目录
1.  [第一部分：配置 Cloudflare R2 (云端存储)](#第一部分配置-cloudflare-r2-云端存储)
2.  [第二部分：配置 ShareX (桌面客户端)](#第二部分配置-sharex-桌面客户端)
3.  [第三部分：优化 ShareX 工作流](#第三部分优化-sharex-工作流)
4.  [常见问题 (F.A.Q) 与排错](#常见问题-faq-与排错)

---

## 第一部分：配置 Cloudflare R2 (云端存储)

首先来到Cloudflare开通R2存储。
![](https://pub-85d4dcece16844bf8290aa4b33608ccd.r2.dev/ShareX/2025/09/%E5%9B%BE%E7%89%87_20250927143514.png)

### 1.1 创建 R2 存储桶 (Bucket)

存储桶是存放你所有图片的容器。

1.  登录 Cloudflare 控制台，在左侧菜单进入 **R2**。
2.  点击 **Create bucket (创建存储桶)**。
3.  **Bucket name**: 输入一个全局唯一的存储桶名称 (例如 `your-org-images-2025`)。
4.  **Location**: 保持默认的 **Automatic**。
5.  点击 **Create bucket**。

![](https://pub-85d4dcece16844bf8290aa4b33608ccd.r2.dev/ShareX/2025/09/xrRAVMB2Sz.png)

### 1.2 开放存储桶的公开访问权限

为了让上传的图片能被外部访问，需要开启公共访问。

1.  进入你刚创建的存储桶，点击顶部的 **Settings (设置)** 选项卡。
![1r0AU3aWkD.png](https://pub-85d4dcece16844bf8290aa4b33608ccd.r2.dev/ShareX/2025/09/1r0AU3aWkD.png)
2.  在 **下方Public Development URL** 部分，点击右侧的 **Enable**。
3.  输入确认信息。
![fgc4amk7S7.png](https://pub-85d4dcece16844bf8290aa4b33608ccd.r2.dev/ShareX/2025/09/fgc4amk7S7.png)
4.  **记下**这里显示的 `https://pub-....r2.dev` 格式的 URL，这是你的公共访问域名。
![RmQwxSxpLi.png](https://pub-85d4dcece16844bf8290aa4b33608ccd.r2.dev/ShareX/2025/09/RmQwxSxpLi.png)

### 1.3 创建用于上传的 API Token

API Token 是让 ShareX 有权限上传文件到 R2 的“钥匙”。

1.  回到 R2 的主页（概览页面），点击右上角的 **Manage API Tokens (管理 R2 API 令牌)**。
![CTzhiiSl04.png](https://pub-85d4dcece16844bf8290aa4b33608ccd.r2.dev/ShareX/2025/09/CTzhiiSl04.png)
1.  点击 **Create Account API token (创建 API 令牌)**。
![FBEzXXohz7.png](https://pub-85d4dcece16844bf8290aa4b33608ccd.r2.dev/ShareX/2025/09/FBEzXXohz7.png)
2.  **Permissions (权限)**: **务必选择 `Object Read & Write` (对象读和写)**。这是最关键的一步，只读权限会导致上传失败。
![xB4pYQOeEI.png](https://pub-85d4dcece16844bf8290aa4b33608ccd.r2.dev/ShareX/2025/09/xB4pYQOeEI.png)
3.  点击 **Create API token**。
4.  **⚠️ 立即复制并保存！** 页面上会显示 `Access Key ID` 和 `Secret Access Key`。**这两个密钥只会出现这一次**，请立刻将它们复制并粘贴到一个安全的地方。最下方的Default endpoints链接也需要保存一下。
![kg9E8tEozI.png](https://pub-85d4dcece16844bf8290aa4b33608ccd.r2.dev/ShareX/2025/09/kg9E8tEozI.png)

---

## 第二部分：配置 ShareX (桌面客户端)

### 2.1 下载并安装 ShareX

从官网下载最新版本：[https://getsharex.com/](https://getsharex.com/)

### 2.2 配置 S3 上传目标

这是整个配置过程的核心。

1.  打开 ShareX，在主界面点击 `Destination settings...`。
2.  在弹出的窗口左侧选择 `Amazon S3`，在右侧输入你的配置信息。
3.  按照下表精确填写你的 R2 信息：

| ShareX 设置项             | 填写内容                                                               | 说明                                                                      |
| ------------------------- | ---------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| **Access key ID** | 粘贴你保存的 `Access Key ID`                                             |                                                                           |
| **Secret access key** | 粘贴你保存的 `Secret Access Key`                                           |                                                                           |
| **Region** | 留空                                                                 |                                                                           |
| **Endpoints** | 留空                       |        
| **Endpoint** | `https://<你的AccountID>.r2.cloudflarestorage.com`                       |  如果你在刚才保存过可以直接复制，切记最后没有`/`。                                           |
| **Bucket name** | 你的 R2 存储桶名称 (例如 `your-org-images-2025`)                         |                                                                           |
| **Upload path** | `img/%y/%mo/%d/`                                                         | 按 `img/年/月/日/` 格式存放图片，有助于管理。ShareX 的变量格式是 `%`。（也可自定义）       |
| **Use custom domain** | **勾选此项** |                                                                           |
| (自定义域名输入框)        | `https://<你记下的r2.dev公共URL>`                                          | **注意**：这里只填刚才你记下的Public Development URL，不要加后面的 `$key$`。ShareX 新版本会自动处理。 |


#### 2.2.1 关键的高级 (Advanced) 设置

在 S3 配置窗口的最下方，找到 **Advanced** 区域，进行如下设置：

* `Set public-read ACL on file`: **必须取消勾选**。R2 不支持此功能，勾选会导致 `403 Forbidden` 错误。
* `Use path style request`: **必须勾选**。R2 需要这种格式的请求 URL。

### 2.3 将 S3 设置为默认图片上传器

1.  回到 ShareX 主界面。
2.  点击 `Destinations` -> `Image uploader` -> `File uploader` -> 选择 `Amazon S3`。
![Code_nqStY1UhqR.png](https://pub-85d4dcece16844bf8290aa4b33608ccd.r2.dev/ShareX/2025/09/Code_nqStY1UhqR.png)

---

## 第三部分：优化 ShareX 工作流

### 3.1 自动复制 Markdown 链接

1.  在 ShareX 主界面，点击 `Task settings...`。
2.  在弹出的窗口左侧选择下方的 **`Advanced`**。
![ShareX_ZWFVlZZu0W.png](https://pub-85d4dcece16844bf8290aa4b33608ccd.r2.dev/ShareX/2025/09/ShareX_ZWFVlZZu0W.png)
1.  接着，点击 `After upload` 下方的 `ClipboardContentFormat`。
2.  将里面的内容替换为markdown格式 `![$filename]($result)` 。

### 3.2 打开自动复制到剪切板 

1.  在主页面的 `After upload tasks` -> 右侧选择 `Copy URL to clipboard`。
![ShareX_zf6qftjnu6.png](https://pub-85d4dcece16844bf8290aa4b33608ccd.r2.dev/ShareX/2025/09/ShareX_zf6qftjnu6.png)

### 3.3 修改热键

在主页面的 `Hotkey settings` 中可以修改热键，实现截图直接上传并复制markdown到剪贴板一条龙。

---

## 常见问题 (F.A.Q) 与排错

**Q1: 上传时报错 `(403) Forbidden`，怎么办？**  
**A1:** 这是最常见的问题，请检查以下两项 S3 高级设置：
1.  确保 **`Set public-read ACL on file`** **没有**被勾选。
2.  确保 **`Use path style request`** **已经**被勾选。
3.  如果依然报错，请重新生成一个**权限为 `Object Read & Write` 的 API Token** 并更新到 ShareX 中。

**Q2: 我不是想上传截图，我想上传我自己本地的别的图片**  
**A2:**
**使用左侧的upload选项卡**: 在选项卡内选择你需要的上传内容。
![ShareX_34TAgHco1T.png](https://pub-85d4dcece16844bf8290aa4b33608ccd.r2.dev/ShareX/2025/09/ShareX_34TAgHco1T.png)

**Q3: PicGo 为什么不能用？**  
**A3:** 在我的测试中，PicGo 的 S3 插件与 Cloudflare R2 存在一些兼容性问题，导致文件名和路径处理不正确。ShareX 的 S3 实现更标准，是目前更可靠的选择。