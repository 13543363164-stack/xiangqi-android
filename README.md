# 中国象棋 H5 → Android APK（原生 WebView 离线套壳）

把单文件 H5 象棋游戏（`assets/index.html` + 4 个音效）套在原生 **WebView** 里，零第三方框架、可完全离线运行。  
工程放在**不含中文的路径**（`D:/xiangqi-android`），避免 Gradle/Java 对中文路径的编码问题。

---

## 〇、先读这句（重要）

- 本工程为**纯 WebView 套壳**，零框架、可离线；游戏所有逻辑都在 `assets/index.html` 内。
- 本机（构建环境）没有 JDK / Android SDK，这里只交付**源码工程**，不编译任何 APK 二进制。
- 你已确定走 **GitHub Actions 云端打包**：全程**纯网页鼠标操作**，不需要本机装 Android Studio / Git / JDK。
- **我不会登录、也不会替你操作 GitHub 账号**。下面所有“点击/上传/触发”都由你本人在 GitHub 网页上手动完成；我仅提供文本与代码。
- ⚠️ **默认（未配置签名 Secrets）Actions 产出的是 `debug 测试包`**，用 Android 自动 debug 签名，**仅供你自己手机本地安装体验/测试，不能提交 TapTap 等商店**。要上架必须配置签名 Secrets（见第四节）。

---

## 一、目录结构与文件清单（代码 → 路径对照）

| GitHub 仓库内的路径                                            | 作用                    | 代码见       |
| -------------------------------------------------------- | --------------------- | --------- |
| `.github/workflows/build-apk.yml`                        | CI 云端打包脚本             | 代码③       |
| `settings.gradle`                                        | 工程/插件仓库设置             | 代码①       |
| `build.gradle`                                           | 顶层构建（AGP 8.2.2）       | 代码②       |
| `gradle.properties`                                      | Gradle JVM 配置         | 代码④       |
| `app/build.gradle`                                       | 模块构建 + 签名回退逻辑         | 代码⑤       |
| `app/proguard-rules.pro`                                 | 混淆规则（留空）              | 代码⑥       |
| `app/src/main/AndroidManifest.xml`                       | 清单/权限/入口 Activity     | 代码⑦       |
| `app/src/main/java/com/xiangqi/h5game/MainActivity.java` | WebView 主活动           | 代码⑧       |
| `app/src/main/res/values/strings.xml`                    | 应用名“中国象棋”             | 代码⑨       |
| `app/src/main/res/values/styles.xml`                     | 无标题栏沉浸式主题             | 代码⑩       |
| `app/src/main/res/values/colors.xml`                     | 启动图标背景色               | 代码⑪       |
| `app/src/main/res/mipmap-anydpi-v26/ic_launcher.xml`     | 自适应启动图标               | 代码⑫       |
| `app/src/main/res/drawable/ic_launcher_foreground.xml`   | 图标前景矢量图               | 代码⑬       |
| `app/src/main/assets/index.html`                         | **游戏本体（约 127KB，单文件）** | 见“如何放入”说明 |
| `app/src/main/assets/privacy.html`                       | 隐私政策网页（极小）            | 代码⑭       |
| `app/src/main/assets/move.wav` 等 4 个                     | 音效（纯英文命名）             | 见“如何放入”说明 |

> `index.html`（127KB）与 4 个 `.wav`（二进制）**不适合在网页文本框里粘贴**。请按第二节“上传大文件”的方式，用 GitHub 网页的 **Add file → Upload files** 直接拖进去。其余文本小文件用“Create new file”粘贴下方代码即可。

---

## 二、懒人网页上传 GitHub 完整步骤（纯鼠标，无 git 命令行）

> 全程在 github.com 网页操作。下面每一步都是你在网页上点/填，我不代操作。

### 第 1 步：网页新建仓库

1. 登录 github.com → 右上角 **“+” → New repository**。
2. **Repository name**：填 `xiangqi-android`（或你喜欢的英文名，只能用字母/数字/中划线）。
3. **Description（可选）**：填 `中国象棋 H5 安卓套壳`。
4. 可见性：**Public**（Actions 免费额度更宽；Private 也行但要先绑定支付方式才能用 Actions 分钟数）。
5. **不要**勾选 “Add a README file” / “Add .gitignore” / “Choose a license”——本工程自带这些，勾了反而冲突。
6. 点 **Create repository**。

