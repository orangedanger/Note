# Demo 设计草案：小型 ARPG 战斗训练房

## 当前版本
- 日期：2026-06-28
- 阶段：第一版范围锁定
- 目标：先做单机最小战斗闭环，后续再逐步扩展技能、AI、UI 和联机。

## 一句话
一个小型 ARPG 战斗训练房：玩家控制第三人称角色，在一个小场景里用一次普通攻击或一个基础技能命中木桩敌人，敌人扣血并在血量归零后死亡/隐藏，用日志和 Debug 显示验证战斗流程。

## 第一版核心目标
- 玩家能在测试场景中移动。
- 场景中有一个木桩敌人。
- 玩家按键触发一次攻击或技能。
- 攻击使用 `SphereTrace` 做命中检测。
- 命中后读取一条 `DataTable` 技能数据，应用伤害。
- 敌人的 `Attribute/HealthComponent` 扣血。
- 血量归零后敌人隐藏、销毁或打印死亡日志。
- 使用 `UE_LOG`、`DrawDebug`、`DebugString` 观察命中、伤害和剩余血量。

## 第一版不做
- 不做联机复制、RPC、`RepNotify`。
- 不做 AI 行为树、感知、追击、攻击。
- 不做正式 UMG 血条和技能冷却 UI。
- 不做复杂 GAS。
- 不做复杂背包、装备、成长系统。
- 不做开放世界和复杂关卡。
- 不做高复杂动画连招树。

## 玩法流程
1. 玩家进入测试场景。
2. 玩家移动到敌人附近。
3. 玩家按攻击键。
4. `CombatComponent` 发起 `SphereTrace`。
5. 如果命中敌人，从 `DataTable` 读取攻击数据。
6. `CombatComponent` 调用敌人 `AttributeComponent::ApplyDamage`。
7. `AttributeComponent` 修改 Health，并打印剩余血量。
8. Health 小于等于 `0` 时，敌人进入死亡状态，隐藏/销毁/打印死亡日志。

## 类和组件草案

### 玩家角色
- `ACombatDemoCharacter`
- 职责：
  - 承载移动和输入。
  - 持有 `UCombatComponent`。
  - 可选持有 `USkillComponent`，第一版可以只做轻量读取技能数据。

### 木桩敌人
- `ATrainingEnemy`
- 职责：
  - 放在测试场景中作为可攻击目标。
  - 持有 `UAttributeComponent`。
  - 第一版不主动移动、不攻击、不感知玩家。

### 属性组件
- `UAttributeComponent`
- 第一版字段：
  - `MaxHealth`
  - `Health`
  - `bDead`
- 第一版接口：
  - `ApplyDamage(float DamageAmount)`
  - `IsDead()`
  - `GetHealthPercent()`
- 职责边界：
  - 只负责属性和死亡状态。
  - 不负责 Trace。
  - 不负责输入。
  - 不负责 UI 逻辑。

### 战斗组件
- `UCombatComponent`
- 第一版职责：
  - 接收攻击请求。
  - 使用 `SphereTrace` 检测命中。
  - 根据技能数据决定伤害和范围。
  - 找到目标的 `UAttributeComponent` 并调用 `ApplyDamage`。
- 不做：
  - 不直接维护 Health。
  - 不直接操作 UI。
  - 不做联机 RPC。

### 技能数据
- `FSkillRow : FTableRowBase`
- 第一版字段：
  - `SkillId`
  - `Damage`
  - `Range`
  - `Cooldown`
- 第一版只配 1 条数据，例如：
  - RowName：`BasicAttack`
  - Damage：`25`
  - Range：`200`
  - Cooldown：`0.5`

## 输入规划
- 使用 Enhanced Input。
- 第一版动作：
  - `Move`
  - `Look` 或沿用模板默认视角。
  - `Attack`
- `Attack` 触发后调用玩家身上的 `CombatComponent`。

## 命中检测规划
- 第一版使用 `SphereTrace`。
- Trace 起点：玩家位置或武器/角色前方位置。
- Trace 方向：角色 ForwardVector。
- Trace 距离：从 `DataTable` 的 `Range` 读取。
- Debug：
  - 命中时打印目标名、伤害、剩余血量。
  - 绘制 Trace 轨迹或命中球体。

## 数据驱动规划
- 第一版使用 `DataTable`。
- 只做一张技能表，只配 1 条技能数据。
- 目标不是做复杂技能系统，而是证明攻击参数可以从表里读取。
- 后续扩展：
  - 多个技能行。
  - 消耗、冷却、范围类型。
  - 技能标签。
  - 动画或特效引用。

## 网络预留原则
- 第一版不实现联机。
- 但代码职责按未来联机方向拆：
  - 伤害统一通过 `ApplyDamage` 入口。
  - 战斗请求统一通过 `CombatComponent`。
  - 属性修改集中在 `AttributeComponent`。
- 暂不写空 RPC，避免过度设计。

## 第一版验收标准
- 有一个可运行测试场景。
- 玩家能移动。
- 场景中有一个木桩敌人。
- 玩家按键能触发攻击。
- `SphereTrace` 能命中敌人。
- 敌人血量能下降。
- 血量为 0 后能死亡、隐藏或销毁。
- 日志或 DebugString 能看到命中、伤害和剩余血量。

## 面试表达预留
- 这个 Demo 第一版刻意先做最小闭环：输入、命中检测、伤害结算、死亡反馈。
- 我把属性、战斗、技能数据拆开，是为了让职责边界清楚，也方便后续扩展联机。
- 第一版没有直接做联机和复杂 AI，是为了先验证核心 Gameplay 数据流，避免范围失控。

## 后续扩展顺序
1. 正式 UMG 血条和技能冷却。
2. 简单敌人追击和攻击。
3. 第二个技能和冷却。
4. 受击反馈、伤害数字、简单动画。
5. 网络同步：服务器权威伤害、血量复制、`RepNotify` 更新 UI。
