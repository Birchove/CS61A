# CS61A 课程仓库

本仓库用于归档加州大学伯克利分校 **CS 61A**（程序设计与抽象）相关的**多个课程项目**与笔记；除 Ants 外，后续将补充 **Hogs**、**Cats** 等子目录。

**知乎专栏（总入口）**：[CS 61A 相关专栏](https://www.zhihu.com/column/c_1981023312877483099)


---

## 关于 CS 61A

**CS 61A** 是伯克利面向初学者的程序设计课程，核心目标包括：

- 用 **Python** 与 **Scheme** 练习程序设计，强调**抽象**、**数据抽象**与**元循环求值**等思想；
- 教材与讲义体系多基于 *Composing Programs*（与经典 SICP 一脉相承）；
- 作业与项目体量较大，注重**阅读大段他人代码**、**按规格扩展**与**用自动评测（如 `ok`）回归测试**。

课程在北美高校本科路线中知名度较高，常被自学与转专业同学用作综合训练。（具体学期政策以伯克利官方为准。）

---

## 仓库结构（规划）

```text
CS61A/                          # 仓库根目录
├── README.md
├── .gitignore
├── Ants/
│   ├── ants_diagram.pdf        # 课程方提供的官方流程 / 设计图
│   └── ants/                   # Ants 工程（starter + 资源）
│       ├── ants.py             # 学生主要修改：游戏逻辑与各类蚂蚁
│       ├── gui.py
│       ├── ants_plans.py
│       ├── ok
│       ├── tests/
│       └── ...
├── Hogs/                       # （待补充）
└── Cats/                       # （待补充）
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

官方项目说明（第三方存档，便于访问）：[Ants Vs. SomeBees | CS 61A](https://insideempire.github.io/CS61A-Website-Archive/proj/ants/index.html)

### 本人的工作

以下内容与专栏文章一致，概括本人在 **`ants.py`** 中的实现与学习重点（**不含可照抄的完整题解代码**；细节与调试过程见上文单篇链接）：

- **Phase 1**：为 `HarvesterAnt` / `ThrowerAnt` 配置食物成本并实现收割；补全 `Place.__init__` 的 **entrance** 链；重写 `ThrowerAnt.nearest_bee`，沿 **entrance** 向前搜索、跳过仍位于 **Hive** 的蜜蜂。调试中区分了 **`None` 与 `False`** 等判断陷阱；理清 **部署（deploy）时才扣食物**、与仅实例化蚂蚁的区别。


- **Phase 2**：在 `ThrowerAnt` 上引入 **`lower_bound` / `upper_bound`**，实现 `ShortThrower` / `LongThrower`；修正循环条件，避免「距离未进入有效区间就永远不递增」的逻辑漏洞。实现 **`FireAnt.reduce_health`**：在必须用 **`super()`** 满足评测的前提下，处理**反弹伤害**与**阵亡额外伤害**，并避免对同一蜜蜂**重复调用** `reduce_health` 导致副作用重复。从零实现 **`WallAnt`**、**`HungryAnt`**（含咀嚼 **`cooldown`**，与 **`chew_cooldown`** 类属性的关系）。完成 **容器蚁** 三阶段：**`ContainerAnt`**（`store_ant`、`action`、`can_contain`）、**`Ant.add_to`** 的双蚁共存规则、**`BodyguardAnt`** 与 **`TankAnt`**（文章步骤与题面中的「护卫 / 容器」叙述一致）。总结抽象父类 → 调整基类行为 → 写具体子类的套路。


- **Phase 3**：**`Water.add_insect`** 与 **`is_waterproof`**；**`ScubaThrower`**；**`QueenAnt`**（身后隧道伤害加倍、`double` 与「只加倍一次」、阵亡调用 **`ants_lose()`** 等）。可选 **EC** 部分在笔记中有展开；


### 游戏规则与玩法（简要）

- **目标**：消耗食物在隧道各格放置蚂蚁，阻止蜜蜂到达蚁巢一端；消灭全部蜜蜂则胜利；若有蜜蜂抵达巢穴、或（存在时）**女王蚁**阵亡，则失败。
- **每回合大致顺序**：新蜜蜂可能从蜂巢进入 → 玩家放置蚂蚁 → **先蚂蚁后蜜蜂**依次执行各自 `action`。
- **蜜蜂**：若当前格未被蚂蚁**阻挡**，则向隧道**出口方向**移动；否则攻击该格蚂蚁。
- **蚂蚁**：不同类型收集食物、远程投掷、反伤、护佑队友等，由子类实现。

在 `ants/` 目录内启动 GUI 后，按终端提示在浏览器打开（常见为 `http://127.0.0.1:31415/`）。

```bash
python3 gui.py
python3 gui.py -d hard
python3 gui.py --water
python3 gui.py --food 10
python3 gui.py --help
```

自动评测示例：

```bash
python3 ok -q 01
python3 ok --score
```

### 环境说明

- **Python 3**（以本地通过 `ok` 为准）。
- GUI 依赖课程自带的 `libs/` 等。

### Ants 相关链接

| 说明 | 链接 |
|------|------|
| 本仓库内 Ants 学习笔记（单篇） | [知乎专栏文章](https://zhuanlan.zhihu.com/p/2001753505514414611) |
| Ants 官方项目页（存档） | [insideempire.github.io/…/proj/ants/index.html](https://insideempire.github.io/CS61A-Website-Archive/proj/ants/index.html) |
| 教材 *Composing Programs* | [composingprograms.com](https://www.composingprograms.com/) |

---

## 致谢与学术诚信

- 项目框架、官方流程图 **`ants_diagram.pdf`**、美术与测试来自 **CS 61A 课程方**；版权与再分发以伯克利课程材料为准。
- `ants.py` 中的**个人实现**版权归作者所有。公开仓库时请遵守注册学期的 **学术诚信政策**，避免发布供他人直接抄袭的完整标准答案。

---

*后续加入 Hogs、Cats 等项目时，可在「仓库结构」中更新树状图，并为每个项目增加与「## Ants」同级的 `##` 章节。*