### 第 2 步：建目录 + 放入文件

GitHub 网页**不能直接“新建文件夹”**，但你在“Create new file”的**文件名里带斜杠**就会自动建目录。两种方式任选：

**方式 A（小文本文件，逐个粘贴）**：点 **Add file → Create new file**，在 `Name your file…` 框里输入**完整带路径的文件名**（例如 `.github/workflows/build-apk.yml`），然后把下方“第三节”对应代码整段粘进编辑框，最后点 **Commit changes**。依次建好：代码①~⑬、⑭（privacy.html 也可粘贴，很小）。

**方式 B（大文件，直接拖）**：点 **Add file → Upload files**，把本工程文件夹里这些内容**拖进上传区**：

- `app/src/main/assets/index.html`
- `app/src/main/assets/privacy.html`
- `app/src/main/assets/move.wav`、`capture.wav`、`check.wav`、`mate.wav`

> 提示：也可以把整个 `xiangqi-android` 文件夹直接拖进 Upload files，GitHub 会保留子目录结构，一次搞定所有文件（最省事）。上传后确认仓库里目录结构与“第一节表格”一致即可。

### 第 3 步：触发构建

1. 进入你的仓库首页，点上方 **Actions** 标签。
2. 左侧若提示“workflows”未启用，按页面提示点 **I understand my workflows, go ahead and enable them**。
3. 左侧列表点 **Build Android APK** → 右侧 **Run workflow** → 分支选 `main`（或 `master`，看你仓库默认分支名）→ 点 **Run workflow**。
4. 页面会出现一条运行记录，状态从 `queued → in progress`，通常 3–8 分钟。

### 第 4 步：下载 APK 产物

1. 等运行记录变成绿色 ✓（或黄色，但已结束）。点进这条记录。
2. 页面下方 **Artifacts** 区会出现 `xiangqi-release-apk`，点它下载 `.zip`。
3. 解压得到 `app-release.apk`。

### 第 5 步：重要声明（务必看）

- **这次下载的 `app-release.apk` 是 debug 测试包**（因为没配置签名 Secrets，自动用 Android debug 签名）。
- 它**只能**装在你自己手机上玩/测，**不能**提交 TapTap、应用宝等任何商店——商店会拒绝 debug 签名。
- 要拿到**能上架的正式签名包**，按**第四节**配置签名 Secrets 后，重跑第 3 步即可。

---

## 三、各文件完整代码（供网页粘贴）

> 每块标题即它在 GitHub 里的完整路径。复制时连同行内的缩进一起复制。

### 代码① `settings.gradle`

```gradle
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}

rootProject.name = "XiangqiH5"
include ':app'
```

### 代码② `build.gradle`

```gradle
// Top-level build file. 子模块在 app/build.gradle 里声明插件。
plugins {
    id 'com.android.application' version '8.2.2' apply false
}
```

### 代码③ `.github/workflows/build-apk.yml`

```yaml
# 一键云端出包（纯网页 GitHub 操作即可，无需本机装 Android Studio / Git / JDK）。
# 用法：把本工程推/上传到 GitHub → Actions → 手动 Run workflow → 下载 Artifacts。
#
# ⚠️ 默认（未配置 Secrets）产出的是【debug 测试包】，用 Android 自动 debug 签名，
#    仅供你自己手机本地安装体验/测试，不能提交 TapTap 等商店。
#    要上架必须配置签名 Secrets（见 README「正式签名包（上架 TapTap 用）」一节）。
name: Build Android APK

on:
  workflow_dispatch:        # 允许在 Actions 页面手动触发
  push:
    branches: [main, master]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - name: Set up Android SDK
        uses: android-actions/setup-android@v3

      - name: Install SDK packages (platforms/build-tools)
        run: sdkmanager "platform-tools" "platforms;android-34" "build-tools;34.0.0"

      - name: Set up Gradle
        uses: gradle/actions/setup-gradle@v3
        with:
          gradle-version: '8.6'

      - name: Generate Gradle wrapper
        run: gradle wrapper

      - name: Decode keystore (可选，仅配置 Secrets 后才会执行)
        if: ${{ secrets.KEYSTORE_BASE64 != '' }}
        run: echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 -d > "$GITHUB_WORKSPACE/release.keystore"

      - name: Build Release APK
        run: ./gradlew assembleRelease
        env:
          KEYSTORE_FILE: ${{ github.workspace }}/release.keystore
          KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
          KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
          KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: xiangqi-release-apk
          path: app/build/outputs/apk/release/*.apk
```

