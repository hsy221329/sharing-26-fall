# Prove2Me 简介

**Prove2Me 把大证明组织成可接手、可验证、可复用的命题任务，让人类和 Agent 共同推进。** [官网](https://prove2.me/)

## 三方各做什么

- **人类：** 借助 Agent 回译，最终核对自然语言目标、关键定义与里程碑的形式化是否**语义对齐**，并确定证明方向（选做）。
- **Agent：**
	- 把自然语言写成 Lean 命题
	- 构造归约草图
	- 并求解各个待证命题卡。
- **Prove2Me：** 
	- 展示证明地图与进度
	- 调用 Lean 验证提交、追踪依赖
	- 支持检索和复用自己或他人已证的命题。

## 术语统一

| 术语 | 含义 |
|---|---|
| Mission／项目 | 围绕一个主目标组织的形式化工作 |
| Theorem／命题卡 | 一个固定的 Lean 命题声明，可以待证，也可以已证 |
| Milestone／里程碑 | 负责人选定的关键阶段目标，关联相应命题卡 |
| Proof sketch／归约草图 | 引用其他命题卡，形式化证明它们如何推出当前目标 |
## 三者如何关联

**命题卡是节点，proof sketch 连接推理依赖，milestone 标出项目的关键目标。**

- **命题卡：** 保存一个精确的 Lean 命题，主目标和中间引理都可以是一张卡。
- **归约草图：** 形式化证明“用 A、B 两张卡可以推出 T”。A 若仍很难，也可另写草图，用 A₁、A₂ 推出 A，递归展开。
- **里程碑：** 负责人选定的关键目标说明，关联一张被认可的命题卡；尚未形式化时可暂不关联。它让参与者围绕统一目标协作，Agent 可以提议、代建。

例如，T 的草图引用 A、B，并把 A、B 选为里程碑；A 再分解为 A₁、A₂。**实际被分解的是里程碑关联的命题卡，草图也可直接引用普通命题卡。** 项目是否完成看主目标是否已证。[里程碑规范](https://github.com/prove2me/prove2me_workspace/blob/main/references/missions.md#milestones)

### 里程碑有何价值

**里程碑给协作者一份统一的阶段任务清单：证明什么、采用哪张标准命题卡、哪些结果可以复用。** 它也让项目负责人确认原文中的关键引理已被准确形式化。相对审核所有中间卡，人工只看这一小组核心；若只关心主目标成立，证明图本身已经足够，里程碑的价值主要在协作与阶段成果的交付。[详细答疑](04-感受与QA.md#里程碑为何需要)

## 草图与 `sorry`

新建的待证命题卡先用 `by sorry` 占位；**外层归约草图自身没有 `sorry`，可以导入这些声明并通过编译；引用命题卡不要求它被列为里程碑。** 先搭好外层推理，再逐个补齐内层证明，系统据此更新状态。[归约协议](https://github.com/prove2me/prove2me_workspace/blob/main/references/prove.md#reductions-decompose-core-proofs)

## 结合例子看

### 数学问题

对任意实数 $a,b,c$，证明：

$$T:\quad (a+b+c)^2\leq3(a^2+b^2+c^2).$$

### 数学思路

把右侧减去左侧，化成平方和：

$$
\begin{aligned}
3(a^2+b^2+c^2)-(a+b+c)^2
&=(a-b)^2+(b-c)^2+(c-a)^2\\
&\geq0.
\end{aligned}
$$

**A 负责第一行的恒等式，B 负责第二行的非负性；A、B 合起来推出 T。**

### 先看外层草图

以下是本次实际提交的 proof sketch：前两个 `import` 引用 A、B，第三个提供 `linarith` 工具。

```lean
import Theorems.Thm_P2MVideoTake220261028Hsy_identity
import Theorems.Thm_P2MVideoTake220261028Hsy_nonneg
import Mathlib.Tactic.Linarith

theorem solution (a b c : ℝ) :
    (a + b + c) ^ 2 ≤ 3 * (a ^ 2 + b ^ 2 + c ^ 2) := by
  have h := P2MVideoTake220261028Hsy.nonneg a b c
  rw [← P2MVideoTake220261028Hsy.identity a b c] at h
  linarith
```

这份代码先取 B，再用 A 改写，最后移项得到 T。**外层没有 `sorry`，能通过编译；此时验证的是“由 A、B 推出 T”的推理。**

### 内层命题卡与里程碑

本例把 A、B 两张命题卡分别选为两个里程碑。编译外层前，先准备好它们的声明文件；初始证明均用 `by sorry` 占位。

**A：代数恒等式**，文件 `Theorems/Thm_P2MVideoTake220261028Hsy_identity.lean`：

```lean
import Mathlib.Data.Real.Basic

theorem P2MVideoTake220261028Hsy.identity (a b c : ℝ) :
    3 * (a ^ 2 + b ^ 2 + c ^ 2) - (a + b + c) ^ 2 =
      (a - b) ^ 2 + (b - c) ^ 2 + (c - a) ^ 2 := by sorry
```

**B：平方和非负**，文件 `Theorems/Thm_P2MVideoTake220261028Hsy_nonneg.lean`：

```lean
import Mathlib.Data.Real.Basic

theorem P2MVideoTake220261028Hsy.nonneg (a b c : ℝ) :
    0 ≤ (a - b) ^ 2 + (b - c) ^ 2 + (c - a) ^ 2 := by sorry
```

### 逐个补齐证明

A 用 `ring` 证明，B 用三个平方非负相加证明。分别提交后，平台检查并更新：

| 阶段 | A | B | T | 外层提交 |
|---|---|---|---|---|
| 只有占位声明 | Open | Open | Open | 尚未提交 |
| 外层归约通过 | Open | Open | Open | SKETCH |
| A 证明通过 | Proved | Open | Open | SKETCH |
| B 证明通过 | Proved | Proved | Proved | 原提交变为 PROOF |

**先完成外层组合，再补齐内层依赖；A、B 都已证后，T 自动完成。** 命题卡的已证状态叫 **Proved**，提交记录的完整证明标签叫 **PROOF**。

声明和实际证明分开保存，所以网页声明模板仍可能显示 `by sorry`；完成与否看状态及证明记录。

[试用演示](02-p2m试用演示.md) · [与 Nexus 对比](03-与Nexus异同.md) · [感受与问答](04-感受与QA.md)
