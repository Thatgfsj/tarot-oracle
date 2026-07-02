# 塔罗镜 / Tarot Oracle (v0.1.0)

一款单屏的 Android 原生应用，从 78 张完整韦特塔罗牌（Rider-Waite deck，含 22 大阿卡那 + 56 小阿卡那）中随机抽取一张牌或三卡阵（过去/现在/未来），并可一键把卡牌信息分享到 DeepSeek AI 求解读。

| | |
|---|---|
| 包名 | `com.thatgfsj.tarot` |
| 当前版本 | 0.1.0 (versionCode 1) |
| 平台 | Android 8.0+ (API 26 ~ 34) |
| 技术栈 | Kotlin 2.0 + Jetpack Compose + Material 3 |
| 数据 | **完全离线**（78 张牌 baked 进 APK assets） |

## 主要功能

- **首页**：塔罗镜标题、点击抽取主按钮 + 三卡阵 过去/现在/未来 次按钮。整体垂直居中。
- **手动翻牌**：抽到牌之后牌面朝下，提示"点击翻牌"。点一下 0°→360° 翻过来（0.8s）。再点一下翻回去。
- **单卡**：160dp×230dp 大图 + 中文名 + 英文名 + 顺位/逆位标识。
- **三卡阵**：横排 3 张 80dp×115dp 小牌，每张上方有"过去/现在/未来"标签。每张独立点击翻牌。
- **问 AI**：底部按钮，文本格式：
  ```
  我抽取了塔罗牌，这是我抽到的卡牌：

  [牌内容]

  我要问的问题是：
  ```
  单卡格式：`<牌名>（<name_en>）<顺位/逆位>`
  三卡阵格式：每张一行，前缀"过去/"/"现在："/"未来："。
  通过 `Intent.ACTION_SEND` + `setPackage("com.deepseek.chat")` 跳到 DeepSeek，Android 11+ 通过 `<queries>` 声明包可见性。
- **返回首页**：底部按钮，清空状态回到首页。

## 完全离线

- 无网络请求
- 无后台 service
- 无 WorkManager / 启动器 receiver
- 78 张牌 baked 进 `res/drawable-nodpi/*.jpg`（1.1MB 来自神婆网 shenpowang.com）
- 78 张 manifest baked 进 `assets/cards.json`
- Android 系统可以随时杀进程，下次启动 < 1s（assets 已经在 APK 里）

## 视觉风格

跟 chief 写的 `workspace/tarot/index.html` 网站一致：
- 深紫黑底 (`#040214` + `#28105A` + `#3C126E` 径向渐变)
- 金色主色 (`#D4AF37`)、香槟金副色 (`#F5E7A0`)
- 米色文字 (`#F5E7C8`)
- Serif 字体 (Cormorant Garamond / Source Han Serif SC)
- 启动器图标：金色塔罗卡轮廓 + 5 角星 + 紫底

## 快速构建

需要 JDK 17+ 与 Android SDK（platform 34、build-tools 34）。

```bash
cd tarot-oracle
./gradlew :app:assembleDebug
# 产物：app/build/outputs/apk/debug/app-debug.apk
```

通过 ADB 安装到设备：

```bash
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

## 目录结构

```
tarot-oracle/
├── app/
│   ├── build.gradle.kts                       # AGP 配置，依赖通过 libs.versions.toml
│   ├── src/main/
│   │   ├── AndroidManifest.xml                # 含 <queries> 声明 DeepSeek 包名
│   │   ├── assets/cards.json                  # 78 张牌 (id, name_zh, name_en, image_res, arcana, suit)
│   │   ├── java/com/thatgfsj/tarot/
│   │   │   ├── MainActivity.kt                # Activity + Compose host
│   │   │   ├── TarotViewModel.kt             # State machine (Initial / Loaded)
│   │   │   ├── share/
│   │   │   │   └── DeepSeekShare.kt          # Intent.ACTION_SEND 到 DeepSeek
│   │   │   ├── tarot/
│   │   │   │   ├── TarotModels.kt            # @Serializable 数据模型
│   │   │   │   └── TarotRepository.kt        # assets/cards.json 加载
│   │   │   └── ui/
│   │   │       ├── oracle/TarotScreen.kt     # 整个 UI 屏幕
│   │   │       └── theme/Theme.kt            # 紫黑金主题
│   │   └── res/                                # 启动器图标 + 78 张牌图
│   │       ├── drawable/ic_launcher_foreground.xml
│   │       ├── drawable/chief_bg.xml          # 星空背景层
│   │       ├── mipmap-anydpi-v26/             # adaptive icon
│   │       └── drawable-nodpi/                # 78 张牌 (the_fool.jpg, ace_of_wands.jpg, ...)
│   └── src/test/...
├── gradle/                                    # wrapper
├── gradlew, gradlew.bat
├── local.properties                           # sdk.dir
├── build.gradle.kts
├── settings.gradle.kts
└── README.md
```

## 数据来源

- 78 张塔罗牌图：神婆网 (https://www.shenpowang.com/taluopai/jieshi/) — 78 张 Rider-Waite 卡图，1.1MB total
- 中文名 / 英文名：神婆网元数据
- 顺位 / 逆位：50/50 随机翻转 (跟 chief 网站一致)

## 边界

- 本仓库是**独立产品**（Thatgfsj/tarot-oracle），不是 Flwntier 的子项目。Flwntier 是 v0.2.0 时 chief 写塔罗站点（`workspace/tarot/index.html` 67KB）的源头，但塔罗镜这个 Android app 是独立发布的。
- 跟 IChingOracle（同 78 张卦 / 64 张卡 vs 78 张塔罗 / 不同领域）只是技术栈相同（Kotlin + Compose + Material3 + 墨纸色），不是同一产品。
- 未来 v0.2 计划：加 meaning 解读文本到 cards.json，DeepSeek 分享时附带解读而不是只发牌名。

## 状态 (2026-07-03)

- ✅ Gradle build：`./gradlew :app:assembleDebug` 53s 完成
- ✅ APK 14.7 MB,valid Android package
- ✅ 78 张图全 baked 进 APK
- ✅ APK 在 `C:/Users/thatg/Desktop/tarot-oracle-v0.1.0-debug.apk`
- ⏳ 等主席装真机后告诉我有什么要调
