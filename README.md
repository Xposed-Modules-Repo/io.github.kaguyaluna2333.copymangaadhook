# CopyManga AdHook / 拷贝漫画去广告

[中文](#中文) | [English](#english)

## 中文

一个 LSPosed (Xposed) 模块，用于去除**拷贝漫画**（`com.copymanga.app`）安卓客户端的广告：开屏/插窗弹窗、横幅、点叉跳浏览器链路。仅做个人设备去广告，**不涉及**任何付费内容解锁。

### 工作原理（五层防御）

宿主为 Flutter + 360 加固应用，全部锚点经运行时解密验证、按需延迟挂载：

1. **YfKill**——yfanads（亿帆）广告 SDK 精准短路：7 个锚点（`YFAdsManager.init`、`ReflectionUtils.initSdk`、`YFAdInterstitialAds.showAds`、`YFAdSplashAds.showAds`、开屏 Activity finish、插件 `onMethodCall` 总闸），class-watch 重试覆盖加固延迟解密
2. **GdtKill**——优量汇（GDT）动态插件拦截：`ClassLoader.loadClass` 监听，命中 `gdt_plugin.jar`（DexClassLoader 动态加载）内的 `POFactoryImpl` 即短路
3. **SpGate**——频控屏障：广告频控时间戳强制 far-future；服务端广告配置缓存（cacheInitData）的间隔字段推至 int 上限
4. **框架层闸门**——广告 Activity 名单（132 精确类 + 18 个 SDK 包前缀）在 `Instrumentation.execStartActivity` 拒启 + `onCreate` 即 finish 双保险；Dialog 广告关闭键判定（资源 id 动态解析，版本漂移免疫）；广告页 WebView `loadUrl` 拦截；url_launcher 内置 WebView 封口
5. **AutoClose**——decor 轮询兜底：命中启发式签名后须确认广告标记（如 content-desc「廣告」）才隐藏，保护章节末页等正常 UI

### 要求

- 已 root 的设备 + **LSPosed**（Zygisk）
- 拷贝漫画客户端（锚点基于 2026-10 版本验证；宿主更新后广告若复现需重修锚点）

### 使用

1. 从 [Releases](../../releases) 下载安装 APK
2. LSPosed Manager 启用模块，作用域勾选 **拷贝漫画**（模块已预声明推荐作用域）
3. 强制停止拷贝漫画后重新打开

模块**不申请任何权限**，不收集任何数据。诊断开关（通道 trace、网络观测）默认关闭，可用模块 APK 内 `assets/config.json` 调整各闸门。

### 构建

```bash
bash build.sh   # 需要 Android SDK（build-tools + platforms）与 JDK
```

手工管线：javac --release 8 → d8 → aapt2 → zipalign → apksigner，产物 `dist/CopyMangaAdHook-*.apk`。

### 免责声明

仅供个人设备去广告使用，与拷贝漫画官方无关联。不做付费内容解锁、不做 DRM 绕过。宿主更新可能导致锚点失效，届时需重新适配。

---

## English

An LSPosed (Xposed) module that removes ads from the **CopyManga** (`com.copymanga.app`) Android app: splash/interstitial popups, banners, and the "close button jumps to browser" chains. Personal-device ad-blocking only — it does **not** unlock any paid content.

### How it works (five defense layers)

The host app is Flutter + 360-packed; all anchors were verified against runtime-decrypted classes and mounted lazily:

1. **YfKill** – short-circuits the yfanads SDK at 7 anchors (`YFAdsManager.init`, `ReflectionUtils.initSdk`, `YFAdInterstitialAds.showAds`, `YFAdSplashAds.showAds`, splash activity finish, plugin `onMethodCall` master switch), with class-watch retry for packer-delayed classes
2. **GdtKill** – blocks the GDT dynamic plugin factory via a `ClassLoader.loadClass` watch (hits `POFactoryImpl` inside `gdt_plugin.jar`, loaded by DexClassLoader)
3. **SpGate** – rewrites ad frequency-control timestamps to far-future and neutralizes the cached server ad config (intervals → int max)
4. **Framework gates** – ad Activity blocklist (132 exact classes + 18 SDK package prefixes) blocked at `Instrumentation.execStartActivity` plus finish-on-create; Dialog ad-close-id gate with dynamically resolved resource id; ad WebView `loadUrl` gate; url_launcher WebView kill
5. **AutoClose** – decor polling fallback: hides a container only after an ad marker is confirmed (e.g. content-desc "廣告"), protecting legitimate UI like chapter-end buttons

### Requirements

- Rooted device with **LSPosed** (Zygisk)
- CopyManga app (anchors verified against the 2026-10 build; host updates may shift them)

### Usage

1. Install the APK from [Releases](../../releases)
2. Enable the module in LSPosed Manager with scope **CopyManga (com.copymanga.app)** — recommended scope is predeclared
3. Force-stop the app and relaunch

The module requests **no permissions** and collects nothing. Diagnostic toggles (channel trace, network observer) default to off; every gate can be toggled via `assets/config.json` inside the module APK.

### Build

```bash
bash build.sh   # needs Android SDK (build-tools + platforms) and a JDK
```

Manual pipeline: javac --release 8 → d8 → aapt2 → zipalign → apksigner; output in `dist/`.

### Disclaimer

For personal-device ad-blocking only. Not affiliated with the app vendor. No paid-content unlocking, no DRM circumvention. Host app updates may break anchors.

## License

[MIT](./LICENSE)
