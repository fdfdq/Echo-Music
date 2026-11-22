# 回声音乐 · Echo-Music

一款 Material Design 3 风格的音乐播放器，用**单个 HTML 文件**实现全部功能：在线搜索、网易云曲库播放、滚动歌词、歌单导入与收藏、本地音乐包、歌手页、锁屏媒体控制。

项目提供两个 Web 版本（手机 / 电脑）和一个安卓原生桥（WebCatX 宿主），全部为个人学习项目。

* 手机版：`Echo-Music-phone.html`（底部三栏导航，平板自动改为左侧竖排导航 + 封面与歌词并列）
* 电脑版：`Echo-Music-pc.html`（弹窗居中、播放页左封面右歌词、鼠标与键盘交互）

\---

## 一、功能一览

**在线部分**

* 多源搜索：三个搜索源串行兜底，任一可用即可出结果
* 播放直链：按 标准 → 较高 → 极高 → 无损 → Hi-Res 逐档申请，无源自动降级
* 歌单：粘贴网易云歌单链接或 ID 即可导入、收藏、本地缓存
* 榜单与推荐：首页展示推荐歌单与多个排行榜，数据来自第三方接口
* 歌手页：一次展示歌手全部歌曲，专辑横排随列表一起滚动，点进专辑才切换为专辑视图并标明所属专辑
* 在线匹配：长按本地歌曲按歌名搜全网，选中后把音频 + 封面 + 歌词打成 ZIP 一次性下载（文件名与歌曲同名）

**歌词**

* MD3 风格滚动歌词，当前行放大、高亮胶囊背景
* 双语歌词同时显示，当前行译文同样放大
* 支持外挂 LRC（本地音乐包内的 `.lrc` 会被识别）

**本地音乐**

* 浏览器无法直接扫描存储，改为两种导入：文件多选、ZIP 音乐包
* ZIP 包自动识别音频、封面图片、歌词；封面名或包名与音频同名即视为一个"音乐包"，合并成一首带封面的歌
* 封面取色驱动整个界面的主题色
* 长按歌曲可在线匹配或从列表中删除（不删除源文件）

**界面与交互**

* 首次打开是四步注册引导（填写昵称、选主题色等），完成后提示"仅供学习交流，请于安装 24 小时后删除"
* 底部三栏（推荐 / 本地音乐 / 我的）带滑动切换动画，返回键在三个 tab 之间不触发后退
* 全部设置收拢进"我的 → 设置"单入口
* 播放页：封面滑动切歌、双击收藏、上滑进歌词
* 大屏（≥720px）与横屏：内容铺满、不留边距

**系统集成**

* Web Media Session：网页内即可向浏览器/系统暴露播放状态与上一首、下一首、播放暂停、进度控制
* WebCatX 原生桥：Android 前台媒体服务 + MediaStyle 通知，锁屏可控；原生还可代发 HTTP 请求，绕开网易云接口的跨域限制

\---

## 二、技术栈

|层面|选用|
|-|-|
|开发语言|HTML5、CSS3、JavaScript（原生 ES Module，无框架、无构建、无打包器）|
|界面|Material Design 3 设计语言；CSS 自定义属性做主题 token；内联 SVG 图标（Material Symbols 造型）|
|模块加载|运行时 `import()` 从 CDN 按需拉取纯前端依赖，因此交付物始终是单文件|
|音频|HTML5 `<audio>` + Blob URL|
|本地存储|IndexedDB（音频与封面二进制）、localStorage（播放状态、设置、引导标记）|
|浏览器 API|Media Session、Fetch、File API、Pointer Events、matchMedia、Canvas 取色、Web Animations / CSS Transition|
|安卓原生|Kotlin + Android SDK 平台 API（MediaSession、Foreground Service、Notification MediaStyle、WebView JSBridge）|

> 没有引入 React / Vue 等框架，全部逻辑手写在同一个 `<script>` 内，约 6000 行；改样式改脚本都只动这一个文件。

\---

## 三、依赖的开源项目

