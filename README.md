# 2048 · JavaFX

> 经典 2048，但棋盘不再是 4×4 —— **3×3 到 8×8 可变**，胜利目标恒定为 2048。
>
> 纯逻辑引擎与渲染层彻底分离：核心规则不依赖 JavaFX，可以脱离图形界面跑单元测试。

<p>
  <img alt="Java" src="https://img.shields.io/badge/Java-17-ED8B00?style=flat-square&logo=openjdk&logoColor=white">
  <img alt="JavaFX" src="https://img.shields.io/badge/JavaFX-17.0.14-FF6B35?style=flat-square">
  <img alt="Maven" src="https://img.shields.io/badge/Maven-3.9-C71A36?style=flat-square&logo=apachemaven&logoColor=white">
  <img alt="Tests" src="https://img.shields.io/badge/测试-3%20个测试类-52C41A?style=flat-square">
</p>

---

## 玩法

方向键 / WASD 移动，相同的数字相撞合并为二者之和，每次有效移动后随机生成一个新方块（**90% 是 2，10% 是 4**）。

- 棋盘无法再移动（无空位且相邻无相等）时结束
- 合成出 **2048** 获胜，可继续玩
- **`Z` 键撤销**，最多回退 20 步

> 棋盘变大后难度会明显下降，所以「胜利目标固定 2048」是这个版本唯一的平衡手段——8×8 下想输都难，但想要 2048 仍然需要规划。

---

## 功能

| | 功能 | 说明 |
| --- | --- | --- |
| 🎛️ | **可变棋盘** | 3×3 ~ 8×8，开局选择 |
| ↩️ | **撤销** | 引擎快照栈，深度 20，`Z` 键触发 |
| 🌗 | **双主题** | 浅色（经典）/ 暗黑，一键切换即时生效 |
| 🌏 | **中英双语** | `ResourceBundle` 拆资源，切换即时生效 |
| ✨ | **移动动画** | 方块滑动 / 合并 / 生成三段动画，动画期间屏蔽输入 |
| 📊 | **统计面板** | 步数、用时、历史最佳榜 |
| 🔊 | **音效** | 合并 / 胜利 / 结束 / 点击四类，可关闭（加载失败静默降级） |
| ⌨️ | **双操作** | 键盘 + 屏幕方向按钮（触屏/鼠标可用） |

---

## 架构

**引擎层完全不知道 JavaFX 的存在。** 这条边界是这个项目最重要的设计：

```
org.example
├── App.java                     启动入口
├── game/                        ← 纯逻辑，零 JavaFX 依赖
│   ├── GameEngine.java          核心规则：移动、合并、生成、胜负判定
│   ├── Tile.java                方块
│   ├── Direction.java           方向
│   ├── MoveResult.java          一次移动的变化记录（供动画消费）
│   ├── TileMove.java / TileSpawn.java   动画所需的位置变化
│   ├── ScoreStore.java          最高分持久化
│   ├── GameStats.java           统计
│   ├── HistoryEntry.java        历史记录
│   ├── UndoManager.java         撤销栈（棋盘 + 分数快照，深度 20）
│   ├── UndoEntry.java / UndoResult.java
└── ui/                          ← 表现层
    ├── GameController.java      输入、动画、主题/语言/音效/统计的总控
    ├── BoardLayout.java         棋盘布局计算
    ├── TileViewFactory.java     方块视图工厂
    ├── ThemeManager.java        两套 CSS 互切
    ├── FontKit.java / Icons.java
    ├── GlowBackground.java      背景光晕
    ├── SoundPlayer.java         AudioClip 播放
    ├── I18n.java                国际化
    ├── effect/EffectManager.java 特效
    └── resources/css/           浅色 / 暗黑两套主题
```

**为什么值得这样拆**：

- `GameEngineTest` 可以在**没有图形界面**的环境里跑（CI、纯命令行），测的是合并规则、边界、胜负判定这些真正容易错的地方；
- `BoardLayoutTest` 单独测布局数学，不用起窗口；
- 想移植到 Swing / Compose / 网页，只需重写 `ui/`，`game/` 一行不动。

---

## 运行

环境：JDK 17+、Maven 3.9+

```bash
git clone https://github.com/sqyCoder/game-2048.git
cd game-2048

mvn javafx:run     # 运行
mvn test           # 跑测试
```

---

## 测试

| 测试类 | 覆盖 |
| --- | --- |
| `GameEngineTest` | 移动方向、合并规则（一次移动中同一个方块不能合并两次）、生成、胜负判定、撤销快照 |
| `ScoreStoreTest` | 最高分读写、损坏数据的容错 |
| `BoardLayoutTest` | 不同棋盘尺寸下的布局计算 |

```bash
mvn test
```

> `GameEngineTest` 里最容易出错的用例是**「[2,2,2,2] 向左应该变成 [4,4] 而不是 [8]」**——一次移动中每个方块只能参与一次合并。这条规则在几乎所有手写 2048 里都写错过。

---

## 开发文档

这个项目是**先写 spec 再写代码**的，四份文档留在仓库里：

| 文件 | 内容 |
| --- | --- |
| [spec.md](spec.md) | 需求分析与定制决策汇总（棋盘、主题、语言、撤销、音效的取舍理由）|
| [spec2.md](spec2.md) | 架构与模块划分 |
| [spec3.md](spec3.md) | 动画与交互细节 |
| [spec4.md](spec4.md) | 收尾与验收项 |

> 留着它们比留着代码有意思——能看到**当时是怎么想的**，以及后来哪些决策被推翻了。

---

## 已知限制

- **无联网排行榜**，最高分只存本机。
- **动画期间屏蔽输入**：这是刻意的（防止快速连按导致逻辑与画面错位），代价是手感上有一点点延迟。
- **只支持键盘和屏幕按钮**，没做手势滑动。
- 棋盘上限 8×8，再大会让方块小到看不清。

---

## License

个人项目，代码可自由参考。