### 代码④ `gradle.properties`

```properties
org.gradle.jvmargs=-Xmx2048m -Dfile.encoding=UTF-8
android.useAndroidX=true
```

### 代码⑤ `app/build.gradle`

```gradle
plugins {
    id 'com.android.application'
}

android {
    namespace 'com.xiangqi.h5game'
    compileSdk 34

    defaultConfig {
        applicationId "com.xiangqi.h5game"
        minSdk 26          // Android 8.0+，2026 年可忽略更低版本
        targetSdk 34
        versionCode 1
        versionName "1.0.0"
    }

    // 签名：通过环境变量注入，绝不写死到仓库里。
    // 未配置 keystore 密钥（没有 KEYSTORE_BASE64 等 Secrets）时，release 自动回退到 debug 签名，
    // 这种包仅供本地测试，不能上架商店；配置 Secrets 后得到正式签名包。
    signingConfigs {
        release {
            def ksFile = System.getenv("KEYSTORE_FILE")
            // 始终指向工作流注入的路径；文件是否存在决定最终用正式签名还是回退 debug
            storeFile file(ksFile != null ? ksFile : "release.keystore")
            storePassword System.getenv("KEYSTORE_PASSWORD")
            keyAlias System.getenv("KEY_ALIAS")
            keyPassword System.getenv("KEY_PASSWORD")
        }
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
            signingConfig (signingConfigs.release.storeFile != null && signingConfigs.release.storeFile.exists())
                ? signingConfigs.release : signingConfigs.debug
        }
        debug {
            // 默认使用 Android 自动生成的 debug 签名，可直接安装测试
        }
    }

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_17
        targetCompatibility JavaVersion.VERSION_17
    }
}
```

### 代码⑥ `app/proguard-rules.pro`

```
# 本游戏为纯 WebView 套壳，未开启混淆（minifyEnabled=false），此文件留空即可。
# 若日后开启混淆，可在此保留 WebView 接口与 JS 交互相关类。
```

### 代码⑦ `app/src/main/AndroidManifest.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:roundIcon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:hardwareAccelerated="true"
        android:theme="@style/Theme.Xiangqi">
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:screenOrientation="portrait"
            android:configChanges="orientation|screenSize|keyboardHidden"
            android:theme="@style/Theme.Xiangqi">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

### 代码⑧ `app/src/main/java/com/xiangqi/h5game/MainActivity.java`

