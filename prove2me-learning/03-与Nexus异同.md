# 与 Nexus 的异同

## Nexus 是什么

**AlphaProof Nexus 是 Google DeepMind 面向困难数学研究问题的自动形式证明搜索框架。** 它用 Gemini 生成和修改 Lean 证明草图；完整配置保留多份候选草图，通过评分选择有希望的路线继续搜索。子目标既可由 Gemini 直接证明，也可调用 AlphaProof，最终由 Lean 检查证明。[论文 §2](https://arxiv.org/html/2605.22763v2#S2)

## 大部分部件对应

两者都遵循 **“写草图 → 拆子目标 → 分别求解 → 验证与复用”** ，核心证明架构高度相近。

| 部件 | Nexus | Prove2Me |
|---|---|---|
| 写草图 | Gemini | 用户选择的 Agent |
| 待证子目标 | subgoal | Open 命题卡；关键目标可选作 milestone |
| 求解子目标 | Gemini 或 AlphaProof | 原 Agent、其他 Agent 或参与者 |
| 多条候选路线 | 草图种群及评分、选择机制 | 同一目标可有多份归约，搜索策略可外接 |
| 结果复用 | 跨候选草图共享子目标缓存 | 检索、引用自己或别人已证的命题 |
| 验证与汇合 | 接回草图并由 Lean 检查 | Lean 验证贡献，平台沿依赖更新上层状态 |

### 缓存也有对应

**Nexus 的完整配置会缓存已解决的子目标，供同一搜索种群中的其他草图复用。** 原草图即使没能证完整题，其中已证的子目标也可以被利用；缓存按完整的 **Lean 上下文和目标**匹配。[缓存机制](https://arxiv.org/html/2605.22763v2#A1.SS1.SSS0.Px6)

Prove2Me 则把可复用结果保存成独立命题，供不同参与者、不同项目检索和引用。**两者都有“已验证成果不必重做”的机制；主要差别是搜索内部缓存与长期共享命题库的组织方式。** [Prove2Me §4.3](https://arxiv.org/html/2608.28433v2#S4.SS3)

## 主要区别

- **草图的封装不同。** Nexus 的草图可以直接在文件内留 `sorry`；Prove2Me 提交的归约自身不留洞，把未完成义务放到独立的 Open 命题卡中。
- **默认配套不同。** Nexus 细化多草图搜索、评分和预算分配；Prove2Me 细化核心语义审核、开放接手、跨项目复用与贡献记录。

这些设计可以组合：将 Nexus 的未解子目标封装成命题卡，交给参与者求解后接回草图。主要需处理完整假设与定义的封装、Lean 环境匹配和证明回填。

## 时间与引用

**论文首次公开：** Nexus 为 **2026-05-21**，Prove2Me 为 **2026-08-28**；论文公开顺序是 Nexus 在前。Prove2Me 另有 2026 年 2 月的课程原型记录。[Nexus](https://arxiv.org/abs/2605.22763v2) · [Prove2Me](https://arxiv.org/abs/2608.28433v2) · [课程记录](https://www.henryyuen.net/classes/spring2026/)

所核对的两个 v2 版本未发现直接互引，共同引用有 **6 项**：Aristotle、早先的 AlphaProof 强化学习证明工作、GPT-f、Mathlib、LeanDojo，以及形式数学推理综述。这里的 AlphaProof 引用指早先工作。
