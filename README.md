# 锡院通 · 应用内更新源

这个目录就是 App 的「更新源」。把它整目录推到 GitHub / Gitee，然后在
**App → 我的 → 应用更新 → 更新源设置** 里填一次地址，之后每次启动会自动自检。

## 目录内容

| 文件 | 作用 |
| --- | --- |
| `update.json` | 版本清单（版本号、SHA-256、文件大小、更新日志） |
| `update.json.sig` | 清单的 RSA-SHA256 签名，App 内置公钥验签后才信任 |
| `xiyuantong-v2.10.apk` | 整包 APK，只在原生层 / 权限变更时需要用户确认安装 |
| `www-r21.dat` | 前端热更资源包，下载校验后免安装生效 |
| `README.md` | 本说明 |

清单里下载地址写的是**相对文件名**，所以这个目录换个仓库 / 分支 / 域名都不用重新生成。

## 推到 GitHub（推荐：直接走 raw，不用开 Pages）

```bash
cd D:\AI工作间\学校实用应用开发\android-app\release
git init
git add .
git commit -m "release v2.10 r21"
git branch -M main
git remote add origin https://github.com/<你的用户名>/<仓库名>.git
git push -u origin main
```

推完后在 App 里填下面任意一种地址（App 会自动补 `update.json`）：

```
https://raw.githubusercontent.com/<用户名>/<仓库名>/main/release
https://<用户名>.github.io/<仓库名>/release          # 若开了 GitHub Pages
```

> GitHub raw 有 CDN 缓存，刚 push 后可能要等几分钟才看到新版本。
> App 每次请求都带 `no-cache` 和时间戳参数，通常不用手动清缓存。

## 推到 Gitee

```bash
git remote add origin https://gitee.com/<用户名>/<仓库名>.git
git push -u origin master
```

地址填 `https://gitee.com/<用户名>/<仓库名>/raw/branch/master/release` 或
开了 Gitee Pages 后填 `https://<用户名>.gitee.io/<仓库名>/release`。
**注意**：Gitee 对 APK / zip 直链偶有登录或防盗链限制，第一次请在 App 里点
「检查更新」看是否报 HTTP 错误；若被拦，改用 GitHub 或自己的静态托管。

## 本机局域网测试（不上传公网）

```bash
cd D:\AI工作间\学校实用应用开发\android-app\release
python -m http.server 8765
```

手机连同一 WiFi，在 App 里填 `http://<电脑局域网IP>:8765`。
清单默认要求验签（局域网源同样有签名），所以这条链路和公网完全一致。

## 我以后怎么发新版本

**只改前端页面 / JS / CSS**（最常见，你不用装 APK）：
```bash
python release.py --bump --changelog "修了课表显示"
git -C release add . && git -C release commit -m "web r<N>" && git -C release push
```
App 启动自检或手动点「检查更新」→ 点「立即热更」→ 自动重载，数据不丢。

**改了原生 Java / Manifest / 权限**（要装新 APK）：
1. 把 `AndroidManifest.xml` 的 `versionCode` 加 1、`versionName` 改掉；
2. `bash build.sh` 重新构建签名 APK；
3. `python release.py --bump --apk-changelog "改了什么"`；
4. push `release/`；App 会提示「下载并安装 APK」，用户确认一次即可。

## 安全约定

- `release-keys/private.pem` 是签名私钥，**绝不进这个目录、绝不上传**。请另存备份；
  私钥丢失后，已安装 APK 的内置公钥无法验证新清单，只能手动重装一次 APK。
- App 内置公钥在 `assets/update.pub`。换密钥 = 换信任根，必须重新分发 APK。
- 严格验签默认开启：清单没有 `update.json.sig` 或验签失败，直接拒绝更新。
- 每个包都额外校验 SHA-256 与文件大小；解压时拒绝 `..` 路径穿越并限制总大小。
- 热更包是会在 WebView 里执行的 JS，所以验签不能关。要临时关闭验签，
  在 App「更新源设置」里把「清单验签」改成关闭（仅校验包体哈希），不建议长期用。

当前：APK v2.10 (versionCode 12) · 前端 r21 · 生成于 2026-11-14 18:46
