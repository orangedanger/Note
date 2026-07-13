# 发现记录

## 已确认
- 用户目标是为“大学毕业后的 1 年社招”做准备，方向是 UE C++ 游戏客户端开发。
- 文档需要写入 `D:\Note\Codex` 下的自建目录。
- 本地 UE 工程目录为 `D:\soft\UE_5.7\Engine`。
- `Build.version` 显示本地引擎为 Unreal Engine `5.7.4`，分支 `++UE5+Release-5.7`。
- 用户当前日期为 `2026-06-13`，仍在上班，预期 `2027-01` 初开始投递简历。
- 用户工作日需要上班，默认理解为周一到周五，工作时间约 `10:00-19:00/20:00`。

## 岗位能力归纳
- UE C++ 客户端社招通常看重：
  - 扎实 C++、数据结构和基础算法。
  - 能实现 Gameplay、UI、交互、角色、战斗等客户端功能。
  - 理解 Unreal Gameplay Framework、Actor/Component、UObject、反射、GC、资源加载。
  - 对多人项目岗位，要求理解 Replication、RPC、服务器权威、网络调试。
  - 对生产项目，性能分析、Debug、协作、代码质量和文档表达会显著影响评价。
- 1 年经验档位最重要的不是“知道很多名词”，而是能拿一个项目讲清楚：需求是什么、模块怎么拆、问题怎么查、方案有什么取舍、最后如何验证。

## 参考依据
- Epic 官方 Gameplay Framework 文档说明，UE 的 Gameplay Framework 提供 `GameMode`、`PlayerState`、`Controller`、`Pawn`、`Camera` 等核心游戏系统基础：https://dev.epicgames.com/documentation/unreal-engine/gameplay-framework-in-unreal-engine
- Epic 官方 Enhanced Input 文档说明，Enhanced Input 支持运行时添加/移除 Mapping Context，适合复杂输入、重映射和上下文输入：https://dev.epicgames.com/documentation/unreal-engine/enhanced-input-in-unreal-engine
- Epic 官方 Networking Overview 文档强调，如果项目可能需要多人能力，应尽早按多人思路设计 Gameplay，并说明 UE 使用 client-server 架构：https://dev.epicgames.com/documentation/unreal-engine/networking-overview-for-unreal-engine
- Epic 官方 Objects 文档说明 `UObject`、`UCLASS`、CDO、反射、GC、UHT、对象创建和销毁等机制：https://dev.epicgames.com/documentation/unreal-engine/objects-in-unreal-engine
- 岗位样本中常见关键词包括 C++、Gameplay、UI、Multiplayer Networking、CPU/Memory Profiling、Debugging、Performance、Perforce、Blueprint 协作、工具链和生产流程。

## 待确认问题
- 目标城市和目标公司类型：大厂、二线游戏厂、外包/项目制、独立团队、仿真/数字孪生 UE 岗。
- 当前基础水平：C++、算法、UE C++、蓝图、网络、渲染、项目经验各自处于什么阶段。
- 可投入时间：工作日晚间实际能稳定投入多少小时，周末是否能稳定投入半天以上。
- 是否已经有可展示项目、实习经历、开源代码或课程 Demo。
- 更偏 Gameplay、UI、战斗、网络、性能、工具、引擎源码还是渲染方向。

## 决策
- 路线先按“UE C++ 游戏客户端 / Gameplay 方向”制定，而不是偏渲染、引擎底层或 TA。
- 对 1 年社招准备，优先级为：作品项目 > C++ 和 UE 基础 > 面试题库 > 源码阅读深度 > 泛泛课程数量。
- Demo 必须能展示工程能力和排错能力，不能只停留在跟课复现。
- 因为用户在职，工作日安排轻量学习和小功能推进，周末安排主要开发、联调、打包、录屏和复盘。
- 将投递启动点暂定为 `2027-01-05` 左右；`2027-01-01` 到 `2027-01-10` 作为第一轮投递窗口。
- 能力评估采用 `0-4` 分制，避免只用“会/不会”导致结果太粗。
- Demo 方向探索先用问卷收敛兴趣和能力展示点，再决定具体项目，避免一开始就做过大的题材。
- 根据 2026-06 答卷，Demo 推荐方向暂定为“小型 ARPG 战斗 Demo”：战斗、技能/Buff/CD、敌人 AI、数据驱动为核心；单机闭环优先，联机同步作为第二阶段亮点。
- `2026-06-15` 至 `2026-06-21` 定为“概念校准周”，暂不推进复杂 Demo 功能，优先补 UE 反射/UHT/CDO/GC/Actor 生命周期和 C++ RAII/移动语义等高频短板。

## 风险
- 如果只刷题不做项目，社招深挖项目时容易露怯。
- 如果只做蓝图 Demo，不写 C++ 系统和文档，难以证明 UE C++ 客户端能力。
- 如果源码阅读没有问题导向，容易耗时巨大但转化不到面试表达。
- 如果项目范围过大，可能做不完；建议做小而完整的闭环。
- 在职备战容易被加班和疲劳打断，因此每周计划必须有保底版，避免因为某几天失败导致整周放弃。
- 用户对战斗、技能、AI 兴趣高，但 UI、动画、性能、表达意愿或经验相对弱；战斗 Demo 必须控制动画、UI 和联机范围。