```java
package com.xiangqi.h5game;

import android.app.Activity;
import android.os.Bundle;
import android.view.View;
import android.webkit.WebSettings;
import android.webkit.WebView;
import android.webkit.WebViewClient;

/**
 * 纯 WebView 套壳：把 assets/index.html 这个单文件 H5 象棋游戏跑在原生 WebView 里。
 * 零第三方框架、可完全离线，游戏所有逻辑都写在 index.html 内。
 */
public class MainActivity extends Activity {
    private WebView webView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        webView = new WebView(this);
        WebSettings ws = webView.getSettings();
        ws.setJavaScriptEnabled(true);                 // 游戏引擎依赖 JS
        ws.setDomStorageEnabled(true);                 // localStorage 存档/设置
        ws.setAllowFileAccess(true);
        ws.setAllowContentAccess(true);
        ws.setMediaPlaybackRequiresUserGesture(false); // 落子音效在点击手势内触发，放宽限制更稳
        ws.setUseWideViewPort(true);
        ws.setLoadWithOverviewMode(true);
        ws.setSupportZoom(false);
        ws.setBuiltInZoomControls(false);
        ws.setDisplayZoomControls(false);
        ws.setCacheMode(WebSettings.LOAD_DEFAULT);

        webView.setScrollBarStyle(View.SCROLLBARS_INSIDE_OVERLAY);
        webView.setWebViewClient(new WebViewClient());  // 所有跳转留在 WebView 内
        setContentView(webView);

        webView.loadUrl("file:///android_asset/index.html");
    }

    @Override
    public void onBackPressed() {
        if (webView != null && webView.canGoBack()) {
            webView.goBack();
        } else {
            super.onBackPressed();
        }
    }

    @Override
    protected void onPause() {
        super.onPause();
        if (webView != null) webView.onPause();
    }

    @Override
    protected void onResume() {
        super.onResume();
        if (webView != null) webView.onResume();
    }

    @Override
    protected void onDestroy() {
        if (webView != null) { webView.destroy(); webView = null; }
        super.onDestroy();
    }
}
```


### 代码⑨ `app/src/main/res/values/strings.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">中国象棋</string>
</resources>
```

### 代码⑩ `app/src/main/res/values/styles.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    
    <style name="Theme.Xiangqi" parent="android:Theme.Material.Light.NoActionBar">
        <item name="android:windowNoTitle">true</item>
    </style>
</resources>
```

### 代码⑪ `app/src/main/res/values/colors.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    
    <color name="ic_launcher_background">#C8A35B</color>
</resources>
```

### 代码⑫ `app/src/main/res/mipmap-anydpi-v26/ic_launcher.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
    <background android:drawable="@color/ic_launcher_background" />
    <foreground android:drawable="@drawable/ic_launcher_foreground" />
</adaptive-icon>
```

### 代码⑬ `app/src/main/res/drawable/ic_launcher_foreground.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="108dp"
    android:height="108dp"
    android:viewportWidth="108"
    android:viewportHeight="108">
    
    <path android:fillColor="#FFFFFF" android:pathData="M54,14a40,40 0 1,0 0.1,0z" />
    <path android:fillColor="#1F1F1F" android:pathData="M54,26a28,28 0 1,0 0.1,0z" />
    <path android:fillColor="#C0392B" android:pathData="M54,36a18,18 0 1,0 0.1,0z" />