|项目|用途|许可|引入方式|
|-|-|-|-|
|[material](https://github.com/floor/material)（md3.io）3.15.0|MD3 组件与主题系统，`theme/image` 负责从封面取色生成配色|MIT|`import()` 动态加载，双 CDN 镜像|
|[fflate](https://github.com/101arrowz/fflate) 0.8.2|浏览器端 ZIP 解压，用于音乐包与在线匹配的打包|MIT|`import()` 动态加载|
|[Meting](https://github.com/metowolf/Meting)|歌曲元数据、歌词、歌单接口协议；项目使用其公共实例|MIT|HTTP 接口|
|[NeteaseCloudMusicApi](https://github.com/Binaryify/NeteaseCloudMusicApi)|网易云接口格式参考，第三方实例按其协议实现|MIT|HTTP 接口|
|[AMLL](https://amll.dev/)（Apple Music like Lyrics）|逐行滚动的歌词排版思路参考|AGPL-3.0|**仅作视觉参考，未引入其任何代码**|
|[Material Design 3](https://m3.material.io/)|设计规范与 token 命名体系|Apache-2.0（规范文档）|设计参考|
|Sora / Noto Sans SC|界面字体|SIL OFL 1.1|经字体镜像 CDN 加载|
|[WebCatX](https://webcat.top/)|安卓 WebView 宿主与 `CatNative` 桥|第三方 App，非本项目依赖|宿主环境|

安卓原生桥**不依赖任何第三方库**，只用 Android 平台 API（API 26+），无 AndroidX、无 OkHttp，可直接放进宿主工程。

\---

## 四、使用的外部接口

|用途|接口|备注|
|-|-|-|
|音频直链|`api.byfuns.top`|按 `level` 逐档请求，返回非 http(s) 即视为无源并降级|
|元数据 / 歌词 / 歌单|`api.injahow.cn/meting`|Meting 协议，`type=song / pic / lyric / playlist`|
|搜索|`api.vkeys.cn`、`music-api.gdstudio.xyz`、`api.i-meto.com`|三源串行兜底|
|歌单与榜单|`music-api.gdstudio.xyz`、Meting `playlist`|榜单 ID 固定，随上游变动可能失效|
|歌手信息|`music.163.com/api/artist/...`|无 CORS 头，仅原生桥可调用；网页版退化为"按歌手名聚合"|
|封面图|`p\\\\\\\*.music.126.net`|展示层请求 600×600，下载时取原图|

这些均为第三方公共服务，可用性、稳定性与内容版权都不由本项目控制。

\---

## 五、目录结构

```
md3-music-player/
├── MD3音乐播放器.html            手机版（单文件）
├── MD3音乐播放器-电脑版.html      电脑版（单文件，html.desk 皮肤）
├── MD3音乐播放器-源码.txt         手机版源码
├── MD3音乐播放器-电脑版-源码.txt   电脑版源码
├── README.md
└── webcatx-native/
    ├── CatMediaBridge.kt          MediaSession + 前台服务 + 通知 + JS 桥 + HTTP 代理
    ├── AndroidManifest.patch.xml  权限与服务声明
    ├── web-native.js             网页端参考桥接脚本
    └── README.md                 接入步骤
```

\---

## 六、运行方式

**网页版**：直接双击 HTML 打开即可，无需服务器。首次会进入注册引导，完成后进入首页。

**电脑版**：浏览器窗口建议 ≥1024px，进入桌面布局；弹窗居中显示，`Esc` 关闭弹窗、空格播放暂停、方向键快退快进。

**安卓端（WebCatX）**：

1. 把 `CatMediaBridge.kt` 放入宿主工程，包名与 `AndroidManifest.patch.xml` 中 `.echo.media.EchoMediaService` 对应
2. 按 patch 文件补齐 `POST\\\\\\\_NOTIFICATIONS`、`FOREGROUND\\\\\\\_SERVICE`、`FOREGROUND\\\\\\\_SERVICE\\\\\\\_MEDIA\\\\\\\_PLAYBACK`、`WAKE\\\\\\\_LOCK` 权限并注册 service（`foregroundServiceType="mediaPlayback"`）
3. WebView 创建后调用一次 `CatMediaBridge.attach(webView)`
4. 页面会自动检测 `window.CatMedia` 并向通知栏推送 `{title, artist, cover, state, position, duration}`；通知按钮回控网页播放器

\---

## 七、数据存储

|存储|键|内容|
|-|-|-|
|localStorage|`echo-m3-player/v4`|播放队列、当前曲目、音量、音质、主题色、收藏歌单|
|localStorage|`echo-m3-player/onboard/v1`|引导完成标记、昵称、初始主题色|
|IndexedDB|`echo-m3-player/audio` / `kv`|`blob:` 在线缓存、`zipaudio:` / `zipcover:` / `ziplyric:` 本地包、`matchaudio:` / `matchcover:` / `matchlyric:` 匹配结果|

所有数据只存在本机，不上传任何服务器。

\---

## 八、已知限制

* 浏览器不能扫描 `/storage/emulated/0/Music/`，本地音乐必须通过文件多选或 ZIP 包导入（这是浏览器沙箱的硬限制，原生壳内同样受 WebView 约束）
* `fflate` 与 `material` 在运行时从 CDN 拉取，纯离线环境下 ZIP 导入与组件增强会不可用
* 部分第三方歌单/榜单接口在 `file://` 场景因 CORS 受限，首页推荐可能为空，需经原生桥或本地服务器访问
* 电脑版沿用同一套 `file://` 数据源，故首页空态与手机版一致，不是桌面皮肤问题

\---

## 九、声明

本项目仅用于前端技术学习与交流，不提供任何音源存储与分发，所有音频均由第三方接口实时解析。

请支持正版，尊重音乐人。**请于安装后 24 小时内删除本软件。**

Echo-Music · 2026

