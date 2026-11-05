# Drama SDK 二进制发布仓库

此工程通过 `maven-publish` 发布预构建的 `drama-sdk.aar`，并从 `drama-sdk.pom` 复制公共第三方依赖。只需要 JDK 17 和 Gradle Wrapper，不需要 Android SDK 或业务源码。

## 发布位置

- GitHub：[aboutwin/DZDramaSDK_Android](https://github.com/aboutwin/DZDramaSDK_Android)
- JitPack：[aboutwin / DZDramaSDK_Android](https://jitpack.io/#aboutwin/DZDramaSDK_Android)
- 首次发布目标：Tag `1.0.0`，坐标 `com.github.aboutwin:DZDramaSDK_Android:1.0.0`。

发布状态以 JitPack 构建日志和远端产物为准。

## 更新二进制

在内部源码工程修改 `drama-sdk/build.gradle` 的 `android.defaultConfig.versionName`，执行：

```powershell
.\gradlew.bat :drama-sdk:assembleRelease :drama-sdk:publishToMavenLocal
```

成功后，将同次构建的 `drama-sdk/build/outputs/aar/drama-sdk-v<版本>-release.aar` 和 `drama-sdk/build/publications/release/pom-default.xml` 分别复制为本目录的 `drama-sdk.aar`、`drama-sdk.pom`。每次源码更新均需重新生成二进制。

不上传业务源码、独立 XML 资源、签名文件、密码、mapping、`local.properties` 或构建缓存。

## 本地发布验证

在本目录执行：

```powershell
.\gradlew.bat publishDramaSdkPublicationToVerificationRepository '-Pgroup=com.github.aboutwin' '-Partifact=DZDramaSDK_Android' '-Pversion=1.0.0'
```

检查 `build/repo/com/github/aboutwin/DZDramaSDK_Android/1.0.0/` 下生成的 POM 坐标、依赖和 AAR SHA-256。默认本地坐标为 `com.dz.sdk:drama-sdk:1.0.0`；环境变量 `GROUP`、`ARTIFACT`、`VERSION` 优先于 `-Pgroup`、`-Partifact`、`-Pversion`。

## JitPack 发布

1. 仅在独立二进制仓库中提交明确列出的发布文件，确认远端及本次版本。
2. 取得推送授权后，推送指定分支和新 Tag；不推送内部源码仓库，不使用 `git add .` 或 `git push --tags`。
3. 在 JitPack 页面选择该 Tag，点击 **Get it**。`jitpack.yml` 使用 JDK 17 执行 `bash gradlew publishToMavenLocal`，发布脚本使用 JitPack 提供的 `GROUP`、`ARTIFACT`、`VERSION`。
4. 构建成功后下载远端 POM/AAR，核对依赖与本次输入 AAR 的 SHA-256；使用独立宿主解析依赖并构建、运行。

升级必须使用新版本和新 Tag，不覆盖已有版本。

## 宿主接入

```groovy
// settings.gradle
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven { url 'https://maven.aliyun.com/repository/releases/' }
        maven { url 'https://jitpack.io' }
    }
}

// app/build.gradle
dependencies {
    implementation 'com.github.aboutwin:DZDramaSDK_Android:1.0.0'
}
```

删除旧本地 AAR 依赖，避免重复打包；SDK 的公共传递依赖由 POM 提供。当前初始化入口为 `DramaSDK.init(application, appKey, oaid)`，使用平台分配的有效 appKey。

SDK 自有资源使用 `dzdr_` 前缀，内部资源不作为宿主定制接口。宿主自行定义颜色、尺寸和布局，文字使用 dp，不依赖 SDK 内部色板或尺寸字典。

验证应包含指定版本依赖解析、APK 编译、真机启动、推荐/剧场/收藏/搜索及实际播放。包名、签名证书与播放器许可证应匹配；不要公开签名配置或密钥。