</vector>
```

### 代码⑭ `app/src/main/assets/privacy.html`

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no" />
<title>隐私政策 · 中国象棋</title>
<style>
  * { box-sizing: border-box; }
  body { margin: 0; font-family: -apple-system, "PingFang SC", "Microsoft YaHei", sans-serif;
         background: #f3ecdc; color: #3a2f22; line-height: 1.7; padding: 22px 16px 40px; }
  .wrap { max-width: 720px; margin: 0 auto; background: #fffdf7; border-radius: 14px;
          padding: 22px 20px; box-shadow: 0 6px 24px rgba(0,0,0,.08); }
  h1 { font-size: 22px; text-align: center; color: #7a3a12; letter-spacing: 2px; margin: 4px 0 18px; }
  h2 { font-size: 16px; color: #8a4a1a; margin: 20px 0 6px; }
  p { margin: 8px 0; font-size: 14px; }
  .note { background: #fff4e0; border-left: 4px solid #d9a441; padding: 10px 12px; border-radius: 6px; font-size: 13px; }
  .back { display: block; width: 100%; margin-top: 26px; padding: 13px; border: none; border-radius: 10px;
          background: #8a4a1a; color: #fff; font-size: 16px; cursor: pointer; }
  .back:active { transform: scale(.98); }
  footer { text-align: center; font-size: 12px; color: #998; margin-top: 18px; }
</style>
</head>
<body>
  <div class="wrap">
    <h1>隐私政策</h1>
    <p>本《隐私政策》适用于「中国象棋」单机版应用/网页（以下简称“本游戏”）。本游戏为纯前端单机程序，我们高度重视你的隐私。</p>

    <h2>一、我们收集哪些信息</h2>
    <p><b>我们不收集任何个人信息。</b>本游戏不包含账号注册、登录、表单提交，也不包含任何付费、内购、广告或第三方统计/追踪 SDK。</p>

    <h2>二、数据的存储位置</h2>
    <p>本游戏运行所需的所有数据——包括你的设置项、对战战绩、棋谱记录、本地账户（昵称/ELO）等——<b>仅保存在你本人设备的浏览器/应用本地存储（localStorage）</b>中，<b>不会上传到任何服务器</b>，我们也无法访问。</p>
    <div class="note">联机对战功能仅在“同一台设备的两个窗口之间”通过浏览器本地通道（BroadcastChannel）即时同步棋步，过程不经过任何远程服务器，也不产生任何日志上传。</div>

    <h2>三、数据的保留与删除</h2>
    <p>上述本地数据会一直保留，直到你主动清除浏览器/应用数据，或卸载应用。清除后数据不可恢复，且不会影响本游戏的其他功能。</p>

    <h2>四、儿童与未成年人</h2>
    <p>本游戏不含任何面向未成年人的不当内容、付费或诱导。若你是未成年人，请在监护人指导下使用。</p>

    <h2>五、政策的变更</h2>
    <p>如本游戏未来接入联网服务、付费或第三方 SDK，我们将在本页面更新政策并重新征求你的同意。</p>

    <h2>六、联系我们</h2>
    <p>如你对本隐私政策有疑问，可通过应用内反馈渠道或开发者公布的方式与我们联系。</p>

    <p style="margin-top:18px;font-size:13px;color:#998;">本政策最后更新：2026-10-12。</p>
    <button class="back" type="button" onclick="location.href='index.html'">返回游戏</button>
    <footer>© 中国象棋 · 纯本地单机应用</footer>
  </div>
</body>
</html>
```

---

## 四、正式签名包（上架 TapTap 用）

debug 包不能上架，必须用自己的签名证书。步骤如下（**全在本机命令行/本地完成，与 GitHub 网页无关**）：

1. **生成证书**（keytool 随 JDK 提供，本机需装好 JDK 17）：
   ```bash
   keytool -genkeypair -v ^
     -keystore release.keystore ^
     -alias xiangqi ^
     -keyalg RSA -keysize 2048 -validity 10000
   ```
   按提示填密码与信息。
2. **备份证书**：`release.keystore` 一旦丢失，**无法更新已上架的应用**，请离线多处备份。
3. **转 base64**（放进 GitHub Secrets，仓库不存明文）：
   ```bash
   # Windows (PowerShell)
   [Convert]::ToBase64String([IO.File]::ReadAllBytes("release.keystore")) | Out-File -NoNewline keystore.b64
   # 或 macOS/Linux
   base64 -i release.keystore -o keystore.b64
   ```
   复制 `keystore.b64` 的整段文本。
4. **在 GitHub 配置 4 个 Secrets**（仓库 → Settings → Secrets and variables → Actions → New repository secret）：
   - `KEYSTORE_BASE64`：上面那段 base64 文本
   - `KEYSTORE_PASSWORD`：生成证书时设的密钥库密码
   - `KEY_ALIAS`：`xiangqi`（上面 -alias 的值）
   - `KEY_PASSWORD`：密钥密码（若与密钥库密码相同则填一样）
5. **重跑第三节的第 3 步**（Actions → Run workflow）。这次 `Decode keystore` 步骤会执行，产出**正式签名包**，下载后即可用于 TapTap 上架。

> 未配置 Secrets 时，构建会自动回退 debug 签名——所以永远先确认这 4 个 Secrets 已填，再跑上架包。

---

## 五、Android Studio 本地构建（备用，可选）

若你后来想在本机直接出包（非必须）：

1. 安装 Android Studio（自带 JDK 17 + SDK）。
2. `File → Open` 选工程目录 → 等待 Sync。
3. `Build → Build Bundle(s)/APK(s) → Build APK(s)` → 产物在 `app/build/outputs/apk/release/`。
4. 正式签名按第四节的 keystore + 配置后重新 Build。

---

