# 塔罗镜 / Tarot Oracle — 开发文档

## 架构

```
┌─────────────────────────────────────────────┐
│  Android chief app (tarot-oracle)            │
│  离线 baked assets + Compose UI              │
│                                              │
│  78 张 Rider-Waite 牌图 + Chinese 名字      │
│  baked into APK at build time                │
│                                              │
│  启动 → 读 cards.json + drawable-nodpi/     │
│       → TarotRepository.getInstance()        │
│       → UI on tap → Intent.ACTION_SEND       │
│         (com.deepseek.chat)                  │
└─────────────────────────────────────────────┘
```

## 数据流

```
User taps 抽取
  ↓
MainActivity → setContent { TarotScreen(viewModel) }
  ↓
ViewModel.drawOne() / drawThree()
  ├─ repo.drawOne() / repo.drawThree()     [TarotRepository, in-memory]
  ├─ repo.isReversed(card)                  [50/50 random]
  └─ _state.value = TarotUiState.Loaded(drawn)
  ↓
TarotScreen recomposes
  ├─ if drawn.size == 1: SingleCardView   (160×230 dp)
  │   tap card → 0°→360° Y-rotation 0.8s
  └─ else: SpreadView                       (3× 80×115 dp)
      tap each card → independent flip
  ↓
Chairman taps 问 AI
  ↓
DeepSeekShare.shareToDeepSeek(context, drawn)
  ├─ Format text (single card or 3-card spread)
  ├─ Intent.ACTION_SEND with setPackage("com.deepseek.chat")
  └─ Fallback to generic chooser if DeepSeek not installed
```

## 文件结构

```
tarot-oracle/
├── app/
│   ├── build.gradle.kts                          # AGP 配置
│   ├── proguard-rules.pro
│   └── src/main/
│       ├── AndroidManifest.xml                  # <queries> DeepSeek
│       ├── assets/cards.json                     # 78 张牌 manifest
│       ├── java/com/thatgfsj/tarot/
│       │   ├── MainActivity.kt                   # Activity + Compose
│       │   ├── TarotViewModel.kt                # State machine
│       │   ├── share/
│       │   │   └── DeepSeekShare.kt             # Share intent
│       │   ├── tarot/
│       │   │   ├── TarotModels.kt               # @Serializable model
│       │   │   └── TarotRepository.kt           # assets loader
│       │   └── ui/
│       │       ├── oracle/TarotScreen.kt        # UI screen
│       │       └── theme/Theme.kt               # 紫黑金
│       └── res/
│           ├── drawable/ic_launcher_foreground.xml
│           ├── drawable/chief_bg.xml             # 星空背景
│           ├── mipmap-anydpi-v26/               # adaptive icon
│           └── drawable-nodpi/                  # 78 张牌图
├── gradle/                                       # wrapper
├── gradlew, gradlew.bat
├── local.properties
├── build.gradle.kts
├── settings.gradle.kts
└── README.md
```

## 关键决策

- **离线 baked 78 张牌**：直接放 `res/drawable-nodpi/`，不下载，不调任何 API
- **手动翻牌**（不是自动）：每张卡在 LoadedView 进场时是 rotation 0°（牌面朝下），点一下翻过来
- **三卡阵 stagger 由用户决定**：不自动顺序翻，每张独立 click target
- **问 AI 用 Intent.ACTION_SEND**：跟 iching-oracle 同款，包可见性靠 `<queries>` 声明

## 版本历史

- **0.1.0** (2026-07-03): 初版
  - 78 张 baked
  - 单卡 + 三卡阵
  - 手动翻牌
  - DeepSeek share
  - 完全离线 (shotgun mode, no background service)
