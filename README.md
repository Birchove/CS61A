# CS61A 课程项目仓库

本仓库用于归档加州大学伯克利分校 **CS 61A**（程序设计与抽象）相关的**课程项目**与个人笔记，目前收录 **Ants**、**Hog**、**Cats** 三个项目（**不包含** lab、homework、discussion 等其余课程文件）。

**知乎专栏（总入口）**：[CS 61A 相关专栏](https://www.zhihu.com/column/c_1981023312877483099)

---

## 关于 CS 61A

**CS 61A** 是伯克利面向初学者的程序设计课程，核心目标包括：

- 用 **Python** 与 **Scheme** 练习程序设计，强调**抽象**、**数据抽象**与**元循环求值**等思想；
- 教材与讲义体系多基于 *Composing Programs*（与经典 SICP 一脉相承）；
- 作业与项目体量较大，注重**阅读大段他人代码**、**按规格扩展**与**用自动评测（如 `ok`）回归测试**。

课程在北美高校本科路线中知名度较高，常被自学与转专业同学用作综合训练。（具体学期政策以伯克利官方为准。）

---

## 仓库结构

```text
CS61A/                          # 仓库根目录
├── README.md
├── .gitignore
├── Ants/
│   ├── ants_diagram.pdf        # 课程设计流程图
│   └── ants/                   # Ants 工程
│       ├── ants.py
│       ├── gui.py
│       ├── ok
│       └── ...
├── Hogs/
│   └── hog/                    # Hog 骰子游戏项目
│       ├── hog.py
│       ├── hog_ui.py
│       ├── ok
│       └── ...
└── Cats/
    └── cats/                   # Cats 打字游戏项目
        ├── cats.py
        ├── cats_gui.py
        ├── ok
        └── ...
```

---

## Ants vs. SomeBees

### 学习笔记（专栏文章）

Ants 项目的逐题记录与心得见专栏单篇：[CS 61A Fall 2025 自学笔记（18）— Ants 项目](https://zhuanlan.zhihu.com/p/2001753505514414611)。

### 项目简介

**Ants** 是 CS 61A 中的经典**面向对象**大作业：在课程方提供的模拟器与 GUI 上，主要修改 **`ants.py`**，实现多种蚂蚁与地图机制，完成类似塔防的「蚂蚁防线 vs. 蜜蜂入侵」游戏。项目强调：

- **`GameState`、`Place`、`Hive`、`Insect`、`Ant`、`Bee`** 等类的协作；
- **继承与多态**：不同蚂蚁通过覆写 `action`、`reduce_health` 等表现不同行为；
- **阅读** `gui.py`、`ants_plans.py`、`tests/` 等**不提交**文件，理解整体架构。

官方项目说明（第三方存档）：[Ants Vs. SomeBees | CS 61A](https://insideempire.github.io/CS61A-Website-Archive/proj/ants/index.html)

### 本人的工作

以下内容与专栏文章一致，概括本人在 **`ants.py`** 中的实现与学习重点（**不含可照抄的完整题解代码**；细节见上文单篇链接）：

- **Phase 1**：为 `HarvesterAnt` / `ThrowerAnt` 配置食物成本并实现收割；补全 `Place.__init__` 的 **entrance** 链；重写 `ThrowerAnt.nearest_bee`，沿 **entrance** 向前搜索、跳过仍位于 **Hive** 的蜜蜂。调试中区分 **`None` 与 `False`** 等判断陷阱；理清 **部署（deploy）时才扣食物**、与仅实例化蚂蚁的区别。


- **Phase 2**：在 `ThrowerAnt` 上引入 **`lower_bound` / `upper_bound`**，实现 `ShortThrower` / `LongThrower`；修正循环条件，避免「距离未进入有效区间就永远不递增」的逻辑漏洞。实现 **`FireAnt.reduce_health`**：在必须用 **`super()`** 满足评测的前提下，处理**反弹伤害**与**阵亡额外伤害**，并避免对同一蜜蜂**重复调用** `reduce_health` 导致副作用重复。从零实现 **`WallAnt`**、**`HungryAnt`**（含咀嚼 **`cooldown`** 与 **`chew_cooldown`** 类属性）。完成 **容器蚁** 三阶段：**`ContainerAnt`**、**`Ant.add_to`**、**`BodyguardAnt`** 与 **`TankAnt`**。总结抽象父类 → 调整基类行为 → 写具体子类的套路。


- **Phase 3**：**`Water.add_insect`** 与 **`is_waterproof`**；**`ScubaThrower`**；**`QueenAnt`**（身后隧道伤害加倍、`double` 与「只加倍一次」、阵亡 **`ants_lose()`** 等）。可选 **EC** 在笔记中有展开。

### 游戏规则与玩法（简要）

- **目标**：消耗食物在隧道各格放置蚂蚁，阻止蜜蜂到达蚁巢一端；消灭全部蜜蜂则胜利；若有蜜蜂抵达巢穴、或（存在时）**女王蚁**阵亡，则失败。
- **每回合大致顺序**：新蜜蜂可能从蜂巢进入 → 玩家放置蚂蚁 → **先蚂蚁后蜜蜂**依次执行各自 `action`。
- **蜜蜂**：若当前格未被蚂蚁**阻挡**，则向隧道**出口方向**移动；否则攻击该格蚂蚁。
- **蚂蚁**：不同类型收集食物、远程投掷、反伤、护佑队友等，由子类实现。

在 `Ants/ants/` 目录内：

```bash
python3 gui.py           # 可视化运行
python3 gui.py -d hard
python3 gui.py --water   # 增加初始化状态
python3 gui.py --food 10
python3 ok -q 01         # 对每个子问题的测试
python3 ok --score       # ok计算得分
```

### Ants 相关链接

| 说明 | 链接 |
|------|------|
| 学习笔记（单篇） | [知乎](https://zhuanlan.zhihu.com/p/2001753505514414611) |
| 官方项目页（存档） | [insideempire…/proj/ants](https://insideempire.github.io/CS61A-Website-Archive/proj/ants/index.html) |
| 教材 *Composing Programs* | [composingprograms.com](https://www.composingprograms.com/) |

---

## Hog（骰子游戏）

### 学习笔记（专栏文章）

[Hog 项目自学笔记 — 知乎](https://zhuanlan.zhihu.com/p/1969801076610925300)

### 项目简介

**Hog** 是 CS 61A 中结合**高阶函数**、**闭包**与**控制构造**的骰子对战项目：在 **`hog.py`** 中实现掷骰、规则更新（如 Boar Brawl、Sus Fuss）、整局模拟 `play`、策略与实验等；图形/终端界面由课程提供的 `hog_ui.py` 等文件驱动。

官方项目说明（存档）：[Hog | CS 61A](https://insideempire.github.io/CS61A-Website-Archive/proj/hog/index.html)

### 本人的工作

与专栏笔记一致的学习与实现要点（**无完整题解**）：

- **Phase 1（规则与模拟）**：`roll_dice` 与测试骰子 **`make_test_dice`**（闭包、**`nonlocal`** 与循环序列）；**Boar Brawl** 得分增量；**`take_turn`** 组合掷骰与规则；**Sus Fuss** 相关（如 **`num_factors`**、**`sus_update`**）；**`play`** 中按回合交替调用双方 **strategy**、且每回合只调用一次当前玩家策略。


- **Interlude / Phase 2**：用高阶函数做 **Printing**、**`interactive_strategy`** 等；**`always_roll`**；**`is_always_roll`**（注意循环变量范围与 **goal**）；**`make_averaged`**（**`*args`** 转发）；**`max_scoring_num_rolls`**（避免对同一测试骰子**连续调用**导致状态错位）；**`boar_strategy` / `sus_strategy`**；可选 **final_strategy**。

### 运行方式

在 **`Hogs/hog/`** 目录下：

```bash
python3 hog_ui.py  # 最终的可视化运行
python3 ok -q 01   # 每个子问题的测试
```

---

## Cats（打字游戏）

### 学习笔记（专栏文章）

[Cats 项目自学笔记 — 知乎](https://zhuanlan.zhihu.com/p/1979833314337632284)

### 项目简介

**Cats** 项目围绕 **`cats.py`** 实现打字练习核心逻辑：**段落选择**、**准确率 / WPM**、**自动纠错**（**`autocorrect`**、编辑距离类函数）、以及多人模式相关数据结构等；Web 端由 **`cats_gui.py`** 与 `gui_files/` 等配合。

官方项目说明（存档）：[Cats | CS 61A](https://insideempire.github.io/CS61A-Website-Archive/proj/cats/index.html)

### 本人的工作

与专栏笔记一致的学习与实现要点（**无完整题解**）：

- **Phase 1**：**`pick`**；返回高阶函数的 **`about`**（大小写、标点、**整词匹配**）；**`accuracy`**（`typed` / `source` 对齐与边界）；**`wpm`**。


- **Phase 2（Autocorrect）**：**`autocorrect`**（词典命中、**`diff_function`** 与 **limit**）；**`furry_fixes`**（递归、早停以配合 limit）；**`minimum_mewtations`**（编辑距离型递归，理解题设给出的 **add/remove/substitute** 分支框架）。


- **Phase 3（Multiplayer）**：按题面实现 **`time_per_word`**、**`fastest_words`** 等（笔记中讨论了列表推导的嵌套顺序与可读性）。

### 运行方式

在 **`Cats/cats/`** 目录下：

```bash
python3 cats_gui.py  # 最终的可视化运行
python3 ok -q 01     # 每个子问题的测试
```

---

## 环境说明

- **Python 3**（以本地通过 `ok` 为准）。
- 各子项目依赖课程下发的 `gui_files/`、`ucb.py` 等，按官方说明运行即可。

---

## 致谢与学术诚信

- 项目框架、测试与资源来自 **CS 61A 课程方**；版权与再分发以伯克利课程材料为准。
- 各项目中**个人实现**版权归作者所有。公开仓库时请遵守注册学期的 **学术诚信政策**，避免发布供他人直接抄袭的完整标准答案。