## 六、TapTap 上架清单

TapTap 以 **APK** 形式上架，传第四节的正式签名包即可。

- **开发者实名**：TapTap 开发者中心注册企业/个人主体，完成实名。
- **素材**：512×512 图标（用 `res` 里的矢量图标导出）、横竖截图各 ≥3 张、简介、详情页图文。
- **隐私政策**：必须有一份独立《隐私政策》网页（本项目已附 `privacy.html`，说明“仅本地存储、不收集任何个人信息”），在 TapTap 后台与游戏内“设置 → 隐私政策”里都要能打开。
- **不计费抢先体验（重点）**：本游戏当前**无任何变现、纯免费**；TapTap 通常允许以「抢先体验 / 不计费测试」形态上架，**无需版号**。最终是否可行以 TapTap 开发者后台**最新政策为准**，提交前请在后台确认。
- **商业化需版号**：若未来接入任何内购/广告，按法规需《网络游戏出版审批》（版号）。
- **年龄分级**：在后台如实填写（棋牌一般 12+/全年龄，按实际选）。
- 提交审核 → 通过后以免费/测试形态上架。

---

## 七、微信方案（二选一，本次暂缓）

> 现有 `index.html` 是 **SVG + DOM** 渲染，**无法直接跑在微信小游戏**里——小游戏是 Canvas-only 运行时，没有 DOM/SVG/`<audio>`/`localStorage`/`BroadcastChannel`，这些都要换成 `wx.*` API。

- **方案 A：微信小程序 + `<web-view>` 嵌入（最快）**
  - 把 `index.html` 整套部署到 **HTTPS** 域名（自有服务器 / 对象存储 + CDN）。
  - 小程序里用 `<web-view src="https://你的域名/index.html">` 加载。
  - 限制：需在微信公众平台配置**业务域名白名单**（HTTPS）；web-view 内无法调用部分微信原生能力；本质是网页套壳。
- **方案 B：微信小游戏 Canvas 原生重构（工作量大）**
  - 渲染全改 Canvas（手画棋盘/棋子/动画），所有浏览器 API 替换为 `wx.*`：音效 `wx.createInnerAudioContext()`、存储 `wx.setStorageSync/getStorageSync`、联机/排行榜 `wx.getOpenDataContext()`、入口 `game.js + game.json`。
  - 等于重写一个原生小游戏，体验最贴合微信，但工作量很大。
  - **棋牌类目限制**：微信小游戏**个人主体基本无法发布棋牌类**，需**企业/组织**主体；棋牌属特殊类目，通常要求**版号**；未接入虚拟支付、纯休闲也要走类目资质审核。

---

## 八、合规说明

- **设置面板已预留「隐私政策」按钮**：游戏“更多 → 设置”面板底部新增「隐私政策」按钮，点击打开 `privacy.html`（离线本地页面，可返回）。
- **国内棋牌渠道版号风险**：中国象棋属棋牌类，是受监管类目。未接入付费、纯休闲时，TapTap「不计费抢先体验」通常不要求版号，但仍受平台类目与内容审核约束；微信小游戏对个人主体与棋牌类限制更严。请在上架前以各平台**最新政策**为准，必要时办理版号。

---

## 九、后续扩展项（简单记录）

- **真·联机对战**：当前仅同设备双窗口（BroadcastChannel）。跨设备需 **WebSocket 后端服务 + 数据库**，可接云开发（Supabase/Firebase 类）。
- **平台 SDK 接入**：TapTap / 微信 的登录、成就、排行榜、支付 SDK（需各自 AppId）。
- **多语言**：游戏内已支持简/繁/英切换，商店上架材料按需补充。

---

## 十、权责声明

- 本工程为**模板/脚手架**，账号（GitHub / TapTap / 微信）、签名证书（keystore）、资质（实名/企业主体）、版号等**全部由开发者自行准备**。
- **GitHub Actions 脚本仅提供代码**；我不会登录或代你操作 GitHub 账号，上传、触发工作流、下载 APK 等所有网页操作由你本人手动完成。
- 构建、签名、上架涉及的合规责任由开发者承担。
