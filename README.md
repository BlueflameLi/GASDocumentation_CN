最近在学 GAS，但可惜几乎没官方文档，[tranek/GASDocumentation](https://github.com/tranek/GASDocumentation) 是个非常好的文档，但可惜是全英文的

中文翻译目前看到两个 

- [BillEliot/GASDocumentation_Chinese](https://github.com/BillEliot/GASDocumentation_Chinese) 的翻译很棒，不过已经很久没更新了
- [DriedMachine/GASDocumentation5.3_CN](https://github.com/DriedMachine/GASDocumentation5.3_CN) 虽然目前没有同步最新的文档，不过也是最新的 5.3 版本

出于学习 GAS 的目的，加上两位大佬的翻译都有些个小瑕疵，以及部分内容确实不太好理解（是我语文太差了），所以我打算边学 GAS，边翻译文档

翻译主要在 Deepseek 翻译的基础上，结合两位大佬的翻译进行修改

一些专有名词，如果有官方中文（虚幻引擎官方中文文档和引擎内中文），我就以官方的翻译为准，比如 "Gameplay 效果(Gameplay Effects)" 、"堆叠 (Stacking)"、"技能系统组件 (Ability System Component)"。但个别比较抽象的翻译就还是按我自己的理解来了，比如"玩法技能 (Gameplay Abilities)"，我还是按照 "GamePlay 技能 (Gameplay Abilities)" 来翻译了

我的翻译原则是除了变量名和函数名相关，其余能翻译的都翻译，不过考虑到引擎内各种 GAS 相关的配置项都还是英文，这些我会额外保留原名的，指向官方文档的链接也都会替换为默认中文的链接，并指定为 5.3 版本的

>一些注意点：
>
>- **Replicated** 直译是复制，虚幻引擎官方文档也是翻译为复制，但基本是特指网络同步中的复制，所以有的地方会翻译成同步、网络同步或者网络复制，我这就还是按照复制翻译，所以除非是明显表达 copy 意思，否则都是特指网络复制
>- 4.1 开头提到的 **自适应型网络更新频率 (Adaptive Network Update Frequency)** ，在 5.3 及以上的文档里已经没了，因此该链接修改为 5.2 版本的文档

# GAS 文档

我对 Unreal Engine 5 的 GameplayAbilitySystem 插件 (GAS)的理解，附带一个简单的多人示例项目。本文档非官方文档，项目与本人均与 Epic Games 无关联。不保证信息的准确性。

本文档目标是解释 GAS 中的主要概念和类，并根据我的使用经验提供额外说明。GAS 社区中存在许多"部落知识"（社区经验），我在此将分享所有积累的知识。

示例项目和文档基于 **Unreal Engine 5.3** (UE5)。旧版引擎有对应分支，但不再维护且可能存在错误或过时信息，请使用与引擎版本匹配的分支。

[GASShooter](https://github.com/tranek/GASShooter) 是一个姊妹示例项目，展示了 GAS 在多人 FPS/TPS 中的高级技巧。

最好的文档始终是插件源代码。

<a name="table-of-contents"></a>

## 目录

> 1. [GameplayAbilitySystem 插件简介](#intro)
> 1. [示例项目](#sp)
> 1. [使用 GAS 设置项目](#setup)
> 1. [核心概念](#concepts)  
>       4.1 [技能系统组件 (Ability System Component)](#concepts-asc)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.1.1 [复制模式 (Replication Mode)](#concepts-asc-rm)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.1.2 [设置与初始化](#concepts-asc-setup)  
>       4.2 [Gameplay标签 (Gameplay Tags)](#concepts-gt)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.2.1 [响应标签变化](#concepts-gt-change)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.2.2 [从插件 .ini文件加载 Gameplay标签](#concepts-gt-loadfromplugin)  
>       4.3 [属性 (Attributes)](#concepts-a)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.3.1 [属性定义](#concepts-a-definition)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.3.2 [基础值 vs 当前值 (BaseValue vs CurrentValue)](#concepts-a-value)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.3.3 [元属性 (Meta Attributes)](#concepts-a-meta)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.3.4 [响应属性变化](#concepts-a-changes)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.3.5 [派生属性 (Derived Attributes)](#concepts-a-derived)  
>       4.4 [属性集 (Attribute Set)](#concepts-as)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.1 [属性集定义](#concepts-as-definition)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2 [属性集设计](#concepts-as-design)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.1 [具有独立属性的子组件](#concepts-as-design-subcomponents)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.2 [运行时添加/移除属性集](#concepts-as-design-addremoveruntime)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.3 [物品属性（武器弹药）](#concepts-as-design-itemattributes)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.3.1 [在物品类中使用普通浮点数](#concepts-as-design-itemattributes-plainfloats)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.3.2 [在物品类中使用 `AttributeSet`](#concepts-as-design-itemattributes-attributeset)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.3.3 [在物品类中使用 `ASC`](#concepts-as-design-itemattributes-asc)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.3 [定义属性](#concepts-as-attributes)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.4 [初始化属性](#concepts-as-init)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.5 [PreAttributeChange()](#concepts-as-preattributechange)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.6 [PostGameplayEffectExecute()](#concepts-as-postgameplayeffectexecute)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.7 [OnAttributeAggregatorCreated()](#concepts-as-onattributeaggregatorcreated)  
>       4.5 [Gameplay效果 (Gameplay Effects)](#concepts-ge)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.1 [Gameplay效果定义](#concepts-ge-definition)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.2 [应用 Gameplay效果](#concepts-ge-applying)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.3 [移除 Gameplay效果](#concepts-ga-removing)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.4 [Gameplay效果修饰器 (Modifiers)](#concepts-ge-mods)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.4.1 [乘除修饰器 (Multiply and Divide Modifiers)](#concepts-ge-mods-multiplydivide)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.4.2 [修饰器上的游戏标签](#concepts-ge-mods-gameplaytags)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.5 [堆叠 (Stacking) Gameplay效果](#concepts-ge-stacking)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.6 [授予技能 (Granted Abilities)](#concepts-ge-ga)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.7 [Gameplay效果标签](#concepts-ge-tags)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.8 [免疫](#concepts-ge-immunity)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.9 [Gameplay效果规格 (Gameplay Effect Spec)](#concepts-ge-spec)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.9.1 [SetByCallers](#concepts-ge-spec-setbycaller)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.10 [Gameplay效果上下文 (Gameplay Effect Context)](#concepts-ge-context)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.11 [修饰量计算 (Modifier Magnitude Calculation)](#concepts-ge-mmc)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12 [Gameplay 效果执行计算 (Gameplay Effect Execution Calculation)](#concepts-ge-ec)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12.1 [向执行计算发送数据](#concepts-ge-ec-senddata)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12.1.1 [SetByCaller](#concepts-ge-ec-senddata-setbycaller)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12.1.2 [基于数据属性的计算修饰器 (Backing Data Attribute Calculation Modifier)](#concepts-ge-ec-senddata-backingdataattribute) 
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12.1.3 [基于临时变量的计算修饰器 (Backing Data Temporary Variable Calculation Modifier)](#concepts-ge-ec-senddata-backingdatatempvariable)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12.1.4 [Gameplay效果上下文 (Gameplay Effect Context)](#concepts-ge-ec-senddata-effectcontext)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.13 [自定义应用需求 (Custom Application Requirement)](#concepts-ge-car)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.14 [花费型 (Cost) Gameplay效果](#concepts-ge-cost)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.15 [冷却型 (Cooldown) Gameplay效果 ](#concepts-ge-cooldown)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.15.1 [获取 Gameplay效果剩余冷却时间](#concepts-ge-cooldown-tr)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.15.2 [监听冷却开始与结束](#concepts-ge-cooldown-listen)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.15.3 [预测冷却](#concepts-ge-cooldown-prediction)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.16 [修改激活的 Gameplay效果持续时间](#concepts-ge-duration)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.17 [运行时动态创建 Gameplay效果](#concepts-ge-dynamic)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.18 [Gameplay效果容器 (Gameplay Effect Containers)](#concepts-ge-containers)  
>       4.6 [GamePlay技能 (Gameplay Abilities)](#concepts-ga)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.1 [GamePlay技能定义](#concepts-ga-definition)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.1.1 [复制策略 (Replication Policy)](#concepts-ga-definition-reppolicy)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.1.2 [服务器端遵循远程技能撤销 (Server Respects Remote Ability Cancellation)](#concepts-ga-definition-remotecancel)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.1.3 [直接输入复制 (Replicate Input Directly)](#concepts-ga-definition-repinputdirectly)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.2 [绑定输入到 ASC](#concepts-ga-input)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.2.1 [绑定输入但不激活技能](#concepts-ga-input-noactivate)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.3 [授予技能](#concepts-ga-granting)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.4 [激活技能](#concepts-ga-activating)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.4.1 [被动能力](#concepts-ga-activating-passive)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.4.2 [激活失败标签](#concepts-ga-activating-failedtags)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.5 [撤销技能](#concepts-ga-cancelabilities)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.6 [获取激活的技能](#concepts-ga-definition-activeability)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.7 [实例化策略](#concepts-ga-instancing)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.8 [网络执行策略](#concepts-ga-net)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.9 [技能标签](#concepts-ga-tags)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.10 [Gameplay技能规格 (Gameplay Ability Spec)](#concepts-ga-spec)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.11 [向技能传递数据](#concepts-ga-data)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.12 [技能消耗与冷却](#concepts-ga-commit)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.13 [技能升级](#concepts-ga-leveling)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.14 [技能集合 (Ability Sets)](#concepts-ga-sets)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.15 [技能批处理 (Ability Batching)](#concepts-ga-batching)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.16 [网络安全策略 (Net Security Policy)](#concepts-ga-netsecuritypolicy)  
>       4.7 [技能任务 (Ability Tasks)](#concepts-at)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.7.1 [技能任务定义](#concepts-at-definition)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.7.2 [自定义技能任务](#concepts-at-definition)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.7.3 [使用技能任务](#concepts-at-using)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.7.4 [根运动源能力任务 (Root Motion Source Ability Tasks)](#concepts-at-rms)  
>       4.8 [Gameplay提示 (Gameplay Cues)](#concepts-gc)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.1 [Gameplay提示定义](#concepts-gc-definition)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.2 [触发 Gameplay提示](#concepts-gc-trigger)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.3 [本地 Gameplay提示](#concepts-gc-local)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.4 [Gameplay提示参数](#concepts-gc-parameters)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.5 [Gameplay提示管理器 (Gameplay Cue Manager)](#concepts-gc-manager)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.6 [阻止 Gameplay提示触发](#concepts-gc-prevention)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.7 [Gameplay提示批处理](#concepts-gc-batching)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.7.1 [手动 RPC](#concepts-gc-batching-manualrpc)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.7.2 [单个 GE 上的多个 GC](#concepts-gc-batching-gcsonge)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.8 [Gameplay提示事件 (Gameplay Cue Events)](#concepts-gc-events)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.9 [Gameplay提示可靠性](#concepts-gc-reliability)  
>       4.9 [技能系统全局 (Ability System Globals)](#concepts-asg)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.9.1 [InitGlobalData()](#concepts-asg-initglobaldata)  
>       4.10 [预测 (Prediction)](#concepts-p)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.10.1 [预测键 (Prediction Key)](#concepts-p-key)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.10.2 [在技能中创建新预测窗口](#concepts-p-windows)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.10.3 [预测生成 Actor](#concepts-p-spawn)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.10.4 [GAS 中预测的未来](#concepts-p-future)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.10.5 [网络预测插件 (Network Prediction Plugin)](#concepts-p-npp)  
>       4.11 [目标选择 (Targeting)](#concepts-targeting)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.11.1 [目标数据 (Target Data)](#concepts-targeting-data)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.11.2 [目标 Actor (Target Actors)](#concepts-targeting-actors)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.11.3 [目标数据过滤器 (Target Data Filters)](#concepts-target-data-filters)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.11.4 [Gameplay技能世界标线 (Gameplay Ability World Reticles)](#concepts-targeting-reticles)  
>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.11.5 [Gameplay效果容器的目标选择](#concepts-targeting-containers)  
> 1. [常见实现的能力与效果](#cae)  
>    5.1 [击晕 (Stun)](#cae-stun)  
>    5.2 [冲刺 (Sprint)](#cae-sprint)  
>    5.3 [瞄准 (Aim Down Sights)](#cae-ads)  
>    5.4 [生命窃取 (Lifesteal)](#cae-ls)  
>    5.5 [在客户端与服务端中生成随机数](#cae-random)  
>    5.6 [暴击 (Critical Hits)](#cae-crit)  
>    5.7 [非堆叠 Gameplay效果但仅最大值实际影响目标](#cae-nonstackingge)  
>    5.8 [游戏暂停时生成目标数据 (Target Data)](#cae-paused)  
>    5.9 [单按键交互系统 (One Button Interaction System)](#cae-onebuttoninteractionsystem)  
> 1. [调试 GAS](#debugging)  
>    6.1 [showdebug abilitysystem](#debugging-sd)  
>    6.2 [Gameplay 调试器 (Gameplay Debugger)](#debugging-gd)  
>    6.3 [GAS 日志 (Logging)](#debugging-log)  
> 1. [优化](#optimizations)  
>    7.1 [技能批处理](#optimizations-abilitybatching)  
>    7.2 [Gameplay提示批处理](#optimizations-gameplaycuebatching)  
>    7.3 [ASC 复制模式 (Replication Mode)](#optimizations-ascreplicationmode)  
>    7.4 [属性代理复制 (Attribute Proxy Replication)](#optimizations-attributeproxyreplication)  
>    7.5 [ASC 懒加载](#optimizations-asclazyloading)  
> 1. [易用性改进建议 (Quality of Life Suggestions)](#qol)  
>    8.1 [Gameplay效果容器](#qol-gameplayeffectcontainers)  
>    8.2 [将蓝图异步任务 (AsyncTask) 绑定到 ASC 委托](#qol-asynctasksascdelegates)  
> 1. [故障排除](#troubleshooting)  
>    9.1 [`LogAbilitySystem: Warning: Can't activate LocalOnly or LocalPredicted ability %s when not local!`](#troubleshooting-notlocal)  
>    9.2 [`ScriptStructCache` 错误](#troubleshooting-scriptstructcache)  
>    9.3 [动画蒙太奇未复制到客户端](#troubleshooting-replicatinganimmontages)  
>    9.4 [复制蓝图 Actor 时 AttributeSets 设为 nullptr](#troubleshooting-duplicatingblueprintactors)  
>    9.5 [未解析的外部符号 UEPushModelPrivate::MarkPropertyDirty(int,int)](#troubleshooting-unresolvedexternalsymbolmarkpropertydirty)  
>    9.6 [枚举名现在以路径名表示](#troubleshooting-enumnamesarenowpathnames)  
> 1. [常用 GAS 缩写](#acronyms)
> 1. [其他资源](#resources)  
>    11.1 [Epic Games 的 Dave Ratti 问答](#resources-daveratti)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;11.1.1 [社区问题 1](#resources-daveratti-community1)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;11.1.2 [社区问题 2](#resources-daveratti-community2)  
> 1. [GAS 更新日志](#changelog)  
>     * [5.3](#changelog-5.3)  
>     * [5.2](#changelog-5.2)  
>     * [5.1](#changelog-5.1)  
>     * [5.0](#changelog-5.0)  
>     * [4.27](#changelog-4.27)  
>     * [4.26](#changelog-4.26)  
>     * [4.25.1](#changelog-4.25.1)  
>     * [4.25](#changelog-4.25)  
>     * [4.24](#changelog-4.24)

<a name="intro"></a>
## 1. GameplayAbilitySystem 插件简介
[官方文档](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/gameplay-ability-system-for-unreal-engine?application_version=5.3)描述：

>**Gameplay 技能系统** 是一个高度灵活的框架，可用于构建你可能会在 RPG 或 MOBA 游戏中看到的技能和属性类型。你可以构建可供游戏中的角色使用的动作或被动技能，使这些动作导致各种属性累积或损耗的状态效果，实现约束这些动作使用的"冷却"计时器或资源消耗，更改技能等级及每个技能等级的技能效果，激活粒子或音效，等等。简单来说，此系统可帮助你在任何现代 RPG 或 MOBA 游戏中设计、实现及高效关联各种游戏中的技能，既包括跳跃等简单技能，也包括你喜欢的角色的复杂技能集。

GameplayAbilitySystem 插件由 Epic Games 开发，随 Unreal Engine 提供。已在《帕拉贡》《堡垒之夜》等多款商业 AAA 游戏中经过实战检验。

该插件为单人和多人游戏提供了开箱即用的解决方案：
* 实现基于等级的角色能力或技能，附带可选的花费型和冷却型机制 ([GameplayAbilities](#concepts-ga))
* 管理属于 Actor 的数值型 `Attributes` ([Attributes](#concepts-a))
* 为 Actor 应用状态效果 ([GameplayEffects](#concepts-ge))
* 为 Actor 应用 **Gameplay标签** ([GameplayTags](#concepts-gt))
* 生成视觉或声音效果 ([GameplayCues](#concepts-gc))
* 上述所有功能的网络复制 (Replication)

在多人游戏中，GAS 提供对以下内容的客户端预测 ([client-side prediction](#concepts-p) ) 支持:
* 技能激活
* 播放动画蒙太奇
*  对 **属性(Attributes)** 的修改
* 应用 `GameplayTags`
* 生成 `GameplayCues`
* 通过连接于 `CharacterMovementComponent` 的 `RootMotionSource` 函数形成的移动

**GAS 必须使用 C++ 创建**, 但是 `GameplayAbility` 和 `GameplayEffect` 可由设计师在蓝图中创建.

当前 GAS 存在的问题:
* `GameplayEffect` 延迟协调 (latency reconciliation)（无法预测技能冷却时间，导致高延迟玩家的低冷却技能发射频率低于低延迟玩家）
* 无法预测 `GameplayEffects` 的移除。但可以通过施加具有反向效果的 `GameplayEffects` 间接实现移除，但这并不总是适用或可行，问题依然存在。
* 缺乏样例模板、多人游戏示例和文档。希望本文档能有所帮助！

**[⬆ 返回顶部](#table-of-contents)**

<a name="sp"></a>

## 2. 示例项目
本文档附带一个多人第三人称射击游戏示例项目，主要面向**初次接触 GameplayAbilitySystem 插件但具备虚幻引擎基础**的开发者。要求使用者掌握 C++、蓝图 (Blueprints)、虚幻示意图形 (UMG)、网络复制 (Replication) 等中级知识。该项目演示了如何构建一个基础且适用于多人联机的第三人称射击游戏框架：

- 对于**玩家/AI 控制的英雄角色**， `AbilitySystemComponent` (`ASC`) 将放在 `PlayerState` 类上
- 对于 **AI 操控的小兵角色**， `ASC` 将放在 `Character` 类

>译者注：这么设计的原因参见后面的 [技能系统组件 (Ability System Component)](#concepts-asc)

该项目旨在保持结构简洁的同时，展示技能系统 (GAS) 的基础功能，并通过注释详尽的代码实现常用技能。因其面向新手，未涉及 [发射物预测 (predicting projectiles)](#concepts-p-spawn) 之类的高级内容。

演示概念：
* `PlayerState` 与 `Character` 上的 `ASC` 对比
* 复制 `Attributes`
* 复制动画蒙太奇
* `GameplayTags`
* 在 `GameplayAbilitys` 内部和外部应用和移除 `GameplayEffects`.
* 应用受护甲减伤后的伤害值以改变角色生命值
* Gameplay效果执行计算 (`GameplayEffectExecutionCalculations`)
* 眩晕效果 (Stun Effect)
* 死亡与重生机制
* 通过服务端技能生成 Actor（发射物）
* 当瞄准与冲刺时，预测性调整本地玩家速度
* 持续消耗耐力以维持冲刺状态
* 消耗法力值施放技能
* 被动技能 (Passive Abilities)
* 堆叠 `GameplayEffects`
* 目标选取（锁定 Actor）
* 在蓝图中创建 `GameplayAbilities` 
*  在 C++ 中创建 `GameplayAbilities`
* 按 `Actor` 独立实例化 `GameplayAbilities`
* 非实例化 `GameplayAbilities`（跳跃）
* 静态 `GameplayCues`（如枪械命中粒子特效）
* 基于 Actor 的 `GameplayCues`（冲刺与眩晕粒子特效）

英雄 (hero) 类包含以下技能：

| 技能             | 输入绑定   | 是否可预测 | C++ / 蓝图 | 技能描述                                                     |
| ---------------- | ---------- | ---------- | ---------- | ------------------------------------------------------------ |
| 跳跃             | 空格键     | 是         | C++        | 使英雄执行跳跃动作。                                         |
| 射击             | 鼠标左键   | 否         | C++        | 从英雄的枪械射出发射物。动画效果支持预测，但发射物生成无法预测。 |
| 瞄准（精准射击） | 鼠标右键   | 是         | 蓝图       | 按住鼠标右键时，英雄移速降低且镜头拉近，以提升枪械射击精度。 |
| 疾跑             | 左 Shift   | 是         | 蓝图       | 按住按键时，英雄加速移动并持续消耗耐力。                     |
| 向前冲刺         | Q          | 是         | 蓝图       | 英雄消耗耐力向前冲刺。                                       |
| 被动护甲叠加     | 无（被动） | 否         | 蓝图       | 每 4 秒获得 1 层护甲（上限 4 层），受到伤害时移除 1 层护甲。 |
| 陨石术           | R          | 否         | 蓝图       | 玩家指定目标位置召唤陨石，对敌人造成伤害并附加眩晕。目标选取可预测，陨石生成无法预测。 |

`GameplayAbilities` 通过 C++ 还是蓝图创建都没关系。本示例中混合使用两者，仅为演示不同语言环境下的实现方式。

小兵单位无任何预先定义的 `GameplayAbilities`。 红色小兵拥有更高的生命恢复速度而蓝色小兵有更高的初始生命值

对与 `GameplayAbility` 的命名，我使用 `_BP` 后缀来表示通过蓝图实现的 `GameplayAbility` 逻辑。无后缀则表示逻辑通过 C++ 实现.

**蓝图资源命名前缀规范**

| 前缀 | 资产类型        |
| ---- | --------------- |
| GA_  | GameplayAbility |
| GC_  | GameplayCue     |
| GE_  | GameplayEffect  |

**[⬆ 返回顶部](#table-of-contents)**

<a name="setup"></a>

## 3. 使用 GAS 设置项目
使用 GAS 配置项目的基础步骤:

1. 在编辑器中启用 GameplayAbilitySystem 插件
1. 修改 `YourProjectName.Build.cs` 文件，在你的 `PrivateDependencyModuleNames` 中添加 `"GameplayAbilities", "GameplayTags", "GameplayTasks"` 
1. 刷新/重新生成 Visual Studio 项目文件
1. 初始化全局数据（仅限引擎版本 4.24 至 5.2）
   - 若需使用 [`TargetData`](#concepts-targeting-data)，必须在代码中调用 `UAbilitySystemGlobals::Get().InitGlobalData()`。示例项目将此逻辑置于 `UAssetManager::StartInitialLoading()` 中。
   - **注意**：引擎 5.3 及以上版本会自动调用此函数。详见 [`InitGlobalData()`](#concepts-asg-initglobaldata) 。

完成上述步骤即可启用 GAS。接下来，为 `Character` 或 `PlayerState` 添加 [`ASC`](#concepts-asc) 和 [`AttributeSet`](#concepts-as)，即可开始制作 [`GameplayAbilities`](#concepts-ga) 与 [`GameplayEffects`](#concepts-ge)！

**[⬆ 回到顶部](#table-of-contents)**

<a name="concepts"></a>

## 4. 核心概念

#### 章节

> 4.1 [技能系统组件 (Ability System Component)](#concepts-asc)  
> 4.2 [游戏标签 (Gameplay Tags)](#concepts-gt)  
> 4.3 [属性 (Attributes)](#concepts-a)  
> 4.4 [属性集 (Attribute Set)](#concepts-as)  
> 4.5 [Gameplay效果 (Gameplay Effects)](#concepts-ge)  
> 4.6 [GamePlay技能 (Gameplay Abilities)](#concepts-ga)  
> 4.7 [技能任务 (Ability Tasks)](#concepts-at)  
> 4.8 [Gameplay提示 (Gameplay Cues)](#concepts-gc)  
> 4.9 [技能系统全局 (Ability System Globals)](#concepts-asg)  
> 4.10 [预测 (Prediction)](#concepts-p)

<a name="concepts-asc"></a>

### 4.1 技能系统组件 (Ability System Component)

**技能系统组件**（`AbilitySystemComponent`，简称 **ASC**）是 GAS 的核心模块。作为继承自 `UActorComponent` 的组件 ([`UAbilitySystemComponent`](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/API/Plugins/GameplayAbilities/UAbilitySystemComponent?application_version=5.3))，它负责处理与系统的所有交互。任何需要使用 [`GameplayAbilities`](#concepts-ga)、持有 [`Attributes`](#concepts-a) 或接收 [`GameplayEffects`](#concepts-ge) 的 `Actor` 都必须挂载 ASC。这些对象均存在于 `ASC` 内部，并由其管理和同步（`Attributes` 的同步由其 [`AttributeSet`](#concepts-as)) 处理）。开发者可（非必须）通过继承扩展 ASC 的功能。

附加了 `ASC` 的 `Actor` 被称为 `ASC` 的 `OwnerActor`。ASC 的物理表现 `Actor` 称为 `AvatarActor`。`OwnerActor` 和 `AvatarActor` 可以是同一个 `Actor`（例如 MOBA 游戏中的简单 AI 小兵），也可以是不同的 `Actor`（例如 MOBA 游戏中玩家控制的英雄，其中 `OwnerActor` 是 `PlayerState`，`AvatarActor` 是英雄的 `Character` 类）。大多数 `Actor` 都会将 `ASC` 附加在自己身上。若 `Actor` 需要重生且在重生时保留 `Attributes` 或 `GameplayEffects`（如 MOBA 中的英雄），则 `ASC` 的理想位置是放在 `PlayerState` 上。

**注意：** 若 `ASC` 放在 `PlayerState` 上，则需提高 `PlayerState` 的 `NetUpdateFrequency`。`PlayerState` 的默认值非常低，可能导致客户端同步 `Attributes` 和 `GameplayTags` 的变更时出现延迟或卡顿。请确保启用 [自适应型网络更新频率(`Adaptive Network Update Frequency`)](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/property-replication-in-unreal-engine?application_version=5.2#自适应型网络更新频率)（《堡垒之夜》就使用了此功能）。

>译者注：关于 `NetUpdateFrequency`
>
>- 服务器不会在每次更新时复制 actor。这会消耗太多的带宽和 CPU 资源。实际上，服务器会按照 `AActor::NetUpdateFrequency` 属性指定的频度来复制 actor。（参见 [Actor 的 Role 和 RemoteRole 属性](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/actor-role-and-remote-role-in-unreal-engine?application_version=5.3#复制模式))
>- 减少 `NetUpdateFrequency` 值：actor 的更新次数越少，更新所用的时间就越短。最好是尽量压低这个数值。该数值代表了这个 actor 每秒复制到客户端的频度（参见 [性能与带宽注意事项](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/performance-and-bandwidth-tips-for-unreal-engine?application_version=5.3)）
>- 如果需要使用 `NetUpdateFrequency`，直接在构造函数中设置即可，如 `NetUpdateFrequency = 100.f`，注意 100 是个很高的值，应根据实际情况调整

若 `OwnerActor` 和 `AvatarActor` 是不同 `Actor`，两者均应实现 `IAbilitySystemInterface` 接口。此接口需重写一个函数：`UAbilitySystemComponent* GetAbilitySystemComponent() const`，返回其 `ASC` 的指针。系统内部通过此接口函数实现 `ASC` 之间的交互。

`ASC` 将其当前生效的 `GameplayEffects` 存储在 `FActiveGameplayEffectsContainer ActiveGameplayEffects` 中。

`ASC` 将已授予的 `GameplayAbilities` 存储在 `FGameplayAbilitySpecContainer ActivatableAbilities` 中。若需遍历 `ActivatableAbilities.Items`，务必在循环前添加 `ABILITYLIST_SCOPE_LOCK();` 以锁定列表（防止因移除技能导致列表变化）。每个 `ABILITYLIST_SCOPE_LOCK();` 会递增 `AbilityScopeLockCount`，并在作用域结束后递减。**不要在 `ABILITYLIST_SCOPE_LOCK();` 作用域内尝试移除技能**（清除技能的函数内部会检查 `AbilityScopeLockCount`，防止列表锁定时移除技能）。

<a name="concepts-asc-rm"></a>

### 4.1.1 复制模式

 `ASC` 定义了三种不同的复制模式，用于同步 `GameplayEffects`、`GameplayTags` 和 `GameplayCues`：**`Full`（全复制）**、**`Mixed`（混合复制）** 和 **`Minimal`（最小复制）**。`Attributes` 的同步由其 `AttributeSet` 处理。

| 复制模式  | 适用场景                     | 描述                                                         |
| --------- | ---------------------------- | ------------------------------------------------------------ |
| `Full`    | 单机游戏                     | 所有 `GameplayEffect` 会同步到每个客户端。                   |
| `Mixed`   | 多人游戏，玩家控制的 `Actor` | `GameplayEffects` 仅同步到所属客户端。`GameplayTags` 和 `GameplayCues` 同步到所有客户端。 |
| `Minimal` | 多人游戏，AI 控制的 `Actor`  | `GameplayEffects` 不向任何客户端同步。仅 `GameplayTags` 和 `GameplayCues` 同步到所有客户端。 |

**注意：**

- **`Mixed` 模式要求**：`OwnerActor` 的 `Owner` 必须是 `Controller`。
  - `PlayerState` 的 `Owner` 默认是 `Controller`，但 `Character` 的 `Owner` 不是。
  - 若在 `OwnerActor` 不是 `PlayerState` 时使用 `Mixed` 模式，需调用 `SetOwner()` 手动设置。
- **引擎版本 4.24+**：`PossessedBy()` 现在会自动将 `Pawn` 的 `Owner` 设为新 `Controller`。

**[⬆ 回到顶部](#table-of-contents)**

<a name="concepts-asc-setup"></a>
### 4.1.2 设置与初始化

`ASCs` 通常在其 **`OwnerActor` 的构造函数** 中创建，并显式标记为可复制 (replicated)。**此操作必须在 C++ 中完成**。

```c++
AGDPlayerState::AGDPlayerState()
{
	// 创建技能系统组件，并显式设置为可复制
	AbilitySystemComponent = CreateDefaultSubobject<UGDAbilitySystemComponent>(TEXT("AbilitySystemComponent"));
	AbilitySystemComponent->SetIsReplicated(true);
	//...
}
```

ASC 需在服务器和客户端上通过 **`OwnerActor`** 和 **`AvatarActor`** 进行初始化。初始化应在**设置 `Pawn` 的 `Controller`（被控制 (Possession)）后** 执行。单机游戏只需处理服务器路径。

#### 场景 1：ASC 挂载在 Pawn 上（玩家控制角色）

- **服务器端初始化**：在 `Pawn` 的 `PossessedBy()` 函数中完成。
- **客户端初始化**：在 `PlayerController` 的 `AcknowledgePossession()` 函数中完成。

```c++
void APACharacterBase::PossessedBy(AController * NewController)
{
	Super::PossessedBy(NewController);

	if (AbilitySystemComponent)
	{
        // 初始化 ASC 的 OwnerActor 和 AvatarActor（此处均为 this）
		AbilitySystemComponent->InitAbilityActorInfo(this, this);
	}

	// Mixed 复制模式要求 OwnerActor 的 Owner 是 Controller
	SetOwner(NewController); // 译者注:4.24 版本后可省略
}
```

```c++
void APAPlayerControllerBase::AcknowledgePossession(APawn* P)
{
	Super::AcknowledgePossession(P);

	APACharacterBase* CharacterBase = Cast<APACharacterBase>(P);
	if (CharacterBase)
	{
		CharacterBase->GetAbilitySystemComponent()->InitAbilityActorInfo(CharacterBase, CharacterBase);
	}

	//...
}
```

#### 场景 2：ASC 挂载在 PlayerState 上（玩家控制角色）

- **服务器端初始化**：在 `Pawn` 的 `PossessedBy()` 函数中完成。
- **客户端初始化**：在 `Pawn` 的 `OnRep_PlayerState()` 函数中完成（确保客户端已存在 PlayerState）。

```c++
// 仅服务端
void AGDHeroCharacter::PossessedBy(AController * NewController)
{
	Super::PossessedBy(NewController);

	AGDPlayerState* PS = GetPlayerState<AGDPlayerState>();
	if (PS)
	{
		// 在服务端设置 ASC，客户端在 OnRep_PlayerState() 中处理
		AbilitySystemComponent = Cast<UGDAbilitySystemComponent>(PS->GetAbilitySystemComponent());

		// AI 没有 PlayerController，可再次初始化确保正确。初始化两次对具有 PlayerController 的英雄无害
		PS->GetAbilitySystemComponent()->InitAbilityActorInfo(PS, this);
	}
	
	//...
}
```

```c++
// 仅客户端
void AGDHeroCharacter::OnRep_PlayerState()
{
	Super::OnRep_PlayerState();

	AGDPlayerState* PS = GetPlayerState<AGDPlayerState>();
	if (PS)
	{
		// 客户端设置 ASC（服务器在 PossessedBy 中处理）
		AbilitySystemComponent = Cast<UGDAbilitySystemComponent>(PS->GetAbilitySystemComponent());

		// 客户端初始化 ASC 的 Actor 信息. 服务端将在控制一个新的 Actor 时初始化它的 ASC
		AbilitySystemComponent->InitAbilityActorInfo(PS, this);
	}

	// ...
}
```

若出现警告 **`LogAbilitySystem: Warning: Can't activate LocalOnly or LocalPredicted ability %s when not local!`**，表明 **客户端未正确初始化 ASC**。

**[⬆ 回到顶部](#table-of-contents)**

<a name="concepts-gt"></a>
### 4.2 Gameplay标签 (Gameplay Tags)

[`FGameplayTags`](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/GameplayTags/FGameplayTag?application_version=5.3) 是以 `Parent.Child.Grandchild...` 形式注册到 `GameplayTagManager` 的层级化命名标签。这些标签在对象状态分类与描述中极为实用。例如，若角色被眩晕，可在眩晕期间为其添加一个 `State.Debuff.Stun` 的 `GameplayTag` 。

开发者通常会逐步将原本用布尔值或枚举处理的逻辑替换为 `GameplayTags`，并通过判断对象是否拥有特定标签实现逻辑控制。

为对象赋予标签时，通常将其添加到其 `ASC`（若存在）中，以便 GAS 能够与之交互。`UAbilitySystemComponent` 实现了 `IGameplayTagAssetInterface` 接口，提供访问其持有的 `GameplayTags` 的功能。

多个 `GameplayTags` 可存储在 `FGameplayTagContainer` 中。相较于 `TArray<FGameplayTag>`，更推荐使用 `GameplayTagContainer`，因为后者具备高效优化机制。虽然标签本质是 `FName`，但若在项目设置中启用 `Fast Replication`（快速复制），`GameplayTagContainers` 可将标签高效打包以优化网络同步。启用此功能需确保服务器与客户端拥有相同的 `GameplayTags` 列表（通常无冲突，建议启用）。`GameplayTagContainers` 也可返回 `TArray<FGameplayTag>` 供遍历使用。

存储在 `FGameplayTagCountContainer` 中的 `GameplayTags` 拥有记录该标签实例数量的 `TagMap`。即使 `TagMapCount` 为零，`FGameplayTagCountContainer` 仍可能保留该 `GameplayTag`。调试时可能遇到 `ASC` 仍持有 `GameplayTag` 但 `TagMapCount` 为零的情况。`HasTag()`、`HasMatchingTag()` 等函数会检查 `TagMapCount`，若 `GameplayTag` 不存在或其 `TagMapCount` 为零则返回 false。

`GameplayTags` 必须预先在 `DefaultGameplayTags.ini` 中定义。虚幻引擎编辑器在项目设置中提供了管理界面，开发者无需手动编辑该文件即可管理 `GameplayTags`。通过 `GameplayTag` 编辑器可创建、重命名、搜索引用及删除 `GameplayTags`。

![GameplayTag Editor in Project Settings](https://github.com/tranek/GASDocumentation/raw/master/Images/gameplaytageditor.png)

搜索 `GameplayTag` 引用时，编辑器将显示开发者熟知的 **引用查看器 (`Reference Viewer`)** 图表界面，显示所有引用该标签的资源。但此方法不会显示引用该 `GameplayTag` 的 C++ 类

重命名 `GameplayTags` 会创建重定向，以便仍引用原始 `GameplayTag` 的资源可以重定向到新标签。如果可能，我更倾向于创建新 `GameplayTag`，手动将所有引用更新到新标签，然后删除旧标签以避免创建重定向。

除 `Fast Replication` 外，`GameplayTag` 编辑器还提供填充常用复制 `GameplayTags` 的选项以进一步优化性能。

`GameplayTag` 在通过 `GameplayEffect` 添加时会进行复制。`ASC` 允许添加不进行复制且需手动管理的 `LooseGameplayTag`（松散游戏标签）。示例项目使用  `LooseGameplayTag` 来作为 `State.Dead` ，以便拥有客户端在生命值归零时立即响应。重生时会手动将 `TagMapCount` 重置为零。仅在处理 `LooseGameplayTag` 时需要手动调整 `TagMapCount`。推荐使用 `UAbilitySystemComponent::AddLooseGameplayTag()` 和 `UAbilitySystemComponent::RemoveLooseGameplayTag()` 函数而非手动调整 `TagMapCount`。

在 C++ 中获取 `GameplayTag` 的引用：

```c++
FGameplayTag::RequestGameplayTag(FName("Your.GameplayTag.Name"))
```

若需进行高级操作，例如获取父/子标签，请使用 `GameplayTagManager` 提供的函数。要访问 `GameplayTagManager` 需包含 `GameplayTagManager.h` 头文件，并调用 `UGameplayTagManager::Get().FunctionName`。`GameplayTagManager` 实际上将 `GameplayTags` 存储为关系节点（父子级等），相比频繁的字符串操作和比较，此方式处理速度更快。

`GameplayTags` 和 `GameplayTagContainers` 可添加可选的 `UPROPERTY` 说明符 `Meta = (Categories = "GameplayCue")`，用于在蓝图中过滤标签，仅显示父标签为 `GameplayCue` 的 `GameplayTags`。当确定该 `GameplayTag` 或 `GameplayTagContainer` 变量仅用于 `GameplayCues`（游戏提示）时，此功能非常实用。

此外，另有一个独立结构体 `FGameplayCueTag`，其封装了 `FGameplayTag` 并自动在蓝图中过滤 `GameplayTags`，仅显示父标签为 `GameplayCue` 的标签。

若需在函数中过滤 `GameplayTag` 参数，可使用 `UFUNCTION` 说明符 `Meta = (GameplayTagFilter = "GameplayCue")`。函数中的 `GameplayTagContainer` 参数无法被过滤。如需修改引擎以实现此功能，请参考以下引擎代码实现：
- `Engine\Plugins\Editor\GameplayTagsEditor\Source\GameplayTagsEditor\Private\SGameplayTagGraphPin.cpp` 中 `SGameplayTagGraphPin::ParseDefaultValueData()` 如何调用 `FilterString = UGameplayTagsManager::Get().GetCategoriesMetaFromField(PinStructType);` 
- 以及 `SGameplayTagGraphPin::GetListContent()` 中如何将 `FilterString` 传递至 `SGameplayTagWidget`。
- `Engine\Plugins\Editor\GameplayTagsEditor\Source\GameplayTagsEditor\Private\SGameplayTagContainerGraphPin.cpp` 中对应的 `GameplayTagContainer` 版本函数未检查元字段属性，也未传递过滤器。

示例项目广泛使用了 `GameplayTags`。

**[⬆ 回到顶部](#table-of-contents)**

<a name="concepts-gt-change"></a>
### 4.2.1 响应标签变化

`ASC` 提供了在 `GameplayTag` 被添加或移除时触发的委托。该委托接收一个 `EGameplayTagEventType` 参数，可指定仅在 `GameplayTag` 被添加/移除时触发，或对 `TagMapCount` 的任何变更都作出响应。

```c++
AbilitySystemComponent->RegisterGameplayTagEvent(FGameplayTag::RequestGameplayTag(FName("State.Debuff.Stun")), EGameplayTagEventType::NewOrRemoved).AddUObject(this, &AGDPlayerState::StunTagChanged);
```

回调函数包含 `GameplayTag` 和新的 `TagCount` 参数：
```c++
virtual void StunTagChanged(const FGameplayTag CallbackTag, int32 NewCount);
```

**[⬆ 回到顶部](#table-of-contents)**

<a name="concepts-gt-loadfromplugin"></a>
### 4.2.2 从插件 .ini 文件加载游戏标签

若您创建的插件包含自定义的 `GameplayTags` 的 .ini 文件，可在插件的 `StartupModule()` 函数中加载该插件的 `GameplayTag` .ini 目录。

以下示例展示了虚幻引擎内置的 CommonConversation 插件实现方式：

```c++
void FCommonConversationRuntimeModule::StartupModule()
{
	TSharedPtr<IPlugin> ThisPlugin = IPluginManager::Get().FindPlugin(TEXT("CommonConversation"));
	check(ThisPlugin.IsValid());
	
	UGameplayTagsManager::Get().AddTagIniSearchPath(ThisPlugin->GetBaseDir() / TEXT("Config") / TEXT("Tags"));

	//...
}
```

当插件启用且引擎启动时，此代码会搜索 `Plugins\CommonConversation\Config\Tags` 目录，并将包含 `GameplayTags` 的 .ini 文件加载至项目中。

**[⬆ 回到顶部](#table-of-contents)**

<a name="concepts-a"></a>

### 4.3 属性 (Attributes)

<a name="concepts-a-definition"></a>
#### 4.3.1 属性定义

**属性 (Attributes)** 是由结构体 [`FGameplayAttributeData`]([Unreal Engine 5.3 Documentation | Unreal Engine 5.3 Documentation | Epic Developer Community](https://dev.epicgames.com/documentation/en-us/unreal-engine?application_version=5.3)) 定义的浮点数值。这些数值可表示从角色的生命值到角色等级，再到药水的充能次数等任意内容。若某个数值是 `Actor` 拥有的与玩法相关的值，则应考虑将其定义为 `Attribute`。`Attributes` 通常应仅通过 **Gameplay效果 ([`GameplayEffects`](#concepts-ge))** 修改，以便技能系统组件 (ASC) 能够预测 ([Predict](#concepts-p)) 其变更。

`Attributes` 由 **属性集([`AttributeSet`](#concepts-as))** 定义并存储其中。`AttributeSet` 负责复制 (Replicate) 标记为需复制的 `Attributes`。定义 `Attributes` 的方法请参阅 [`AttributeSets`](#concepts-as) 章节。

**提示：** 若希望某个 `Attribute` 不显示在编辑器的 `Attributes` 列表中，可使用 `Meta = (HideInDetailsView)` 属性说明符 (`Property Specifier`)。

**[⬆ 回到顶部](#table-of-contents)**

<a name="concepts-a-value"></a>

#### 4.3.2 基础值 vs 当前值 (BaseValue vs CurrentValue)

`Attribute` 由两个值构成 —— **基础值 (BaseValue)** 和 **当前值 (CurrentValue)**。`BaseValue` 是 `Attribute` 的永久值，而 `CurrentValue` 则是 `BaseValue` 加上来自 `GameplayEffects` 的临时修正。例如，你的角色可能拥有移动速度 `Attribute`，其 `BaseValue` 为 600 单位/秒。由于当前没有修改移速的 `GameplayEffects`，`CurrentValue` 也保持为 600 单位/秒。若角色获得临时 +50 单位/秒的移速增益，`BaseValue` 仍为 600 单位/秒，而 `CurrentValue` 将变为 600+50=650 单位/秒。当增益效果结束时，`CurrentValue` 会恢复至 `BaseValue` 600 单位/秒。

刚接触 GAS 的开发者常常会将 **基础值 (BaseValue)** 误认为是 `Attribute` 的最大值，并试图将其作为上限使用。这种做法是错误的。对于需要在技能或 UI 中引用或变更最大值的 `Attributes`，应当将其定义为独立的 `Attributes`。对于硬编码的最大/最小值，可通过定义包含 `FAttributeMetaData` 的 **数据表(DataTable)** 来实现，但 Epic 在该结构体上方的注释称其为"开发中功能"。更多信息请参考 `AttributeSet.h` 。为避免混淆，建议：

- 在技能或 UI 中需要引用的最大值应设为独立的 `Attributes`
- 仅用于限制 (Clamp) `Attributes` 范围的硬编码最大/最小值应定义为 `AttributeSet` 中的硬编码浮点数

关于 `Attributes` 限制 (Clamp) 的讨论参见：

- **[PreAttributeChange()](#concepts-as-preattributechange)**：处理 **当前值 (CurrentValue)** 变更
- **[PostGameplayEffectExecute()](#concepts-as-postgameplayeffectexecute)** ：处理来自 **Gameplay效果 (GameplayEffects)** 的 **基础值 (BaseValue)** 变更

对 `BaseValue` 的永久性修改来自**即时型 (Instant)** 的 `GameplayEffects`，而 **持续型 (Duration)** 和 **无限型 (Infinite)** 的 `GameplayEffects` 则修改 `CurrentValue`。**周期性 (Periodic)** 的 `GameplayEffects` 按 **即时型 (Instant)** 的 `GameplayEffects` 处理，会改变 `BaseValue`。

**[⬆ 回到顶部](#table-of-contents)**

<a name="concepts-a-meta"></a>
#### 4.3.3 元属性 (Meta Attributes)

一些 `Attributes` 被用作临时值的占位符，用于与其他 `Attributes` 交互。这些属性被称为 **元属性 (Meta Attributes)**。例如，我们通常将伤害值定义为 `Meta Attribute` 。我们不会让 `GameplayEffect` 直接修改生命值 `Attribute`，而是使用名为伤害值 (Damage) 的 `Meta Attribute` 作为占位符。这样，伤害值可以通过 [Gameplay效果执行计算 (`GameplayEffectExecutionCalculation`)](#concepts-ge-ec) 中的增益/减益效果进行修改，并可在 `AttributeSet` 中进一步处理，例如，先扣除护盾属性 (shield Attribute) 的值，再将剩余伤害从生命值属性中扣除。伤害值 `Meta Attribute` 不会在 `GameplayEffects` 之间持久保留，且每次都会被覆盖。`Meta Attributes` 通常不会被复制 (Replicated)。

`Meta Attributes` 为伤害、治疗等逻辑提供了良好的分离，区分了“我们造成了多少伤害？”和“如何处理这些伤害？”。这种逻辑分离意味着 `Gameplay Effects` 和 `Execution Calculations` 无需了解目标如何处理伤害。以伤害为例：`Gameplay Effect` 决定伤害量，`AttributeSet` 决定如何处理该伤害。并非所有角色都拥有相同的 `Attributes`，尤其是当使用子类的 `AttributeSets` 时。`AttributeSet` 基类可能仅包含生命值 `Attribute`，而继承的子类 `AttributeSet` 可能添加护盾值 `Attribute`。拥有护盾值 `Attribute` 的子类 `AttributeSet` 会以不同于基类  `AttributeSet` 的方式分配受到的伤害。

尽管 `Meta Attributes` 是优秀的设计模式，但并非强制使用。若所有伤害实例均使用同一个  `Execution Calculation`，且所有角色共享同一个 `Attribute Set`，则直接在 `Execution Calculation` 内部处理伤害分配（如生命值、护盾等）并直接修改这些 `Attributes` 是可行的。此方案仅会牺牲灵活性，但可能符合您的需求。 

**[⬆ 回到顶部](#table-of-contents)**

<a name="concepts-a-changes"></a>
#### 4.3.4 响应属性变化

要监听 `Attribute`（属性）变化并更新 UI 或触发其他游戏逻辑，可使用 `UAbilitySystemComponent::GetGameplayAttributeValueChangeDelegate(FGameplayAttribute Attribute)`。此函数返回一个 **委托 (Delegate)**，可绑定至目标函数，当 `Attribute` 变化时自动触发。该委托提供 `FOnAttributeChangeData` 参数，包含 `NewValue`（新值）、`OldValue`（旧值）和 `FGameplayEffectModCallbackData`（Gameplay效果修改回调数据）。**注意：**`FGameplayEffectModCallbackData` 仅服务器端有效。

```c++
AbilitySystemComponent->GetGameplayAttributeValueChangeDelegate(AttributeSetBase->GetHealthAttribute()).AddUObject(this, &AGDPlayerState::HealthChanged);
```

```c++
virtual void HealthChanged(const FOnAttributeChangeData& Data);
```

示例项目在 `GDPlayerState` 中绑定 `Attribute` 值变化委托，用于更新 HUD 并在生命值归零时触发玩家死亡逻辑。

示例项目还包含一个自定义蓝图节点，将此功能封装为 **异步任务 (`ASyncTask`)**，并在 `UI_HUD` UMG 控件中用于动态更新生命值、法力值和耐力值。此 `AsyncTask` 将一直持续运行直至手动调用 `EndTask()`，示例中在 UMG 控件的 **析构 (Destruct)** 事件中执行。详见 `AsyncTaskAttributeChanged.h/cpp`。

![Listen for Attribute Change BP Node](https://github.com/tranek/GASDocumentation/raw/master/Images/attributechange.png)

**[⬆ 回到顶部](#table-of-contents)**

<a name="concepts-a-derived"></a>
#### 4.3.5 派生属性 (Derived Attributes)

若需使一个 `Attribute` 的全部或部分值源自一个或多个其他 `Attributes`，可使用带有 `Attribute Based` 或 [`MMC`(Modifier Magnitude Calculation，修饰量计算)](#concepts-ge-mmc) [修饰符(Modifiers)](#concepts-ge-mods) 的 `Infinite` 型 `GameplayEffect`。当依赖的 `Attributes` 更新时，**派生属性 (Derived Attributes)** 将自动同步更新。

所有作用于 `Derived Attribute` 的 `Modifiers` 的最终计算公式与 **修饰器聚合器 (Modifier Aggregators)** 的公式相同。若需按特定顺序执行计算，应全部在 `MMC` 内部完成。

```
((CurrentValue + Additive) * Multiplicitive) / Division
```

**注意：** 若在 PIE（Play In Editor，编辑器中播放）中进行多客户端测试，需在编辑器偏好设置中禁用 `Run Under One Process`（在单个进程下运行）选项，否则除首个客户端外的其他客户端上的独立 `Attributes` 更新时，`Derived Attributes` 将无法同步更新。
本例中，通过一个 `Infinite` 型 `GameplayEffect`，使 `TestAttrA` 的值按公式 `TestAttrA = (TestAttrA + TestAttrB) * ( 2 * TestAttrC)` 从 `TestAttrB` 和 `TestAttrC` 派生。当任一依赖的 `Attributes` 更新时，`TestAttrA` 将自动重新计算其值。

![Derived Attribute Example](https://github.com/tranek/GASDocumentation/raw/master/Images/derivedattribute.png)

**[⬆ 回到顶部](#table-of-contents)**

<a name="concepts-as"></a>
### 4.4 属性集 (Attribute Set)

<a name="concepts-as-definition"></a>
#### 4.4.1 属性集定义

`AttributeSet` 用于定义、持有并管理 `Attributes` 的变更。开发者应从 [`UAttributeSet`](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Plugins/GameplayAbilities/UAttributeSet?application_version=5.3) 继承。在 `OwnerActor` 的构造函数中创建 `AttributeSet` 时，其会自动注册到对应的 `ASC`。**此操作必须通过 C++ 实现**。

**[⬆ 回到顶部](#table-of-contents)**

<a name="concepts-as-design"></a>

#### 4.4.2 属性集设计

一个 `ASC` 可以包含一个或多个 `AttributeSet`。属性集的内存开销可以忽略不计，因此具体使用多少个 `AttributeSets` 属于开发者的架构设计决策。

可接受的做法是：为游戏中所有 `Actor` 创建一个单一庞大的  `AttributeSet`，仅使用需要的属性而忽略未使用的属性。

另一种方案是：根据属性分组创建多个 `AttributeSet`，按需选择性地添加到 `Actor`。例如，可以创建管理生命相关属性的 `AttributeSet`、管理法力相关属性的 `AttributeSet` 等。在 MOBA 游戏中，英雄单位可能需要法力属性，而小兵单位可能不需要。因此英雄单位将包含法力 `AttributeSet`，小兵单位则不包含。

此外，可以通过继承 `AttributeSet` 来选择性决定 `Actor` 应具备哪些属性。属性在内部通过 `AttributeSetClassName.AttributeName` 的形式进行引用。当继承 `AttributeSet` 时，父类的所有属性仍会保留父类名称作为前缀。

虽然可以拥有多个 `AttributeSet`，但同一个 `ASC` 中不应存在多个同一类的 `AttributeSet`。若存在多个同一类的 `AttributeSet`，系统将无法确定应使用哪个 `AttributeSet`，并会随机选取其中

<a name="concepts-as-design-subcomponents"></a>
##### 4.4.2.1 具有独立属性的子组件

当 `Pawn` 上存在多个可损组件（如可单独受损的护甲部件）时，若已知 `Pawn` 可能拥有的最大可损组件数量，建议在单个 `AttributeSet` 中创建对应数量的生命值 `Attributes` —— 例如 `DamageableCompHealth0`、`DamageableCompHealth1` 等，以此表示这些可损组件的逻辑"插槽 (Slot)"。在可损组件类的实例中，分配一个可通过 `GameplayAbilities` 或 [Gameplay 效果执行 (`Executions`)](#concepts-ge-ec) 读取的插槽编号 `Attribute`，以此确定应对哪个 `Attribute` 应用伤害。即使 `Pawn` 实际拥有的可损组件数量少于最大值或为零，该方案仍可正常运作。属性集拥有某个属性并不意味着必须使用它，未使用的属性仅占用极少内存。

若你的每个子组件都需要携带大量 `Attributes`，或者子组件数量可以是无上限的，亦或者子组件可脱离并被其他玩家使用（如武器），或存在其他导致此方案不可行的情况，建议改用传统浮点数存储方案，而非 `Attributes`。具体实现可参考 [物品属性 (Item Attributes)](#concepts-as-design-itemattributes)。

<a name="concepts-as-design-addremoveruntime"></a>
##### 4.4.2.2 运行时添加/移除属性集

`AttributeSet` 可在运行时 (runtime) 动态添加或移除出 `ASC`。然而移除 `AttributeSets` 存在危险性。例如，一个客户端在服务端之前移除了某个 `AttributeSet`，而此时该 `Attribute` 的数值变更又被复制 (replicate) 到客户端，该属性将无法找到对应的 `AttributeSet`，从而导致游戏崩溃。

当武器被添加到库存时：

```c++
AbilitySystemComponent->GetSpawnedAttributes_Mutable().AddUnique(WeaponAttributeSetPointer);
AbilitySystemComponent->ForceReplication();
```

当武器从库存中被移除时：
```c++
AbilitySystemComponent->GetSpawnedAttributes_Mutable().Remove(WeaponAttributeSetPointer);
AbilitySystemComponent->ForceReplication();
```
<a name="concepts-as-design-itemattributes"></a>
##### 4.4.2.3 物品属性（武器弹药）

有几种方法可以实现带有 `Attributes` 的可装备的物品（武器弹药、护甲耐久度等）。所有这些方法都将数值直接存储在物品本身上。对于在其生命周期内可能被多个玩家装备的物品，这是必要的。

> 1. 在物品类中使用普通浮点数 (**推荐**)
> 1. 在物品类中使用独立的 `AttributeSet` 
> 1. 在物品类中使用独立的 `ASC`

<a name="concepts-as-design-itemattributes-plainfloats"></a>

###### 4.4.2.3.1 在物品类中使用普通浮点数

不使用 `Attributes`，而是在物品类实例上存储普通浮点数值。堡垒之夜和 [GASShooter](https://github.com/tranek/GASShooter) 就是这样处理武器弹药的。对于一把枪，直接在枪的实例上以可复制浮点数（`COND_OwnerOnly`）的形式存储最大弹匣容量、当前弹匣弹药、备用弹药等。如果多把武器共用同一备用弹药，你可以将备用弹药作为 `Attribute` 移动到角色身上，存储在角色共享的弹药 `AttributeSet` 中（换弹技能可以使用一个花费型 Gameplay效果 (`Cost GE`) 将弹药从备用弹药中转移到枪的浮点数弹匣弹药中）。因为你不使用 `Attributes` 来表示当前弹匣弹药，所以需要在 `UGameplayAbility` 中重写某些函数，以对枪上的浮点数弹药检查和应用花费。当授予技能时，将枪作为 [Gameplay技能规格 (`GameplayAbilitySpec`)](#concepts-ga-spec) 的 `SourceObject`（来源对象），这样在技能内部就能访问到授予该技能的枪。

为防止枪支在自动射击期间弹药量复制回传并覆盖本地弹药量，可在 `PreReplication()` 中当玩家持有 `IsFiring` `GameplayTag` 时禁用复制。这本质上是实现自定义本地预测。

```c++
void AGSWeapon::PreReplication(IRepChangedPropertyTracker& ChangedPropertyTracker)
{
	Super::PreReplication(ChangedPropertyTracker);

	DOREPLIFETIME_ACTIVE_OVERRIDE(AGSWeapon, PrimaryClipAmmo, (IsValid(AbilitySystemComponent) && !AbilitySystemComponent->HasMatchingGameplayTag(WeaponIsFiringTag)));
	DOREPLIFETIME_ACTIVE_OVERRIDE(AGSWeapon, SecondaryClipAmmo, (IsValid(AbilitySystemComponent) && !AbilitySystemComponent->HasMatchingGameplayTag(WeaponIsFiringTag)));
}
```

优势：
1. 避免了使用 `AttributeSets` 的局限（见下文）

局限:
1. 无法使用现用的 `GameplayEffect` 工作流（如用于弹药消耗的 `Cost GEs` 等）
1. 需重写 `UGameplayAbility` 上的关键函数来实现对枪上浮点数弹药的检查和应用花费

<a name="concepts-as-design-itemattributes-attributeset"></a>
###### 4.4.2.3.2 在物品类中使用  `AttributeSet`

在物品上类使用独立的 `AttributeSet`，可以通过 [在物品添加至玩家库存时添加到玩家的 `ASC`](#concepts-as-design-addremoveruntime) 实现，但存在一些主要限制。我在早期版本的 [GASShooter](https://github.com/tranek/GASShooter) 中曾以此方式实现武器弹药系统。武器将其 `Attributes` 如最大弹匣容量、当前弹匣弹药、备用弹药等存储在武器类的 `AttributeSet` 中。如果武器共用备用弹药，则将备用弹药移动到角色共享的弹药 `AttributeSet` 中。当在服务器上将武器添加到玩家库存时，武器会将其 `AttributeSet` 添加到玩家的 `ASC::SpawnedAttributes`。服务器随后将其复制到客户端。如果武器从库存中移除，则会从 `ASC::SpawnedAttributes` 中移除该 `AttributeSet`。

当 `AttributeSet` 存在于（除 `OwnerActor` 之外的）其他对象上（例如武器）时，最初会在 `AttributeSet` 中遇到一些编译错误。解决方法是在 `BeginPlay()` 中构造 `AttributeSet`，而不是在构造函数中，并在武器上实现 `IAbilitySystemInterface`（当你将武器添加到玩家库存时设置指向 `ASC` 的指针）。

```c++
void AGSWeapon::BeginPlay()
{
	if (!AttributeSet)
	{
		AttributeSet = NewObject<UGSWeaponAttributeSet>(this);
	}
	//...
}
```

具体实现可参考 [GASShooter历史版本](https://github.com/tranek/GASShooter/tree/df5949d0dd992bd3d76d4a728f370f2e2c827735)。

优势：
1. 可以使用现有的 `GameplayAbility` 和 `GameplayEffect` 工作流（如用于弹药消耗的 `Cost GEs` 等）
1. 对于极小数量的物品集设置简单

局限:
1. 你必须为每种武器类型创建一个新的 `AttributeSet` 类。`ASCs` 在功能上只能拥有每一个类的一个 `AttributeSet` 实例，因为对 `Attribute` 的更改会在 `ASCs` 的 `SpawnedAttributes` 数组中查找其 `AttributeSet` 类的第一个实例。相同 `AttributeSet` 类的其他实例将被忽略。
1. 由于前述每个 `AttributeSet` 类仅能有一个实例，因此玩家的库存中只能拥有每种类型的武器各一把。
1. 移除 `AttributeSet` 存在风险。在 GASShooter 中，如果玩家用火箭自杀，玩家会立即将火箭发射器从库存中移除（包括从 `ASC` 中移除其 `AttributeSet`）。当服务器复制火箭发射器的弹药 `Attribute` 更改时，该 `AttributeSet` 已不存在于客户端的 `ASC` 上，导致游戏崩溃。

<a name="concepts-as-design-itemattributes-asc"></a>
###### 4.4.2.3.3 在物品类中使用 `ASC`

把完整 `AbilitySystemComponent` 挂载在每个物品上是一种极端方案。本人未亲自实现过该方案，也未在实际项目中见过。此方案需要大量工程化工作才能正常运行。

> 能否让多个技能系统组件 (AbilitySystemComponents) 共享相同的 Owner 但是不同的 avatars（例如在 Pawn、武器/物品/投射物 上将其 Owner 设置为 PlayerState）？
>
> 首要问题在于 Owning Actor 需要实现 `IGameplayTagAssetInterface` 和 `IAbilitySystemInterface` 接口。前者或许可行：只需聚合所有 ASC 的标签即可（但要注意——`HasAllMatchingGameplayTags` 可能只能通过跨 ASC 聚合才能满足。仅将这些调用转发到每个 ASC 并合并结果是不够的）。但后者更棘手：哪个 ASC 才是权威？如果有人想应用一个 GE（Gameplay效果），应该由哪个 ASC 接收？也许这些问题都能解决，但 Owners 底下多个 ASC 共存引发的关联问题将是最难的。
>
> 不过，在 Pawn 和武器上使用独立的 ASC 本身是有意义的。例如，用于区分描述武器的标签与描述所有者 Pawn 的标签。或许确实可以让授予给武器的标签“应用”到所有者身上，而无需其他额外操作（例如属性和 GEs 可以相互独立，但所有者会像我上面描述的那样聚合这些标签）。我相信这能行得通，但在相同所有者下拥有多个 ASC 可能会变得棘手。

*Dave Ratti 来自 Epic 的回答，取自 [community questions #6](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89)*

优势：
1. 可以使用现有的 `GameplayAbility` 和 `GameplayEffect` 工作流（如用于弹药消耗的 `Cost GEs` 等）
1. 可复用 `AttributeSet` 类（在每把武器的 ASC 上各一个）

局限：
1. 工程实现成本未知
1. 可行性存疑

**[⬆ 回到顶部](#table-of-contents)**

<a name="concepts-as-attributes"></a>
#### 4.4.3 定义属性

**`Attributes` 只能通过 C++ 定义**，且必须定义在 `AttributeSet` 的头文件中。推荐在每个 `AttributeSet` 头文件的顶部添加如下宏块。它将自动为你的 `Attributes` 生成 getter 和 setter 函数。

```c++
// 使用 AttributeSet.h 中的宏
#define ATTRIBUTE_ACCESSORS(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)
```

一个可复制 (Replicated) 的生命值 (health) 属性可以这样定义：

```c++
UPROPERTY(BlueprintReadOnly, Category = "Health", ReplicatedUsing = OnRep_Health)
FGameplayAttributeData Health;
ATTRIBUTE_ACCESSORS(UGDAttributeSetBase, Health)
```

同时，在头文件中定义 `OnRep` 函数：
```c++
UFUNCTION()
virtual void OnRep_Health(const FGameplayAttributeData& OldHealth);
```

在 `AttributeSet` 的 .cpp 文件中，使用预测系统所需的 `GAMEPLAYATTRIBUTE_REPNOTIFY` 宏填写 `OnRep` 函数内容：

```c++
void UGDAttributeSetBase::OnRep_Health(const FGameplayAttributeData& OldHealth)
{
	GAMEPLAYATTRIBUTE_REPNOTIFY(UGDAttributeSetBase, Health, OldHealth);
}
```

最后，需要在 `GetLifetimeReplicatedProps` 中添加该 `Attribute`：
```c++
void UGDAttributeSetBase::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
	Super::GetLifetimeReplicatedProps(OutLifetimeProps);

	DOREPLIFETIME_CONDITION_NOTIFY(UGDAttributeSetBase, Health, COND_None, REPNOTIFY_Always);
}
```

`REPNOTIFY_Always` 表示即使本地值已经与从服务器复制下来的值相同（由于预测导致），也要触发 `OnRep` 函数。默认情况下，如果本地值与服务器同步下来的值相同，是不会触发 `OnRep` 函数的。

如果某个 `Attribute` 不需要复制（例如 `Meta Attribute`），则可以跳过 `OnRep` 和 `GetLifetimeReplicatedProps` 这两个步骤。

**[⬆ 回到顶部](#table-of-contents)**

<a name="concepts-as-init"></a>

#### 4.4.4 初始化属性

有多种方法可以初始化 `Attributes`（将其 `BaseValue` 以及相应的 `CurrentValue` 设置为某个初始值）。Epic 推荐使用即时型 (Instant) 的 `GameplayEffect`。示例项目也使用了这种方法。

请参考示例项目中的 `GE_HeroAttributes` 蓝图，了解如何制作一个用于初始化 `Attributes` 的即时 `GameplayEffect`。该 `GameplayEffect` 的应用在 C++ 中完成。

如果在定义 `Attributes` 时使用了 `ATTRIBUTE_ACCESSORS` 宏，则在 `AttributeSet` 上会为每个 `Attribute` 自动生成一个初始化函数，你可以在 C++ 中随时调用它。

```c++
// InitHealth(float InitialValue) 是一个为使用 `ATTRIBUTE_ACCESSORS` 宏定义的属性 'Health' 自动生成的函数
AttributeSet->InitHealth(100.0f);
```

更多初始化 `Attributes` 的方法可参考 `AttributeSet.h`。

**注意：** 在 4.24 版本之前，`FAttributeSetInitterDiscreteLevels` 无法与 `FGameplayAttributeData` 一起使用。它是在 `Attributes` 还是原始浮点数时创建的，并且在处理 `FGameplayAttributeData` 时会报错，提示其不是 `Plain Old Data`（POD，简单旧数据类型）。此问题已在 4.24 中修复：https://issues.unrealengine.com/issue/UE-76557。

**[⬆ 回到顶部](#table-of-contents)**

<a name="concepts-as-preattributechange"></a>
#### 4.4.5 PreAttributeChange()

`PreAttributeChange(const FGameplayAttribute& Attribute, float& NewValue)` 是 `AttributeSet` 中用于在 `Attribute` 的 `CurrentValue` 发生更改前响应更改的主要函数之一。它是通过引用参数 `NewValue` 对传入的 `CurrentValue` 更改进行限制 (clamp) 的理想位置。

例如，示例工程限制移动速度修饰符的方式如下：
```c++
if (Attribute == GetMoveSpeedAttribute())
{
	// 不能低于每秒 150 单位，且不能超过每秒 1000 单位
	NewValue = FMath::Clamp<float>(NewValue, 150, 1000);
}
```
`GetMoveSpeedAttribute()` 函数由我们在 `AttributeSet.h` 中添加的宏块生成（见 [定义属性](#concepts-as-attributes)）。

此函数会在对 `Attributes` 进行任何更改时触发，无论是使用由 `AttributeSet.h` 宏块定义的 `Attribute` 设置器 (setter)（见 [定义属性](#concepts-as-attributes)），还是通过 [`GameplayEffects`](#concepts-ge) 触发。

**注意：** 这里进行的任何限制 (clamping) 并不会永久更改 `ASC` 上的修饰器 (modifier)。它仅更改查询该修饰器时返回的值。这意味着，任何从所有修饰器重新计算 `CurrentValue` 的操作，比如 [`GameplayEffectExecutionCalculations`](#concepts-ge-ec) 和 [`ModifierMagnitudeCalculations`](#concepts-ge-mmc)，都需要再次实现限制 (clamping)。

**注意：** Epic 对 `PreAttributeChange()` 的注释指出，不要将其用于处理游戏玩法逻辑，而应主要用于限制 (clamping)。推荐在 `Attribute` 更改时处理游戏玩法逻辑的位置是 `UAbilitySystemComponent::GetGameplayAttributeValueChangeDelegate(FGameplayAttribute Attribute)`（见 [响应属性变化](#concepts-a-changes)）。

**[⬆ 回到顶部](#table-of-contents)**

<a name="concepts-as-postgameplayeffectexecute"></a>
#### 4.4.6 PostGameplayEffectExecute()

`PostGameplayEffectExecute(const FGameplayEffectModCallbackData & Data)` 仅在即时型 (Instant) [`GameplayEffect`](#concepts-ge) 导致的 `Attribute` 的 `BaseValue` 更改之后触发。这是对由 `GameplayEffect` 引起的更改进行更多 `Attribute` 操作的合适位置。

例如，在示例项目中，我们在此处将最终伤害值 `Meta Attribute` 从生命值 `Attribute` 中扣除。如果存在护盾值 `Attribute`，则会先从护盾值中扣除伤害值，再将剩余部分从生命值中扣除。示例项目还利用此位置来播放受击反应动画、显示浮动伤害数字，以及将经验值和金币奖励分配给击杀者。按设计，伤害值 `Meta Attribute` 始终通过即时型 (Instant) `GameplayEffect` 传递，而不会通过 `Attribute` 设置器 (setter)。

其他仅会因即时型 (Instant) `GameplayEffects` 更改其 `BaseValue` 的 `Attribute`（如法力值和耐力值）也可以在此处将其限制 (clamp) 到其最大值对应的 `Attribute` 上。

**注意：** 调用 `PostGameplayEffectExecute()` 时，`Attribute` 的更改已生效，但尚未复制到客户端，因此在此处进行限制不会导致向客户端发送两次网络更新。客户端仅会在限制后接收更新。

**[⬆ 回到顶部](#table-of-contents)**

<a name="concepts-as-onattributeaggregatorcreated"></a>

#### 4.4.7 OnAttributeAggregatorCreated()

`OnAttributeAggregatorCreated(const FGameplayAttribute& Attribute, FAggregator* NewAggregator)` 在此属性集中为某个 ` Attribute` 创建 **聚合器 (` Aggregator`)** 时触发。它允许自定义设置 [`FAggregatorEvaluateMetaData`](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Plugins/GameplayAbilities/FAggregatorEvaluateMetaData?application_version=5.3)。在评估某个 `Attribute` 的 `CurrentValue` 时，`Aggregator` 会根据所有作用在该属性上的 [`Modifiers`](#concepts-ge-mods) 来使用 `AggregatorEvaluateMetaData`。默认情况下，`AggregatorEvaluateMetaData` 仅由 `Aggregator` 用于确定哪些 `Modifiers` 符合条件，例如 `MostNegativeMod_AllPositiveMods`，它允许所有正向 `Modifiers`，但限制负向 `Modifiers` 仅保留最负向的一个。《虚幻争霸》曾使用此方式，仅允许对玩家应用最高的移动减速效果（无论同时存在多少减速效果），同时应用所有正向的移动速度增益效果。不符合条件的 `Modifiers` 仍然保留在 `ASC` 中，只是不被聚合到最终的 `CurrentValue`。一旦条件改变，例如最负向的 `Modifier` 过期，下一个最负向的 `Modifier`（如果存在）便会重新符合条件。

以下示例演示如何通过 `AggregatorEvaluateMetaData` 实现仅允许最负向 `Modifier ` 和所有正向 `Modifier`：

```c++
virtual void OnAttributeAggregatorCreated(const FGameplayAttribute& Attribute, FAggregator* NewAggregator) const override;
```

```c++
void UGSAttributeSetBase::OnAttributeAggregatorCreated(const FGameplayAttribute& Attribute, FAggregator* NewAggregator) const
{
	Super::OnAttributeAggregatorCreated(Attribute, NewAggregator);

	if (!NewAggregator)
	{
		return;
	}

	if (Attribute == GetMoveSpeedAttribute())
	{
		NewAggregator->EvaluationMetaData = &FAggregatorEvaluateMetaDataLibrary::MostNegativeMod_AllPositiveMods;
	}
}
```

用于限定条件的自定义 `AggregatorEvaluateMetaData` 应作为静态变量添加到 `FAggregatorEvaluateMetaDataLibrary` 中。

**[⬆ 回到顶部](#table-of-contents)**

<a name="concepts-ge"></a>
### 4.5 Gameplay Effects

<a name="concepts-ge-definition"></a>
#### 4.5.1 Gameplay Effect Definition
[`GameplayEffects`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayEffect/index.html) (`GE`) are the vessels through which abilities change [`Attributes`](#concepts-a) and [`GameplayTags`](#concepts-gt) on themselves and others. They can cause immediate `Attribute` changes like damage or healing or apply long term status buff/debuffs like a movespeed boost or stunning. The `UGameplayEffect` class is a meant to be a **data-only** class that defines a single gameplay effect. No additional logic should be added to `GameplayEffects`. Typically designers will create many Blueprint child classes of `UGameplayEffect`.

`GameplayEffects` change `Attributes` through [`Modifiers`](#concepts-ge-mods) and [`Executions` (`GameplayEffectExecutionCalculation`)](#concepts-ge-ec).

`GameplayEffects` have three types of duration: `Instant`, `Duration`, and `Infinite`.

Additionally, `GameplayEffects` can add/execute [`GameplayCues`](#concepts-gc). An `Instant` `GameplayEffect` will call `Execute` on the `GameplayCue` `GameplayTags` whereas a `Duration` or `Infinite` `GameplayEffect` will call `Add` and `Remove` on the `GameplayCue` `GameplayTags`.

| Duration Type | GameplayCue Event | When to use                                                  |
| ------------- | ----------------- | ------------------------------------------------------------ |
| `Instant`     | Execute           | For immediate permanent changes to `Attribute's` `BaseValue`. `GameplayTags` will not be applied, not even for a frame. |
| `Duration`    | Add & Remove      | For temporary changes to `Attribute's` `CurrentValue` and to apply `GameplayTags` that will be removed when the `GameplayEffect` expires or is manually removed. The duration is specified in the `UGameplayEffect` class/Blueprint. |
| `Infinite`    | Add & Remove      | For temporary changes to `Attribute's` `CurrentValue` and to apply `GameplayTags` that will be removed when the `GameplayEffect` is removed. These will never expire on their own and must be manually removed by an ability or the `ASC`. |

`Duration` and `Infinite` `GameplayEffects` have the option of applying `Periodic Effects` that apply its `Modifiers` and `Executions` every `X` seconds as defined by its `Period`. `Periodic Effects` are treated as `Instant` `GameplayEffects` when it comes to changing the `Attribute's` `BaseValue` and `Executing` `GameplayCues`. These are useful for damage over time (DOT) type effects. **Note:** `Periodic Effects` cannot be [predicted](#concepts-p).

`Duration` and `Infinite` `GameplayEffects` can be temporarily turned off and on after application if their `Ongoing Tag Requirements` are not met/met ([Gameplay Effect Tags](#concepts-ge-tags)). Turning off a `GameplayEffect` removes the effects of its `Modifiers` and applied `GameplayTags` but does not remove the `GameplayEffect`. Turning the `GameplayEffect` back on reapplies its `Modifiers` and `GameplayTags`.

If you need to manually recalculate the `Modifiers` of a `Duration` or `Infinite` `GameplayEffect` (say you have an `MMC` that uses data that doesn't come from `Attributes`), you can call `UAbilitySystemComponent::ActiveGameplayEffects.SetActiveGameplayEffectLevel(FActiveGameplayEffectHandle ActiveHandle, int32 NewLevel)` with the same level that it already has using `UAbilitySystemComponent::ActiveGameplayEffects.GetActiveGameplayEffect(ActiveHandle).Spec.GetLevel()`. `Modifiers` that are based on backing `Attributes` automatically update when those backing `Attributes` update. The key functions of `SetActiveGameplayEffectLevel()` to update the `Modifiers` are:

```C++
MarkItemDirty(Effect);
Effect.Spec.CalculateModifierMagnitudes();
// Private function otherwise we'd call these three functions without needing to set the level to what it already is
UpdateAllAggregatorModMagnitudes(Effect);
```

`GameplayEffects` are not typically instantiated. When an ability or `ASC` wants to apply a `GameplayEffect`, it creates a [`GameplayEffectSpec`](#concepts-ge-spec) from the `GameplayEffect's` `ClassDefaultObject`. Successfully applied `GameplayEffectSpecs` are then added to a new struct called `FActiveGameplayEffect` which is what the `ASC` keeps track of in a special container struct called `ActiveGameplayEffects`.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ge-applying"></a>
#### 4.5.2 Applying Gameplay Effects
`GameplayEffects` can be applied in many ways from functions on [`GameplayAbilities`](#concepts-ga) and functions on the `ASC` and usually take the form of `ApplyGameplayEffectTo`. The different functions are essentially convenience functions that will eventually call `UAbilitySystemComponent::ApplyGameplayEffectSpecToSelf()` on the `Target`.

To apply `GameplayEffects` outside of a `GameplayAbility` for example from a projectile, you need to get the `Target's` `ASC` and use one of its functions to `ApplyGameplayEffectToSelf`.

You can listen for when any `Duration` or `Infinite` `GameplayEffects` are applied to an `ASC` by binding to its delegate:
```c++
AbilitySystemComponent->OnActiveGameplayEffectAddedDelegateToSelf.AddUObject(this, &APACharacterBase::OnActiveGameplayEffectAddedCallback);
```
The callback function:
```c++
virtual void OnActiveGameplayEffectAddedCallback(UAbilitySystemComponent* Target, const FGameplayEffectSpec& SpecApplied, FActiveGameplayEffectHandle ActiveHandle);
```

The server will always call this function regardless of replication mode. The autonomous proxy will only call this for replicated `GameplayEffects` in `Full` and `Mixed` replication modes. Simulated proxies will only call this in `Full` [replication mode](#concepts-asc-rm).

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ga-removing"></a>
#### 4.5.3 Removing Gameplay Effects
`GameplayEffects` can be removed in many ways from functions on [`GameplayAbilities`](#concepts-ga) and functions on the `ASC` and usually take the form of `RemoveActiveGameplayEffect`. The different functions are essentially convenience functions that will eventually call `FActiveGameplayEffectsContainer::RemoveActiveEffects()` on the `Target`.

To remove `GameplayEffects` outside of a `GameplayAbility`, you need to get the `Target's` `ASC` and use one of its functions to `RemoveActiveGameplayEffect`.

You can listen for when any `Duration` or `Infinite` `GameplayEffects` are removed from an `ASC` by binding to its delegate:
```c++
AbilitySystemComponent->OnAnyGameplayEffectRemovedDelegate().AddUObject(this, &APACharacterBase::OnRemoveGameplayEffectCallback);
```
The callback function:
```c++
virtual void OnRemoveGameplayEffectCallback(const FActiveGameplayEffect& EffectRemoved);
```

The server will always call this function regardless of replication mode. The autonomous proxy will only call this for replicated `GameplayEffects` in `Full` and `Mixed` replication modes. Simulated proxies will only call this in `Full` [replication mode](#concepts-asc-rm).

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ge-mods"></a>
#### 4.5.4 Gameplay Effect Modifiers
`Modifiers` change an `Attribute` and are the only way to [predictively](#concepts-p) change an `Attribute`. A `GameplayEffect` can have zero or many `Modifiers`. Each `Modifier` is responsible for changing only one `Attribute` via a specified operation.

| Operation  | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| `Add`      | Adds the result to the `Modifier's` specified `Attribute`. Use a negative value for subtraction. |
| `Multiply` | Multiplies the result to the `Modifier's` specified `Attribute`. |
| `Divide`   | Divides the result against the `Modifier's` specified `Attribute`. |
| `Override` | Overrides the `Modifier's` specified `Attribute` with the result. |

The `CurrentValue` of an `Attribute` is the aggregate result of all of its `Modifiers` added to its `BaseValue`. The formula for how `Modifiers` are aggregated is defined as follows in `FAggregatorModChannel::EvaluateWithBase` in `GameplayEffectAggregator.cpp`:
```c++
((InlineBaseValue + Additive) * Multiplicitive) / Division
```

Any `Override` `Modifiers` will override the final value with the last applied `Modifier` taking precedence.

**Note:** For percentage based changes, make sure to use the `Multiply` operation so that it happens after addition.

**Note:** [Prediction](#concepts-p) has trouble with percentage changes.

There are four types of `Modifiers`: Scalable Float, Attribute Based, Custom Calculation Class, and Set By Caller. They all generate some float value that is then used to change the specified `Attribute` of the `Modifier` based on its operation.

| `Modifier` Type            | Description                                                  |
| -------------------------- | ------------------------------------------------------------ |
| `Scalable Float`           | `FScalableFloats` are a structure that can point to a Data Table that has the variables as rows and levels as columns. The Scalable Floats will automatically read the value of the specified table row at the ability's current level (or different level if overriden on the [`GameplayEffectSpec`](#concepts-ge-spec)). This value can further be manipulated by a coefficient. If no Data Table/Row is specified, it treats the value as a 1 so the coefficient can be used to hard code in a single value at all levels. ![ScalableFloat](https://github.com/tranek/GASDocumentation/raw/master/Images/scalablefloats.png) |
| `Attribute Based`          | `Attribute Based` `Modifiers` take the `CurrentValue` or `BaseValue` of a backing `Attribute` on the `Source` (who created the `GameplayEffectSpec`) or `Target` (who received the `GameplayEffectSpec`) and further modifies it with a coefficient and pre and post coefficient additions. `Snapshotting` means the backing `Attribute` is captured when the `GameplayEffectSpec` is created whereas no snapshotting means the `Attribute` is captured when the `GameplayEffectSpec` is applied. |
| `Custom Calculation Class` | `Custom Calculation Class` provides the most flexibility for complex `Modifiers`. This `Modifier` takes a [`ModifierMagnitudeCalculation`](#concepts-ge-mmc) class and can further manipulate the resulting float value with a coefficient and pre and post coefficient additions. |
| `Set By Caller`            | `SetByCaller` `Modifiers` are values that are set outside of the `GameplayEffect` at runtime by the ability or whoever made the `GameplayEffectSpec` on the `GameplayEffectSpec`. For example, you would use a `SetByCaller` if you want to set the damage to be based on how long the player held down a button to charge the ability. `SetByCallers` are essentially `TMap<FGameplayTag, float>` that live on the `GameplayEffectSpec`. The `Modifier` is just telling the `Aggregator` to look for a `SetByCaller` value associated with the supplied `GameplayTag`. The `SetByCallers` used by `Modifiers` can only use the `GameplayTag` version of the concept. The `FName` version is disabled here. If the `Modifier` is set to `SetByCaller` but a `SetByCaller` with the correct `GameplayTag` does not exist on the `GameplayEffectSpec`, the game will throw a runtime error and return a value of 0. This might cause issues in the case of a `Divide` operation. See [`SetByCallers`](#concepts-ge-spec-setbycaller) for more information on how to use `SetByCallers`. |

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ge-mods-multiplydivide"></a>
##### 4.5.4.1 Multiply and Divide Modifiers
By default, all `Multiply` and `Divide` `Modifiers` are added together before multiplying or dividing them into the `Attribute`'s `BaseValue`.

```c++
float FAggregatorModChannel::EvaluateWithBase(float InlineBaseValue, const FAggregatorEvaluateParameters& Parameters) const
{
	...
	float Additive = SumMods(Mods[EGameplayModOp::Additive], GameplayEffectUtilities::GetModifierBiasByModifierOp(EGameplayModOp::Additive), Parameters);
	float Multiplicitive = SumMods(Mods[EGameplayModOp::Multiplicitive], GameplayEffectUtilities::GetModifierBiasByModifierOp(EGameplayModOp::Multiplicitive), Parameters);
	float Division = SumMods(Mods[EGameplayModOp::Division], GameplayEffectUtilities::GetModifierBiasByModifierOp(EGameplayModOp::Division), Parameters);
	...
	return ((InlineBaseValue + Additive) * Multiplicitive) / Division;
	...
}
```

```c++
float FAggregatorModChannel::SumMods(const TArray<FAggregatorMod>& InMods, float Bias, const FAggregatorEvaluateParameters& Parameters)
{
	float Sum = Bias;

	for (const FAggregatorMod& Mod : InMods)
	{
		if (Mod.Qualifies())
		{
			Sum += (Mod.EvaluatedMagnitude - Bias);
		}
	}

	return Sum;
}
```
*from `GameplayEffectAggregator.cpp`*

Both `Multiply` and `Divide` `Modifiers` have a `Bias` value of `1` in this formula (`Addition` has a `Bias` of `0`). So it would look something like:

```
1 + (Mod1.Magnitude - 1) + (Mod2.Magnitude - 1) + ...
```

This formula leads to some unexpected results. Firstly, this formula adds all the modifiers together before multiplying or dividing them into the `BaseValue`. Most people would expect it to multiply or divide them together. For example, if you have two `Multiply` modifiers of `1.5`, most people would expect the `BaseValue` to be multiplied by `1.5 x 1.5 = 2.25`. Instead, this adds the `1.5`s together to multiply the `BaseValue` by `2` (`50% increase + another 50% increase = 100% increase`). This was for the example from `GameplayPrediction.h` of a `10%` speed buff on `500` base speed would be `550`. Add another `10%` speed buff and it will be `600`.

Secondly, this formula has some undocumented rules about what values can be used as it was designed with Paragon in mind.

Rules for `Multiply` and `Divide` multiplication addition formula:
* `(No more than one value < 1) AND (Any number of values [1, 2))`
* `OR (One value >= 2)`

The `Bias` in the formula basically subtracts out the integer digit of numbers in the range `[1, 2)`. The first `Modifier`'s `Bias` subtracts out from the starting `Sum` value (set to the `Bias` before the loop) which is why any value by itself works and why one value `< 1` will work with the numbers in the range `[1, 2)`.

Some examples with `Multiply`:  
Multipliers: `0.5`  
`1 + (0.5 - 1) = 0.5`, correct

Multipliers: `0.5, 0.5`  
`1 + (0.5 - 1) + (0.5 - 1) = 0`, incorrect expected `1`? Multiple values less than `1` don't make sense for adding multipliers. Paragon was designed to only use the [greatest negative value for `Multiply` `Modifiers`](#cae-nonstackingge) so there would only ever be at most one value less than `1` multiplying into the `BaseValue`.

Multipliers: `1.1, 0.5`  
`1 + (0.5 - 1) + (1.1 - 1) = 0.6`, correct

Multipliers: `5, 5`  
`1 + (5 - 1) + (5 - 1) = 9`, incorrect expected `10`. Will always be the `sum of the Modifiers - number of Modifiers + 1`.

Many games will want their `Multiply` and `Divide` `Modifiers` to multiply and divide together before applying to the `BaseValue`. To achieve this, you will need to **change the engine code** for `FAggregatorModChannel::EvaluateWithBase()`.

```c++
float FAggregatorModChannel::EvaluateWithBase(float InlineBaseValue, const FAggregatorEvaluateParameters& Parameters) const
{
	...
	float Multiplicitive = MultiplyMods(Mods[EGameplayModOp::Multiplicitive], Parameters);
	float Division = MultiplyMods(Mods[EGameplayModOp::Division], Parameters);
	...

	return ((InlineBaseValue + Additive) * Multiplicitive) / Division;
}
```

```c++
float FAggregatorModChannel::MultiplyMods(const TArray<FAggregatorMod>& InMods, const FAggregatorEvaluateParameters& Parameters)
{
	float Multiplier = 1.0f;

	for (const FAggregatorMod& Mod : InMods)
	{
		if (Mod.Qualifies())
		{
			Multiplier *= Mod.EvaluatedMagnitude;
		}
	}

	return Multiplier;
}
```

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ge-mods-gameplaytags"></a>
##### 4.5.4.2 Gameplay Tags on Modifiers

`SourceTags` and `TargetTags` can be set for each [Modifier](#concepts-ge-mods). They work the same like the [`Application Tag requirements`](#concepts-ge-tags) of a `GameplayEffect`. So the tags are considered only when the effect is applied. I.e. when having a periodic, infinite effect, they are only taken into consideration on the first application of the effect but *not* on each periodic execution.

`Attribute Based` Modifiers can also set `SourceTagFilter` and `TargetTagFilter`. When determining the magnitude of the attribute which is the source of the `Attribute Based` Modifier, these filters are used to exclude certain Modifiers to that attribute. Modifiers which source or target didn't have all of the tags of the filter are excluded.

This means in detail: The tags of the source ASC and the target ASC are captured by `GameplayEffects`. The source ASC tags are captured, when the `GameplayEffectSpec` is created, the target ASC tags are captured on execution of the effect. When determining, if a Modifier of an infinite or duration effect "qualifies" to be applied (i.e. its Aggregator qualifies) and those filters are set, the captured tags are compared against the filters.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ge-stacking"></a>
#### 4.5.5 Stacking Gameplay Effects
`GameplayEffects` by default will apply new instances of the `GameplayEffectSpec` that don't know or care about previously existing instances of the `GameplayEffectSpec` on application. `GameplayEffects` can be set to stack where instead of a new instance of the `GameplayEffectSpec` is added, the currently existing `GameplayEffectSpec's` stack count is changed. Stacking only works for `Duration` and `Infinite` `GameplayEffects`.

There are two types of stacking: Aggregate by Source and Aggregate by Target.

| Stacking Type       | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| Aggregate by Source | There is a separate instance of stacks per Source `ASC` on the Target. Each Source can apply X amount of stacks. |
| Aggregate by Target | There is only one instance of stacks on the Target regardless of Source. Each Source can apply a stack up to the shared stack limit. |

Stacks also have policies for expiration, duration refresh, and period reset. They have helpful hover tooltips in the `GameplayEffect` Blueprint.

The Sample Project includes a custom Blueprint node that listens for `GameplayEffect` stack changes. The HUD UMG Widget uses it to update the amount of passive armor stacks that the player has. This `AsyncTask` will live forever until manually called `EndTask()`, which we do in the UMG Widget's `Destruct` event. See `AsyncTaskEffectStackChanged.h/cpp`.

![Listen for GameplayEffect Stack Change BP Node](https://github.com/tranek/GASDocumentation/raw/master/Images/gestackchange.png)

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ge-ga"></a>
#### 4.5.6 Granted Abilities
`GameplayEffects` can grant new [`GameplayAbilities`](#concepts-ga) to `ASCs`. Only `Duration` and `Infinite` `GameplayEffects` can grant abilities.

A common usecase for this is when you want to force another player to do something like moving them from a knockback or pull. You would apply a `GameplayEffect` to them that grants them an automatically activating ability (see [Passive Abilities](#concepts-ga-activating-passive) for how to automatically activate an ability when it is granted) that does the desired action to them.

Designers can choose which abilities a `GameplayEffect` grants, what level to grant them at, what [input to bind](#concepts-ga-input) them at and the removal policy for the granted ability.

| Removal Policy             | Description                                                  |
| -------------------------- | ------------------------------------------------------------ |
| Cancel Ability Immediately | The granted ability is canceled and removed immediately when the `GameplayEffect` that granted it is removed from the Target. |
| Remove Ability on End      | The granted ability is allowed to finish and then is removed from the Target. |
| Do Nothing                 | The granted ability is not affected by the removal of the granting `GameplayEffect` from the Target. The Target has the ability permanently until it is manually removed later. |

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ge-tags"></a>
#### 4.5.7 Gameplay Effect Tags
`GameplayEffects` carry multiple [`GameplayTagContainers`](#concepts-gt). Designers will edit the `Added` and `Removed` `GameplayTagContainers` for each category and the result will show up in the `Combined` `GameplayTagContainer` on compilation. `Added` tags are new tags that this `GameplayEffect` adds that its parents did not previously have. `Removed` tags are tags that parent classes have but this subclass does not have.

| Category                          | Description                                                  |
| --------------------------------- | ------------------------------------------------------------ |
| Gameplay Effect Asset Tags        | Tags that the `GameplayEffect` has. They do not do any function on their own and serve only the purpose of describing the `GameplayEffect`. |
| Granted Tags                      | Tags that live on the `GameplayEffect` but are also given to the `ASC` that the `GameplayEffect` is applied to. They are removed from the `ASC` when the `GameplayEffect` is removed. This only works for `Duration` and `Infinite` `GameplayEffects`. |
| Ongoing Tag Requirements          | Once applied, these tags determine whether the `GameplayEffect` is on or off. A `GameplayEffect` can be off and still be applied. If a `GameplayEffect` is off due to failing the Ongoing Tag Requirements, but the requirements are then met, the `GameplayEffect` will turn on again and reapply its modifiers. This only works for `Duration` and `Infinite` `GameplayEffects`. |
| Application Tag Requirements      | Tags on the Target that determine if a `GameplayEffect` can be applied to the Target. If these requirements are not met, the `GameplayEffect` is not applied. |
| Remove Gameplay Effects with Tags | `GameplayEffects` on the Target that have any of these tags in their `Asset Tags` or `Granted Tags` will be removed from the Target when this `GameplayEffect` is successfully applied. |

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ge-immunity"></a>
#### 4.5.8 Immunity
`GameplayEffects` can grant immunity, effectively blocking the application of other `GameplayEffects`, based on [`GameplayTags`](#concepts-gt). While immunity can be effectively achieved through other means like `Application Tag Requirements`, using this system provides a delegate for when `GameplayEffects` are blocked due to immunity `UAbilitySystemComponent::OnImmunityBlockGameplayEffectDelegate`.

`GrantedApplicationImmunityTags` checks if the Source `ASC` (including tags from the Source ability's `AbilityTags` if there was one) has any of the specified tags. This is a way to provide immunity from all `GameplayEffects` from certain characters or sources based on their tags.

`Granted Application Immunity Query` checks the incoming `GameplayEffectSpec` if it matches any of the queries to block or allow its application.

The queries have helpful hover tooltips in the `GameplayEffect` Blueprint.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ge-spec"></a>
#### 4.5.9 Gameplay Effect Spec
The [`GameplayEffectSpec`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FGameplayEffectSpec/index.html) (`GESpec`) can be thought of as the instantiations of `GameplayEffects`. They hold a reference to the `GameplayEffect` class that they represent, what level it was created at, and who created it. These can be freely created and modified at runtime before application unlike `GameplayEffects` which should be created by designers prior to runtime. When applying a `GameplayEffect`, a `GameplayEffectSpec` is created from the `GameplayEffect` and that is actually what is applied to the Target.

`GameplayEffectSpecs` are created from `GameplayEffects` using `UAbilitySystemComponent::MakeOutgoingSpec()` which is `BlueprintCallable`. `GameplayEffectSpecs` do not have to be immediately applied. It is common to pass a `GameplayEffectSpec` to a projectile created from an ability that the projectile can apply to the target it hits later. When `GameplayEffectSpecs` are successfully applied, they return a new struct called `FActiveGameplayEffect`.

Notable `GameplayEffectSpec` Contents:
* The `GameplayEffect` class that this `GameplayEffect` was created from.
* The level of this `GameplayEffectSpec`. Usually the same as the level of the ability that created the `GameplayEffectSpec` but can be different.
* The duration of the `GameplayEffectSpec`. Defaults to the duration of the `GameplayEffect` but can be different.
* The period of the `GameplayEffectSpec` for periodic effects. Defaults to the period of the `GameplayEffect` but can be different.
* The current stack count of this `GameplayEffectSpec`. The stack limit is on the `GameplayEffect`.
* The [`GameplayEffectContextHandle`](#concepts-ge-context) tells us who created this `GameplayEffectSpec`.
* `Attributes` that were captured at the time of the `GameplayEffectSpec`'s creation due to snapshotting.
* `DynamicGrantedTags` that the `GameplayEffectSpec` grants to the Target in addition to the `GameplayTags` that the `GameplayEffect` grants.
* `DynamicAssetTags` that the `GameplayEffectSpec` has in addition to the `AssetTags` that the `GameplayEffect` has.
* `SetByCaller` `TMaps`.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ge-spec-setbycaller"></a>
##### 4.5.9.1 SetByCallers
`SetByCallers` allow the `GameplayEffectSpec` to carry float values associated with a `GameplayTag` or `FName` around. They are stored in their respective `TMaps`: `TMap<FGameplayTag, float>` and `TMap<FName, float>` on the `GameplayEffectSpec`. These can be used as `Modifiers` on the `GameplayEffect` or as generic means of ferrying floats around. It is common to pass numerical data generated inside of an ability to [`GameplayEffectExecutionCalculations`](#concepts-ge-ec) or [`ModifierMagnitudeCalculations`](#concepts-ge-mmc) via `SetByCallers`.

| `SetByCaller` Use | Notes                                                        |
| ----------------- | ------------------------------------------------------------ |
| `Modifiers`       | Must be defined ahead of time in the `GameplayEffect` class. Can only use the `GameplayTag` version. If one is defined on the `GameplayEffect` class but the `GameplayEffectSpec` does not have the corresponding tag and float value pair, the game will have a runtime error on application of the `GameplayEffectSpec` and return 0. This is a potential problem for a `Divide` operation. See [`Modifiers`](#concepts-ge-mods). |
| Elsewhere         | Does not need to be defined ahead of time anywhere. Reading a `SetByCaller` that does not exist on a `GameplayEffectSpec` can return a developer defined default value with optional warnings. |

To assign `SetByCaller` values in Blueprint, use the Blueprint node for the version that you need (`GameplayTag` or `FName`):

![Assigning SetByCaller](https://github.com/tranek/GASDocumentation/raw/master/Images/setbycaller.png)

To read a `SetByCaller` value in Blueprint, you will need to make custom nodes in your Blueprint Library.

To assign `SetByCaller` values in C++, use the version of the function that you need (`GameplayTag` or `FName`):

```c++
void FGameplayEffectSpec::SetSetByCallerMagnitude(FName DataName, float Magnitude);
```
```c++
void FGameplayEffectSpec::SetSetByCallerMagnitude(FGameplayTag DataTag, float Magnitude);
```

To read a `SetByCaller` value in C++, use the version of the function that you need (`GameplayTag` or `FName`):

```c++
float GetSetByCallerMagnitude(FName DataName, bool WarnIfNotFound = true, float DefaultIfNotFound = 0.f) const;
```
```c++
float GetSetByCallerMagnitude(FGameplayTag DataTag, bool WarnIfNotFound = true, float DefaultIfNotFound = 0.f) const;
```

I recommend using the `GameplayTag` version over the `FName` version. This can prevent spelling errors in Blueprint.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ge-context"></a>
#### 4.5.10 Gameplay Effect Context
The [`GameplayEffectContext`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FGameplayEffectContext/index.html) structure holds information about a `GameplayEffectSpec's` instigator and [`TargetData`](#concepts-targeting-data). This is also a good structure to subclass to pass arbitrary data around between places like [`ModifierMagnitudeCalculations`](#concepts-ge-mmc) / [`GameplayEffectExecutionCalculations`](#concepts-ge-ec), [`AttributeSets`](#concepts-as), and [`GameplayCues`](#concepts-gc).

To subclass the `GameplayEffectContext`:

1. Subclass `FGameplayEffectContext`
1. Override `FGameplayEffectContext::GetScriptStruct()`
1. Override `FGameplayEffectContext::Duplicate()`
1. Override `FGameplayEffectContext::NetSerialize()` if your new data needs to be replicated
1. Implement `TStructOpsTypeTraits` for your subclass, like the parent struct `FGameplayEffectContext` has
1. Override `AllocGameplayEffectContext()` in your [`AbilitySystemGlobals`](#concepts-asg) class to return a new object of your subclass

[GASShooter](https://github.com/tranek/GASShooter) uses a subclassed `GameplayEffectContext` to add `TargetData` which can be accessed in `GameplayCues`, specifically for the shotgun since it can hit more than one enemy.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ge-mmc"></a>
#### 4.5.11 Modifier Magnitude Calculation
[`ModifierMagnitudeCalculations`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayModMagnitudeCalculation/index.html) (`ModMagCalc` or `MMC`) are powerful classes used as [`Modifiers`](#concepts-ge-mods) in `GameplayEffects`. They function similarly to [`GameplayEffectExecutionCalculations`](#concepts-ge-ec) but are less powerful and most importantly they can be [predicted](#concepts-p). Their sole purpose is to return a float value from `CalculateBaseMagnitude_Implementation()`. You can subclass and override this function in Blueprint and C++.

`MMCs` can be used in any duration of `GameplayEffects` - `Instant`, `Duration`, `Infinite`, or `Periodic`.

`MMCs'` strength lies in their capability to capture the value of any number of `Attributes` on the `Source` or the `Target` of `GameplayEffect` with full access to the `GameplayEffectSpec` to read `GameplayTags` and `SetByCallers`. `Attributes` can either be snapshotted or not. Snapshotted `Attributes` are captured when the `GameplayEffectSpec` is created whereas non snapshotted `Attributes` are captured when the `GameplayEffectSpec` is applied and automatically update when the `Attribute` changes for `Infinite` and `Duration` `GameplayEffects`. Capturing `Attributes` recalculates their `CurrentValue` from existing mods on the `ASC`. This recalculation will **not** run [`PreAttributeChange()`](#concepts-as-preattributechange) in the `AbilitySet` so any clamping must be done here again.

| Snapshot | Source or Target | Captured on `GameplayEffectSpec` | Automatically updates when `Attribute` changes for `Infinite` or `Duration` `GE` |
| -------- | ---------------- | -------------------------------- | ------------------------------------------------------------ |
| Yes      | Source           | Creation                         | No                                                           |
| Yes      | Target           | Application                      | No                                                           |
| No       | Source           | Application                      | Yes                                                          |
| No       | Target           | Application                      | Yes                                                          |

The resultant float from an `MMC` can further be modified in the `GameplayEffect's` `Modifier` by a coefficient and a pre and post coefficient addition.

An example `MMC` that captures the `Target's` mana `Attribute` reduces it from a poison effect where the amount reduced changes depending on how much mana the `Target` has and a tag that the `Target` might have:
```c++
UPAMMC_PoisonMana::UPAMMC_PoisonMana()
{

	//ManaDef defined in header FGameplayEffectAttributeCaptureDefinition ManaDef;
	ManaDef.AttributeToCapture = UPAAttributeSetBase::GetManaAttribute();
	ManaDef.AttributeSource = EGameplayEffectAttributeCaptureSource::Target;
	ManaDef.bSnapshot = false;

	//MaxManaDef defined in header FGameplayEffectAttributeCaptureDefinition MaxManaDef;
	MaxManaDef.AttributeToCapture = UPAAttributeSetBase::GetMaxManaAttribute();
	MaxManaDef.AttributeSource = EGameplayEffectAttributeCaptureSource::Target;
	MaxManaDef.bSnapshot = false;

	RelevantAttributesToCapture.Add(ManaDef);
	RelevantAttributesToCapture.Add(MaxManaDef);
}

float UPAMMC_PoisonMana::CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec & Spec) const
{
	// Gather the tags from the source and target as that can affect which buffs should be used
	const FGameplayTagContainer* SourceTags = Spec.CapturedSourceTags.GetAggregatedTags();
	const FGameplayTagContainer* TargetTags = Spec.CapturedTargetTags.GetAggregatedTags();

	FAggregatorEvaluateParameters EvaluationParameters;
	EvaluationParameters.SourceTags = SourceTags;
	EvaluationParameters.TargetTags = TargetTags;

	float Mana = 0.f;
	GetCapturedAttributeMagnitude(ManaDef, Spec, EvaluationParameters, Mana);
	Mana = FMath::Max<float>(Mana, 0.0f);

	float MaxMana = 0.f;
	GetCapturedAttributeMagnitude(MaxManaDef, Spec, EvaluationParameters, MaxMana);
	MaxMana = FMath::Max<float>(MaxMana, 1.0f); // Avoid divide by zero

	float Reduction = -20.0f;
	if (Mana / MaxMana > 0.5f)
	{
		// Double the effect if the target has more than half their mana
		Reduction *= 2;
	}
	
	if (TargetTags->HasTagExact(FGameplayTag::RequestGameplayTag(FName("Status.WeakToPoisonMana"))))
	{
		// Double the effect if the target is weak to PoisonMana
		Reduction *= 2;
	}
	
	return Reduction;
}
```

If you don't add the `FGameplayEffectAttributeCaptureDefinition` to `RelevantAttributesToCapture` in the `MMC's` constructor and try to capture `Attributes`, you will get an error about a missing Spec while capturing. If you don't need to capture `Attributes`, then you don't have to add anything to `RelevantAttributesToCapture`.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ge-ec"></a>
#### 4.5.12 Gameplay Effect Execution Calculation
[`GameplayEffectExecutionCalculations`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayEffectExecutionCalculat-/index.html) (`ExecutionCalculation`, `Execution` (you will often see this term in the plugin's source code), or `ExecCalc`) are the most powerful way for `GameplayEffects` to make changes to an `ASC`. Like [`ModifierMagnitudeCalculations`](#concepts-ge-mmc), these can capture `Attributes` and optionally snapshot them. Unlike `MMCs`, these can change more than one `Attribute` and essentially do anything else that the programmer wants. The downside to this power and flexibility is that they can not be [predicted](#concepts-p) and they must be implemented in C++.

`ExecutionCalculations` can only be used with `Instant` and `Periodic` `GameplayEffects`. Anything with the word 'Execute' in it typically refers to these two types of `GameplayEffects`.

Snapshotting captures the `Attribute` when the `GameplayEffectSpec` is created whereas not snapshotting captures the `Attribute` when the `GameplayEffectSpec` is applied. Capturing `Attributes` recalculates their `CurrentValue` from existing mods on the `ASC`. This recalculation will **not** run [`PreAttributeChange()`](#concepts-as-preattributechange) in the `AbilitySet` so any clamping must be done here again.

| Snapshot | Source or Target | Captured on `GameplayEffectSpec` |
| -------- | ---------------- | -------------------------------- |
| Yes      | Source           | Creation                         |
| Yes      | Target           | Application                      |
| No       | Source           | Application                      |
| No       | Target           | Application                      |

To set up `Attribute` capture, we follow a pattern set by Epic's ActionRPG Sample Project by defining a struct holding and defining how we capture the `Attributes` and creating one copy of it in the struct's constructor. You will have a struct like this for every `ExecCalc`. **Note:** Each struct needs a unique name as they share the same namespace. Using the same name for the structs will cause incorrect behavior in capturing your `Attributes` (mostly capturing the values of the wrong `Attributes`).

For `Local Predicted`, `Server Only`, and `Server Initiated` [`GameplayAbilities`](#concepts-ga), the `ExecCalc` only calls on the Server.

Calculating damage received based on a complex formula reading from many attributes on the `Source` and the `Target` is the most common example of an `ExecCalc`. The included Sample Project has a simple `ExecCalc` for calculating damage that reads the value of damage from the `GameplayEffectSpec's` [`SetByCaller`](#concepts-ge-spec-setbycaller) and then mitigates that value based on the armor `Attribute` captured from the `Target`. See `GDDamageExecCalculation.cpp/.h`.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ge-ec-senddata"></a>
##### 4.5.12.1 Sending Data to Execution Calculations
There are a few ways to send data to an `ExecutionCalculation` in addition to capturing `Attributes`.

<a name="concepts-ge-ec-senddata-setbycaller"></a>
###### 4.5.12.1.1 SetByCaller
Any [`SetByCallers` set on the `GameplayEffectSpec`](#concepts-ge-spec-setbycaller) can be directly read in the `ExecutionCalculation`.

```c++
const FGameplayEffectSpec& Spec = ExecutionParams.GetOwningSpec();
float Damage = FMath::Max<float>(Spec.GetSetByCallerMagnitude(FGameplayTag::RequestGameplayTag(FName("Data.Damage")), false, -1.0f), 0.0f);
```

<a name="concepts-ge-ec-senddata-backingdataattribute"></a>
###### 4.5.12.1.2 Backing Data Attribute Calculation Modifier
If you want to hardcode values to a `GameplayEffect`, you can pass them in using a `CalculationModifier` that uses one of the captured `Attributes` as the backing data.

In this screenshot example, we're adding 50 to the captured Damage `Attribute`. You could also set this to `Override` to just take in only the hardcoded value.

![Backing Data Attribute Calculation Modifier](https://github.com/tranek/GASDocumentation/raw/master/Images/calculationmodifierbackingdataattribute.png)

The `ExecutionCalculation` reads this value in when it captures the `Attribute`.

```c++
float Damage = 0.0f;
// Capture optional damage value set on the damage GE as a CalculationModifier under the ExecutionCalculation
ExecutionParams.AttemptCalculateCapturedAttributeMagnitude(DamageStatics().DamageDef, EvaluationParameters, Damage);
```

<a name="concepts-ge-ec-senddata-backingdatatempvariable"></a>
###### 4.5.12.1.3 Backing Data Temporary Variable Calculation Modifier
If you want to hardcode values to a `GameplayEffect`, you can pass them in using a `CalculationModifier` that uses a `Temporary Variable` or `Transient Aggregator` as it's called in C++. The `Temporary Variable` is associated with a `GameplayTag`.

In this screenshot example, we're adding 50 to a `Temporary Variable` using the `Data.Damage` `GameplayTag`.

![Backing Data Temporary Variable Calculation Modifier](https://github.com/tranek/GASDocumentation/raw/master/Images/calculationmodifierbackingdatatempvariable.png)

Add backing `Temporary Variables` to your `ExecutionCalculation`'s constructor:

```c++
ValidTransientAggregatorIdentifiers.AddTag(FGameplayTag::RequestGameplayTag("Data.Damage"));
```

The `ExecutionCalculation` reads this value in using special capture functions similar to the `Attribute` capture functions.

```c++
float Damage = 0.0f;
ExecutionParams.AttemptCalculateTransientAggregatorMagnitude(FGameplayTag::RequestGameplayTag("Data.Damage"), EvaluationParameters, Damage);
```

<a name="concepts-ge-ec-senddata-effectcontext"></a>
###### 4.5.12.1.4 Gameplay Effect Context
You can send data to the `ExecutionCalculation` via a custom [`GameplayEffectContext` on the `GameplayEffectSpec`](#concepts-ge-context).

In the `ExecutionCalculation` you can access the `EffectContext` from the `FGameplayEffectCustomExecutionParameters`.

```c++
const FGameplayEffectSpec& Spec = ExecutionParams.GetOwningSpec();
FGSGameplayEffectContext* ContextHandle = static_cast<FGSGameplayEffectContext*>(Spec.GetContext().Get());
```

If you need change something on the `GameplayEffectSpec` or the `EffectContext`:

```c++
FGameplayEffectSpec* MutableSpec = ExecutionParams.GetOwningSpecForPreExecuteMod();
FGSGameplayEffectContext* ContextHandle = static_cast<FGSGameplayEffectContext*>(MutableSpec->GetContext().Get());
```

Use caution if modifying the `GameplayEffectSpec` in the `ExecutionCalculation`. See the comment for `GetOwningSpecForPreExecuteMod()`.

```c++
/** Non const access. Be careful with this, especially when modifying a spec after attribute capture. */
FGameplayEffectSpec* GetOwningSpecForPreExecuteMod() const;
```

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ge-car"></a>
#### 4.5.13 Custom Application Requirement
[`CustomApplicationRequirement`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayEffectCustomApplication-/index.html) (`CAR`) classes give the designers advanced control over whether a `GameplayEffect` can be applied versus the simple `GameplayTag` checks on the `GameplayEffect`. These can be implemented in Blueprint by overriding `CanApplyGameplayEffect()` and in C++ by overriding `CanApplyGameplayEffect_Implementation()`.

Examples of when to use `CARs`:
* `Target` needs to have a certain amount of an `Attribute`
* `Target` needs to have a certain number of stacks of a `GameplayEffect`

`CARs` can also do more advanced things like checking if an instance of this `GameplayEffect` is already on the `Target` and [changing the duration](#concepts-ge-duration) of the existing instance instead of applying a new instance (return false for `CanApplyGameplayEffect()`).

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ge-cost"></a>
#### 4.5.14 Cost Gameplay Effect
[`GameplayAbilities`](#concepts-ga) have an optional `GameplayEffect` specifically designed to use as the cost of the ability. Costs are how much of an `Attribute` an `ASC` needs to have to be able to activate the `GameplayAbility`. If a `GA` cannot afford the `Cost GE`, then they will not be able to activate. This `Cost GE` should be an `Instant` `GameplayEffect` with one or more `Modifiers` that subtract from `Attributes`. By default, `Cost GEs` are meant to be predicted and it is recommended to maintain that capability meaning do not use `ExecutionCalculations`. `MMCs` are perfectly acceptable and encouraged for complex cost calculations.

When starting out, you will most likely have one unique `Cost GE` per `GA` that has a cost. A more advanced technique is to reuse one `Cost GE` for multiple `GAs` and just modify the `GameplayEffectSpec` created from the `Cost GE` with the `GA`-specific data (the cost value is defined on the `GA`). **This only works for `Instanced` abilities.**

Two techniques for reusing the `Cost GE`:

1. **Use an `MMC`.** This is the easiest method. Create an [`MMC`](#concepts-ge-mmc) that reads the cost value from the `GameplayAbility` instance which you can get from the `GameplayEffectSpec`.

```c++
float UPGMMC_HeroAbilityCost::CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec & Spec) const
{
	const UPGGameplayAbility* Ability = Cast<UPGGameplayAbility>(Spec.GetContext().GetAbilityInstance_NotReplicated());

	if (!Ability)
	{
		return 0.0f;
	}

	return Ability->Cost.GetValueAtLevel(Ability->GetAbilityLevel());
}
```

In this example the cost value is an `FScalableFloat` on the `GameplayAbility` child class that I added to it.
```c++
UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cost")
FScalableFloat Cost;
```

![Cost GE With MMC](https://github.com/tranek/GASDocumentation/raw/master/Images/costmmc.png)

2. **Override `UGameplayAbility::GetCostGameplayEffect()`.** Override this function and [create a `GameplayEffect` at runtime](#concepts-ge-dynamic) that reads the cost value on the `GameplayAbility`.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ge-cooldown"></a>
#### 4.5.15 Cooldown Gameplay Effect
[`GameplayAbilities`](#concepts-ga) have an optional `GameplayEffect` specifically designed to use as the cooldown of the ability. Cooldowns determine how long after activation the ability can be activated again. If a `GA` is still on cooldown, it cannot activate. This `Cooldown GE` should be a `Duration` `GameplayEffect` with no `Modifiers` and a unique `GameplayTag` per `GameplayAbility` or per ability slot (if your game has interchangeable abilities assigned to slots that share a cooldown) in the `GameplayEffect's` `GrantedTags` ("`Cooldown Tag`"). The `GA` actually checks for the presence of the `Cooldown Tag` instead of the presence of the `Cooldown GE`. By default, `Cooldown GEs` are meant to be predicted and it is recommended to maintain that capability meaning do not use `ExecutionCalculations`. `MMCs` are perfectly acceptable and encouraged for complex cooldown calculations.

When starting out, you will most likely have one unique `Cooldown GE` per `GA` that has a cooldown. A more advanced technique is to reuse one `Cooldown GE` for multiple `GAs` and just modify the `GameplayEffectSpec` created from the `Cooldown GE` with the `GA`-specific data (the cooldown duration and the `Cooldown Tag` are defined on the `GA`). **This only works for `Instanced` abilities.**

Two techniques for reusing the `Cooldown GE`:

1. **Use a [`SetByCaller`](#concepts-ge-spec-setbycaller).** This is the easiest method. Set the duration of your shared `Cooldown GE` to `SetByCaller` with a `GameplayTag`. On your `GameplayAbility` subclass, define a float / `FScalableFloat` for the duration, a `FGameplayTagContainer` for the unique `Cooldown Tag`, and a temporary `FGameplayTagContainer` that we will use as the return pointer of the union of our `Cooldown Tag` and the `Cooldown GE's` tags.
```c++
UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FScalableFloat CooldownDuration;

UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FGameplayTagContainer CooldownTags;

// Temp container that we will return the pointer to in GetCooldownTags().
// This will be a union of our CooldownTags and the Cooldown GE's cooldown tags.
UPROPERTY(Transient)
FGameplayTagContainer TempCooldownTags;
```

Then override `UGameplayAbility::GetCooldownTags()` to return the union of our `Cooldown Tags` and any existing `Cooldown GE's` tags.
```c++
const FGameplayTagContainer * UPGGameplayAbility::GetCooldownTags() const
{
	FGameplayTagContainer* MutableTags = const_cast<FGameplayTagContainer*>(&TempCooldownTags);
	MutableTags->Reset(); // MutableTags writes to the TempCooldownTags on the CDO so clear it in case the ability cooldown tags change (moved to a different slot)
	const FGameplayTagContainer* ParentTags = Super::GetCooldownTags();
	if (ParentTags)
	{
		MutableTags->AppendTags(*ParentTags);
	}
	MutableTags->AppendTags(CooldownTags);
	return MutableTags;
}
```

Finally, override `UGameplayAbility::ApplyCooldown()` to inject our `Cooldown Tags` and to add the `SetByCaller` to the cooldown `GameplayEffectSpec`.
```c++
void UPGGameplayAbility::ApplyCooldown(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo * ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo) const
{
	UGameplayEffect* CooldownGE = GetCooldownGameplayEffect();
	if (CooldownGE)
	{
		FGameplayEffectSpecHandle SpecHandle = MakeOutgoingGameplayEffectSpec(CooldownGE->GetClass(), GetAbilityLevel());
		SpecHandle.Data.Get()->DynamicGrantedTags.AppendTags(CooldownTags);
		SpecHandle.Data.Get()->SetSetByCallerMagnitude(FGameplayTag::RequestGameplayTag(FName(  OurSetByCallerTag  )), CooldownDuration.GetValueAtLevel(GetAbilityLevel()));
		ApplyGameplayEffectSpecToOwner(Handle, ActorInfo, ActivationInfo, SpecHandle);
	}
}
```

In this picture, the cooldown's duration `Modifier` is set to `SetByCaller` with a `Data Tag` of `Data.Cooldown`. `Data.Cooldown` would be `OurSetByCallerTag` in the code above.

![Cooldown GE with SetByCaller](https://github.com/tranek/GASDocumentation/raw/master/Images/cooldownsbc.png)

2. **Use an [`MMC`](#concepts-ge-mmc).** This has the same setup as above except for setting the `SetByCaller` as the duration on the `Cooldown GE` and in `ApplyCooldown`. Instead, set the duration to be a `Custom Calculation Class` and point to the new `MMC` that we will make.
```c++
UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FScalableFloat CooldownDuration;

UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FGameplayTagContainer CooldownTags;

// Temp container that we will return the pointer to in GetCooldownTags().
// This will be a union of our CooldownTags and the Cooldown GE's cooldown tags.
UPROPERTY(Transient)
FGameplayTagContainer TempCooldownTags;
```

Then override `UGameplayAbility::GetCooldownTags()` to return the union of our `Cooldown Tags` and any existing `Cooldown GE's` tags.
```c++
const FGameplayTagContainer * UPGGameplayAbility::GetCooldownTags() const
{
	FGameplayTagContainer* MutableTags = const_cast<FGameplayTagContainer*>(&TempCooldownTags);
	MutableTags->Reset(); // MutableTags writes to the TempCooldownTags on the CDO so clear it in case the ability cooldown tags change (moved to a different slot)
	const FGameplayTagContainer* ParentTags = Super::GetCooldownTags();
	if (ParentTags)
	{
		MutableTags->AppendTags(*ParentTags);
	}
	MutableTags->AppendTags(CooldownTags);
	return MutableTags;
}
```

Finally, override `UGameplayAbility::ApplyCooldown()` to inject our `Cooldown Tags` into the cooldown `GameplayEffectSpec`.
```c++
void UPGGameplayAbility::ApplyCooldown(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo * ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo) const
{
	UGameplayEffect* CooldownGE = GetCooldownGameplayEffect();
	if (CooldownGE)
	{
		FGameplayEffectSpecHandle SpecHandle = MakeOutgoingGameplayEffectSpec(CooldownGE->GetClass(), GetAbilityLevel());
		SpecHandle.Data.Get()->DynamicGrantedTags.AppendTags(CooldownTags);
		ApplyGameplayEffectSpecToOwner(Handle, ActorInfo, ActivationInfo, SpecHandle);
	}
}
```

```c++
float UPGMMC_HeroAbilityCooldown::CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec & Spec) const
{
	const UPGGameplayAbility* Ability = Cast<UPGGameplayAbility>(Spec.GetContext().GetAbilityInstance_NotReplicated());

	if (!Ability)
	{
		return 0.0f;
	}

	return Ability->CooldownDuration.GetValueAtLevel(Ability->GetAbilityLevel());
}
```

![Cooldown GE with MMC](https://github.com/tranek/GASDocumentation/raw/master/Images/cooldownmmc.png)

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ge-cooldown-tr"></a>
##### 4.5.15.1 Get the Cooldown Gameplay Effect's Remaining Time
```c++
bool APGPlayerState::GetCooldownRemainingForTag(FGameplayTagContainer CooldownTags, float & TimeRemaining, float & CooldownDuration)
{
	if (AbilitySystemComponent && CooldownTags.Num() > 0)
	{
		TimeRemaining = 0.f;
		CooldownDuration = 0.f;

		FGameplayEffectQuery const Query = FGameplayEffectQuery::MakeQuery_MatchAnyOwningTags(CooldownTags);
		TArray< TPair<float, float> > DurationAndTimeRemaining = AbilitySystemComponent->GetActiveEffectsTimeRemainingAndDuration(Query);
		if (DurationAndTimeRemaining.Num() > 0)
		{
			int32 BestIdx = 0;
			float LongestTime = DurationAndTimeRemaining[0].Key;
			for (int32 Idx = 1; Idx < DurationAndTimeRemaining.Num(); ++Idx)
			{
				if (DurationAndTimeRemaining[Idx].Key > LongestTime)
				{
					LongestTime = DurationAndTimeRemaining[Idx].Key;
					BestIdx = Idx;
				}
			}

			TimeRemaining = DurationAndTimeRemaining[BestIdx].Key;
			CooldownDuration = DurationAndTimeRemaining[BestIdx].Value;

			return true;
		}
	}

	return false;
}
```

**Note:** Querying the cooldown's time remaining on clients requires that they can receive replicated `GameplayEffects`. This will depend on their `ASC's` [replication mode](#concepts-asc-rm).

<a name="concepts-ge-cooldown-listen"></a>
##### 4.5.15.2 Listening for Cooldown Begin and End
To listen for when a cooldown begins, you can either respond to when the `Cooldown GE` is applied by binding to `AbilitySystemComponent->OnActiveGameplayEffectAddedDelegateToSelf` or when the `Cooldown Tag` is added by binding to `AbilitySystemComponent->RegisterGameplayTagEvent(CooldownTag, EGameplayTagEventType::NewOrRemoved)`. I recommend listening for when the `Cooldown GE` is added because you also have access to the `GameplayEffectSpec` that applied it. From this you can determine if the `Cooldown GE` is the locally predicted one or the Server's correcting one.

To listen for when a cooldown ends, you can either respond to when the `Cooldown GE` is removed by binding to `AbilitySystemComponent->OnAnyGameplayEffectRemovedDelegate()` or when the `Cooldown Tag` is removed by binding to `AbilitySystemComponent->RegisterGameplayTagEvent(CooldownTag, EGameplayTagEventType::NewOrRemoved)`. I recommend listening for when the `Cooldown Tag` is removed because when the Server's corrected `Cooldown GE` comes in, it will remove our locally predicted one causing the `OnAnyGameplayEffectRemovedDelegate()` to fire even though we're still on cooldown. The `Cooldown Tag` will not change during the removal of the predicted `Cooldown GE` and the application of the Server's corrected `Cooldown GE`.

**Note:** Listening for a `GameplayEffect` to be added or removed on clients requires that they can receive replicated `GameplayEffects`. This will depend on their `ASC's` [replication mode](#concepts-asc-rm).

The Sample Project includes a custom Blueprint node that listens for cooldowns beginning and ending. The HUD UMG Widget uses it to update the amount of time remaining on the Meteor's cooldown. This `AsyncTask` will live forever until manually called `EndTask()`, which we do in the UMG Widget's `Destruct` event. See [`AsyncTaskCooldownChanged.h/cpp`](Source/GASDocumentation/Private/Characters/Abilities/AsyncTaskCooldownChanged.cpp).

![Listen for Cooldown Change BP Node](https://github.com/tranek/GASDocumentation/raw/master/Images/cooldownchange.png)

<a name="concepts-ge-cooldown-prediction"></a>
##### 4.5.15.3 Predicting Cooldowns
Cooldowns cannot really be predicted currently. We can start UI cooldown timer's when the locally predicted `Cooldown GE` is applied but the `GameplayAbility's` actual cooldown is tied to the server's cooldown's time remaining. Depending on the player's latency, the locally predicted cooldown could expire but the `GameplayAbility` would still be on cooldown on the server and this would prevent the `GameplayAbility's` immediate re-activation until the server's cooldown expires.

The Sample Project handles this by graying out the Meteor ability's UI icon when the locally predicted cooldown begins and then starting the cooldown timer once the server's corrected `Cooldown GE` comes in.

A gameplay consequence of this is that players with high latencies have a lower rate of fire on short cooldown abilities than players with lower latencies putting them at a disadvantage. Fortnite avoids this by their weapons having custom bookkeeping that do not use cooldown `GameplayEffects`.

Allowing for true predicted cooldowns (player could activate a `GameplayAbility` when the local cooldown expires but the server is still on cooldown) is something that Epic would like to implement someday in a [future iteration of GAS](#concepts-p-future).

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ge-duration"></a>
#### 4.5.16 Changing Active Gameplay Effect Duration
To change the time remaining for a `Cooldown GE` or any `Duration` `GameplayEffect`, we need to change the `GameplayEffectSpec's` `Duration`, update its `StartServerWorldTime`, update its `CachedStartServerWorldTime`, update its `StartWorldTime`, and rerun the check on the duration with `CheckDuration()`. Doing this on the server and marking the `FActiveGameplayEffect` dirty will replicate the changes to clients.
**Note:** This does involve a `const_cast` and may not be Epic's intended way of changing durations, but it seems to work well so far.

```c++
bool UPAAbilitySystemComponent::SetGameplayEffectDurationHandle(FActiveGameplayEffectHandle Handle, float NewDuration)
{
	if (!Handle.IsValid())
	{
		return false;
	}

	const FActiveGameplayEffect* ActiveGameplayEffect = GetActiveGameplayEffect(Handle);
	if (!ActiveGameplayEffect)
	{
		return false;
	}

	FActiveGameplayEffect* AGE = const_cast<FActiveGameplayEffect*>(ActiveGameplayEffect);
	if (NewDuration > 0)
	{
		AGE->Spec.Duration = NewDuration;
	}
	else
	{
		AGE->Spec.Duration = 0.01f;
	}

	AGE->StartServerWorldTime = ActiveGameplayEffects.GetServerWorldTime();
	AGE->CachedStartServerWorldTime = AGE->StartServerWorldTime;
	AGE->StartWorldTime = ActiveGameplayEffects.GetWorldTime();
	ActiveGameplayEffects.MarkItemDirty(*AGE);
	ActiveGameplayEffects.CheckDuration(Handle);

	AGE->EventSet.OnTimeChanged.Broadcast(AGE->Handle, AGE->StartWorldTime, AGE->GetDuration());
	OnGameplayEffectDurationChange(*AGE);

	return true;
}
```

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ge-dynamic"></a>
#### 4.5.17 Creating Dynamic Gameplay Effects at Runtime
Creating Dynamic `GameplayEffects` at runtime is an advanced topic. You shouldn't have to do this too often.

Only `Instant` `GameplayEffects` can be created at runtime from scratch in C++. `Duration` and `Infinite` `GameplayEffects` cannot be created dynamically at runtime because when they replicate they look for the `GameplayEffect` class definition that does not exist. To achieve this functionality, you should instead make an archetype `GameplayEffect` class like you would normally do in the Editor. Then customize the `GameplayEffectSpec` instance with what you need at runtime.

`Instant` `GameplayEffects` created at runtime can also be called from within a [local predicted](#concepts-p) `GameplayAbility`. However, it is unknown yet if the dynamic creation can have side effects.

##### Examples

The Sample Project creates one to send the gold and experience points back to the killer of a character when it takes the killing blow in its `AttributeSet`.

```c++
// Create a dynamic instant Gameplay Effect to give the bounties
UGameplayEffect* GEBounty = NewObject<UGameplayEffect>(GetTransientPackage(), FName(TEXT("Bounty")));
GEBounty->DurationPolicy = EGameplayEffectDurationType::Instant;

int32 Idx = GEBounty->Modifiers.Num();
GEBounty->Modifiers.SetNum(Idx + 2);

FGameplayModifierInfo& InfoXP = GEBounty->Modifiers[Idx];
InfoXP.ModifierMagnitude = FScalableFloat(GetXPBounty());
InfoXP.ModifierOp = EGameplayModOp::Additive;
InfoXP.Attribute = UGDAttributeSetBase::GetXPAttribute();

FGameplayModifierInfo& InfoGold = GEBounty->Modifiers[Idx + 1];
InfoGold.ModifierMagnitude = FScalableFloat(GetGoldBounty());
InfoGold.ModifierOp = EGameplayModOp::Additive;
InfoGold.Attribute = UGDAttributeSetBase::GetGoldAttribute();

Source->ApplyGameplayEffectToSelf(GEBounty, 1.0f, Source->MakeEffectContext());
```

A second example shows a runtime `GameplayEffect` created within a local predicted `GameplayAbility`. Use at your own risk (see comments in code)!

```c++
UGameplayAbilityRuntimeGE::UGameplayAbilityRuntimeGE()
{
	NetExecutionPolicy = EGameplayAbilityNetExecutionPolicy::LocalPredicted;
}

void UGameplayAbilityRuntimeGE::ActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, const FGameplayEventData* TriggerEventData)
{
	if (HasAuthorityOrPredictionKey(ActorInfo, &ActivationInfo))
	{
		if (!CommitAbility(Handle, ActorInfo, ActivationInfo))
		{
			EndAbility(Handle, ActorInfo, ActivationInfo, true, true);
		}

		// Create the GE at runtime.
		UGameplayEffect* GameplayEffect = NewObject<UGameplayEffect>(GetTransientPackage(), TEXT("RuntimeInstantGE"));
		GameplayEffect->DurationPolicy = EGameplayEffectDurationType::Instant; // Only instant works with runtime GE.

		// Add a simple scalable float modifier, which overrides MyAttribute with 42.
		// In real world applications, consume information passed via TriggerEventData.
		const int32 Idx = GameplayEffect->Modifiers.Num();
		GameplayEffect->Modifiers.SetNum(Idx + 1);
		FGameplayModifierInfo& ModifierInfo = GameplayEffect->Modifiers[Idx];
		ModifierInfo.Attribute.SetUProperty(UMyAttributeSet::GetMyModifiedAttribute());
		ModifierInfo.ModifierMagnitude = FScalableFloat(42.f);
		ModifierInfo.ModifierOp = EGameplayModOp::Override;

		// Apply the GE.

		// Create the GESpec here to avoid the behavior of ASC to create GESpecs from the GE class default object.
		// Since we have a dynamic GE here, this would create a GESpec with the base GameplayEffect class, so we
		// would lose our modifiers. Attention: It is unknown, if this "hack" done here can have drawbacks!
		// The spec prevents the GE object being collected by the GarbageCollector, since the GE is a UPROPERTY on the spec.
		FGameplayEffectSpec* GESpec = new FGameplayEffectSpec(GameplayEffect, {}, 0.f); // "new", since lifetime is managed by a shared ptr within the handle
		ApplyGameplayEffectSpecToOwner(Handle, ActorInfo, ActivationInfo, FGameplayEffectSpecHandle(GESpec));
	}
	EndAbility(Handle, ActorInfo, ActivationInfo, false, false);
}
```

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ge-containers"></a>
#### 4.5.18 Gameplay Effect Containers
Epic's [Action RPG Sample Project](https://www.unrealengine.com/marketplace/en-US/product/action-rpg) implements a structure called `FGameplayEffectContainer`. These are not in vanilla GAS but are extremely handy for containing `GameplayEffects` and [`TargetData`](#concepts-targeting-data). It automates some of the effort like creating `GameplayEffectSpecs` from `GameplayEffects` and setting default values in its `GameplayEffectContext`. Making a `GameplayEffectContainer` in a `GameplayAbility` and passing it to spawned projectiles is very easy and straightforward. I opted not to implement the `GameplayEffectContainers` in the included Sample Project to show how you would work without them in vanilla GAS, but I highly recommend looking into them and considering adding them to your project.

To access the `GESpecs` inside of the `GameplayEffectContainers` to do things like adding `SetByCallers`, break the `FGameplayEffectContainer` and access the `GESpec` reference by its index in the array of `GESpecs`. This requires that you know the index ahead of time of the `GESpec` that you want to access.

![SetByCaller with a GameplayEffectContainer](https://github.com/tranek/GASDocumentation/raw/master/Images/gecontainersetbycaller.png)

`GameplayEffectContainers` also contain an optional efficient means of [targeting](#concepts-targeting-containers).

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ga"></a>
### 4.6 Gameplay Abilities

<a name="concepts-ga-definition"></a>
#### 4.6.1 Gameplay Ability Definition
[`GameplayAbilities`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/UGameplayAbility/index.html) (`GA`) are any actions or skills that an `Actor` can do in the game. More than one `GameplayAbility` can be active at one time for example sprinting and shooting a gun. These can be made in Blueprint or C++.

Examples of `GameplayAbilities`:
* Jumping
* Sprinting
* Shooting a gun
* Passively blocking an attack every X number of seconds
* Using a potion
* Opening a door
* Collecting a resource
* Constructing a building

Things that should not be implemented with `GameplayAbilities`:
* Basic movement input
* Some interactions with UIs - Don't use a `GameplayAbility` to purchase an item from a store.

These are not rules, just my recommendations. Your design and implementations may vary.

`GameplayAbilities` come with default functionality to have a level to modify the amount of change to attributes or to change the `GameplayAbility's` functionality.

`GameplayAbilities` run on the owning client and/or the server depending on the [`Net Execution Policy`](#concepts-ga-net) but not simulated proxies. The `Net Execution Policy` determines if a `GameplayAbility` will be locally [predicted](#concepts-p). They include default behavior for [optional cost and cooldown `GameplayEffects`](#concepts-ga-commit). `GameplayAbilities` use [`AbilityTasks`](#concepts-at) for actions that happen over time like waiting for an event, waiting for an attribute change, waiting for players to choose a target, or moving a `Character` with `Root Motion Source`. **Simulated clients will not run `GameplayAbilities`**. Instead, when the server runs the ability, anything that visually needs to play on the simulated proxies (like animation montages) will be replicated or RPC'd through `AbilityTasks` or [`GameplayCues`](#concepts-gc) for cosmetic things like sounds and particles.

All `GameplayAbilities` will have their `ActivateAbility()` function overriden with your gameplay logic. Additional logic can be added to `EndAbility()` that runs when the `GameplayAbility` completes or is canceled.

Flowchart of a simple `GameplayAbility`:
![Simple GameplayAbility Flowchart](https://github.com/tranek/GASDocumentation/raw/master/Images/abilityflowchartsimple.png)


Flowchart of a more complex `GameplayAbility`:
![Complex GameplayAbility Flowchart](https://github.com/tranek/GASDocumentation/raw/master/Images/abilityflowchartcomplex.png)

Complex abilities can be implemented using multiple `GameplayAbilities` that interact (activate, cancel, etc) with each other.

<a name="concepts-ga-definition-reppolicy"></a>
##### 4.6.1.1 Replication Policy
Don't use this option. The name is misleading and you don't need it. [`GameplayAbilitySpecs`](#concepts-ga-spec) are replicated from the server to the owning client by default. As mentioned above, **`GameplayAbilities` don't run on simulated proxies**. They use `AbilityTasks` and `GameplayCues` to replicate or RPC visual changes to the simulated proxies. Dave Ratti from Epic has stated his desire to [remove this option in the future](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89).

<a name="concepts-ga-definition-remotecancel"></a>
##### 4.6.1.2 Server Respects Remote Ability Cancellation
This option causes trouble more often than not. It means if the client's `GameplayAbility` ends either due to cancellation or natural completion, it will force the server's version to end whether it completed or not. The latter issue is the important one, especially for locally predicted `GameplayAbilities` used by players with high latencies. Generally you will want to disable this option.

<a name="concepts-ga-definition-repinputdirectly"></a>
##### 4.6.1.3 Replicate Input Directly
Setting this option will always replicate input press and release events to the server. Epic recommends not using this and instead relying on the `Generic Replicated Events` that are built into the existing input related [`AbilityTasks`](#concepts-at) if you have your [input bound to your `ASC`](#concepts-ga-input).

Epic's comment:
```c++
/** Direct Input state replication. These will be called if bReplicateInputDirectly is true on the ability and is generally not a good thing to use. (Instead, prefer to use Generic Replicated Events). */
UAbilitySystemComponent::ServerSetInputPressed()
```

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ga-input"></a>
#### 4.6.2 Binding Input to the ASC
The `ASC` allows you to directly bind input actions to it and assign those inputs to `GameplayAbilities` when you grant them. Input actions assigned to `GameplayAbilities` automatically activate those `GameplayAbilities` when pressed if the `GameplayTag` requirements are met. Assigned input actions are required to use the built-in `AbilityTasks` that respond to input.

In addition to input actions assigned to activate `GameplayAbilities`, the `ASC` also accepts generic `Confirm` and `Cancel` inputs. These special inputs are used by `AbilityTasks` for confirming things like [`Target Actors`](#concepts-targeting-actors) or canceling them.

To bind input to an `ASC`, you must first create an enum that translates the input action name to a byte. The enum name must match exactly to the name used for the input action in the project settings. The `DisplayName` does not matter.

From the Sample Project:
```c++
UENUM(BlueprintType)
enum class EGDAbilityInputID : uint8
{
	// 0 None
	None			UMETA(DisplayName = "None"),
	// 1 Confirm
	Confirm			UMETA(DisplayName = "Confirm"),
	// 2 Cancel
	Cancel			UMETA(DisplayName = "Cancel"),
	// 3 LMB
	Ability1		UMETA(DisplayName = "Ability1"),
	// 4 RMB
	Ability2		UMETA(DisplayName = "Ability2"),
	// 5 Q
	Ability3		UMETA(DisplayName = "Ability3"),
	// 6 E
	Ability4		UMETA(DisplayName = "Ability4"),
	// 7 R
	Ability5		UMETA(DisplayName = "Ability5"),
	// 8 Sprint
	Sprint			UMETA(DisplayName = "Sprint"),
	// 9 Jump
	Jump			UMETA(DisplayName = "Jump")
};
```

If your `ASC` lives on the `Character`, then in `SetupPlayerInputComponent()` include the function for binding to the `ASC`:
```c++
// Bind to AbilitySystemComponent
FTopLevelAssetPath AbilityEnumAssetPath = FTopLevelAssetPath(FName("/Script/GASDocumentation"), FName("EGDAbilityInputID"));
AbilitySystemComponent->BindAbilityActivationToInputComponent(PlayerInputComponent, FGameplayAbilityInputBinds(FString("ConfirmTarget"),
	FString("CancelTarget"), AbilityEnumAssetPath, static_cast<int32>(EGDAbilityInputID::Confirm), static_cast<int32>(EGDAbilityInputID::Cancel)));
```

If your `ASC` lives on the `PlayerState`, there is a potential race condition inside of `SetupPlayerInputComponent()` where the `PlayerState` may not have replicated to the client yet. Therefore, I recommend attempting to bind to input in `SetupPlayerInputComponent()` and `OnRep_PlayerState()`. `OnRep_PlayerState()` is not sufficient by itself because there could be a case where the `Actor's` `InputComponent` could be null when `PlayerState` replicates before the `PlayerController` tells the client to call `ClientRestart()` which creates the `InputComponent`. The Sample Project demonstrates attempting to bind in both locations with a boolean gating the process so it only actually binds the input once.

**Note:** In the Sample Project `Confirm` and `Cancel` in the enum don't match the input action names in the project settings (`ConfirmTarget` and `CancelTarget`), but we supply the mapping between them in `BindAbilityActivationToInputComponent()`. These are special since we supply the mapping and they don't have to match, but they can match. All other inputs in the enum must match the input action names in the project settings.

For `GameplayAbilities` that will only ever be activated by one input (they will always exist in the same "slot" like a MOBA), I prefer to add a variable to my `UGameplayAbility` subclass where I can define their input. I can then read this from the `ClassDefaultObject` when granting the ability.

<a name="concepts-ga-input-noactivate"></a>
##### 4.6.2.1 Binding to Input without Activating Abilities
If you don't want your `GameplayAbilities` to automatically activate when an input is pressed but still bind them to input to use with `AbilityTasks`, you can add a new bool variable to your `UGameplayAbility` subclass, `bActivateOnInput`, that defaults to `true` and override `UAbilitySystemComponent::AbilityLocalInputPressed()`.

```c++
void UGSAbilitySystemComponent::AbilityLocalInputPressed(int32 InputID)
{
	// Consume the input if this InputID is overloaded with GenericConfirm/Cancel and the GenericConfim/Cancel callback is bound
	if (IsGenericConfirmInputBound(InputID))
	{
		LocalInputConfirm();
		return;
	}

	if (IsGenericCancelInputBound(InputID))
	{
		LocalInputCancel();
		return;
	}

	// ---------------------------------------------------------

	ABILITYLIST_SCOPE_LOCK();
	for (FGameplayAbilitySpec& Spec : ActivatableAbilities.Items)
	{
		if (Spec.InputID == InputID)
		{
			if (Spec.Ability)
			{
				Spec.InputPressed = true;
				if (Spec.IsActive())
				{
					if (Spec.Ability->bReplicateInputDirectly && IsOwnerActorAuthoritative() == false)
					{
						ServerSetInputPressed(Spec.Handle);
					}

					AbilitySpecInputPressed(Spec);

					// Invoke the InputPressed event. This is not replicated here. If someone is listening, they may replicate the InputPressed event to the server.
					InvokeReplicatedEvent(EAbilityGenericReplicatedEvent::InputPressed, Spec.Handle, Spec.ActivationInfo.GetActivationPredictionKey());
				}
				else
				{
					UGSGameplayAbility* GA = Cast<UGSGameplayAbility>(Spec.Ability);
					if (GA && GA->bActivateOnInput)
					{
						// Ability is not active, so try to activate it
						TryActivateAbility(Spec.Handle);
					}
				}
			}
		}
	}
}
```

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ga-granting"></a>
#### 4.6.3 Granting Abilities
Granting a `GameplayAbility` to an `ASC` adds it to the `ASC's` list of `ActivatableAbilities` allowing it to activate the `GameplayAbility` at will if it meets the [`GameplayTag` requirements](#concepts-ga-tags).

We grant `GameplayAbilities` on the server which then automatically replicates the [`GameplayAbilitySpec`](#concepts-ga-spec) to the owning client. Other clients / simulated proxies do not receive the `GameplayAbilitySpec`.

The Sample Project stores a `TArray<TSubclassOf<UGDGameplayAbility>>` on the `Character` class that it reads from and grants when the game starts:
```c++
void AGDCharacterBase::AddCharacterAbilities()
{
	// Grant abilities, but only on the server	
	if (Role != ROLE_Authority || !AbilitySystemComponent.IsValid() || AbilitySystemComponent->bCharacterAbilitiesGiven)
	{
		return;
	}

	for (TSubclassOf<UGDGameplayAbility>& StartupAbility : CharacterAbilities)
	{
		AbilitySystemComponent->GiveAbility(
			FGameplayAbilitySpec(StartupAbility, GetAbilityLevel(StartupAbility.GetDefaultObject()->AbilityID), static_cast<int32>(StartupAbility.GetDefaultObject()->AbilityInputID), this));
	}

	AbilitySystemComponent->bCharacterAbilitiesGiven = true;
}
```

When granting these `GameplayAbilities`, we're creating `GameplayAbilitySpecs` with the `UGameplayAbility` class, the ability level, the input that it is bound to, and the `SourceObject` or who gave this `GameplayAbility` to this `ASC`.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ga-activating"></a>
#### 4.6.4 Activating Abilities
If a `GameplayAbility` is assigned an input action, it will be automatically activated if the input is pressed and it meets its `GameplayTag` requirements. This may not always be the desirable way to activate a `GameplayAbility`. The `ASC` provides four other methods of activating `GameplayAbilities`: by `GameplayTag`, `GameplayAbility` class, `GameplayAbilitySpec` handle, and by an event. Activating a `GameplayAbility` by event allows you to [pass in a payload of data with the event](#concepts-ga-data).

```c++
UFUNCTION(BlueprintCallable, Category = "Abilities")
bool TryActivateAbilitiesByTag(const FGameplayTagContainer& GameplayTagContainer, bool bAllowRemoteActivation = true);

UFUNCTION(BlueprintCallable, Category = "Abilities")
bool TryActivateAbilityByClass(TSubclassOf<UGameplayAbility> InAbilityToActivate, bool bAllowRemoteActivation = true);

bool TryActivateAbility(FGameplayAbilitySpecHandle AbilityToActivate, bool bAllowRemoteActivation = true);

bool TriggerAbilityFromGameplayEvent(FGameplayAbilitySpecHandle AbilityToTrigger, FGameplayAbilityActorInfo* ActorInfo, FGameplayTag Tag, const FGameplayEventData* Payload, UAbilitySystemComponent& Component);

FGameplayAbilitySpecHandle GiveAbilityAndActivateOnce(const FGameplayAbilitySpec& AbilitySpec, const FGameplayEventData* GameplayEventData);
```
To activate a `GameplayAbility` by event, the `GameplayAbility` must have its `Triggers` set up in the `GameplayAbility`. Assign a `GameplayTag` and pick an option for `GameplayEvent`. To send the event, use the function `UAbilitySystemBlueprintLibrary::SendGameplayEventToActor(AActor* Actor, FGameplayTag EventTag, FGameplayEventData Payload)`. Activating a `GameplayAbility` by event allows you to pass in a payload with data.

`GameplayAbility` `Triggers` also allow you to activate the `GameplayAbility` when a `GameplayTag` is added or removed.

**Note:** When activating a `GameplayAbility` from event in Blueprint, you must use the `ActivateAbilityFromEvent` node.

**Note:** Don't forget to call `EndAbility()` when the `GameplayAbility` should terminate unless you have a `GameplayAbility` that will always run like a passive ability.

Activation sequence for **locally predicted** `GameplayAbilities`:
1. **Owning client** calls `TryActivateAbility()`
1. Calls `InternalTryActivateAbility()`
1. Calls `CanActivateAbility()` and returns whether `GameplayTag` requirements are met, if the `ASC` can afford the cost, if the `GameplayAbility` is not on cooldown, and if no other instances are currently active
1. Calls `CallServerTryActivateAbility()` and passes it the `Prediction Key` that it generates
1. Calls `CallActivateAbility()`
1. Calls `PreActivate()` Epic refers to this as "boilerplate init stuff"
1. Calls `ActivateAbility()` finally activating the ability

**Server** receives `CallServerTryActivateAbility()`
1. Calls `ServerTryActivateAbility()`
1. Calls `InternalServerTryActivateAbility()` 
1. Calls `InternalTryActivateAbility()`
1. Calls `CanActivateAbility()` and returns whether `GameplayTag` requirements are met, if the `ASC` can afford the cost, if the `GameplayAbility` is not on cooldown, and if no other instances are currently active
1. Calls `ClientActivateAbilitySucceed()` if successful telling it to update its `ActivationInfo` that its activation was confirmed by the server and broadcasting the `OnConfirmDelegate` delegate. This is not the same as input confirmation.
1. Calls `CallActivateAbility()`
1. Calls `PreActivate()` Epic refers to this as "boilerplate init stuff"
1. Calls `ActivateAbility()` finally activating the ability

If at any time the server fails to activate, it will call `ClientActivateAbilityFailed()`, immediately terminating the client's `GameplayAbility` and undoing any predicted changes.

<a name="concepts-ga-activating-passive"></a>
##### 4.6.4.1 Passive Abilities
To implement passive `GameplayAbilities` that automatically activate and run continuously, override `UGameplayAbility::OnAvatarSet()` which is automatically called when a `GameplayAbility` is granted and the `AvatarActor` is set and call `TryActivateAbility()`.

I recommend adding a `bool` to your custom `UGameplayAbility` class specifying if the `GameplayAbility` should be activated when granted. The Sample Project does this for its passive armor stacking ability.

Passive `GameplayAbilities` will typically have a [`Net Execution Policy`](#concepts-ga-net) of `Server Only`.

```c++
void UGDGameplayAbility::OnAvatarSet(const FGameplayAbilityActorInfo * ActorInfo, const FGameplayAbilitySpec & Spec)
{
	Super::OnAvatarSet(ActorInfo, Spec);

	if (bActivateAbilityOnGranted)
	{
		ActorInfo->AbilitySystemComponent->TryActivateAbility(Spec.Handle, false);
	}
}
```

Epic describes this function as the correct place to initiate passive abilities and to do `BeginPlay` type things.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ga-activating-failedtags"></a>
##### 4.6.4.2 Activation Failed Tags

Abilities have default logic to tell you why an ability activation failed. To enable this, you must set up the GameplayTags that correspond to the default failure cases.

Add these tags (or your own naming convention) to your project:
```
+GameplayTagList=(Tag="Activation.Fail.BlockedByTags",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.CantAffordCost",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.IsDead",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.MissingTags",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.Networking",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.OnCooldown",DevComment="")
```

Then add them to the [`GASDocumentation\Config\DefaultGame.ini`](https://github.com/tranek/GASDocumentation/blob/master/Config/DefaultGame.ini#L8-L13):
```
[/Script/GameplayAbilities.AbilitySystemGlobals]
ActivateFailIsDeadName=Activation.Fail.IsDead
ActivateFailCooldownName=Activation.Fail.OnCooldown
ActivateFailCostName=Activation.Fail.CantAffordCost
ActivateFailTagsBlockedName=Activation.Fail.BlockedByTags
ActivateFailTagsMissingName=Activation.Fail.MissingTags
ActivateFailNetworkingName=Activation.Fail.Networking
```

Now whenever an ability activation fails, this corresponding GameplayTag will be included in output log messages or visible on the `showdebug AbilitySystem` hud.
```
LogAbilitySystem: Display: InternalServerTryActivateAbility. Rejecting ClientActivation of Default__GA_FireGun_C. InternalTryActivateAbility failed: Activation.Fail.BlockedByTags
LogAbilitySystem: Display: ClientActivateAbilityFailed_Implementation. PredictionKey :109 Ability: Default__GA_FireGun_C
```

![Activation Failed Tags Displayed in showdebug AbilitySystem](https://github.com/tranek/GASDocumentation/raw/master/Images/activationfailedtags.png)

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ga-cancelabilities"></a>
#### 4.6.5 Canceling Abilities
To cancel a `GameplayAbility` from within, you call `CancelAbility()`. This will call `EndAbility()` and set its `WasCancelled` parameter to true.

To cancel a `GameplayAbility` externally, the `ASC` provides a few functions:

```c++
/** Cancels the specified ability CDO. */
void CancelAbility(UGameplayAbility* Ability);	

/** Cancels the ability indicated by passed in spec handle. If handle is not found among reactivated abilities nothing happens. */
void CancelAbilityHandle(const FGameplayAbilitySpecHandle& AbilityHandle);

/** Cancel all abilities with the specified tags. Will not cancel the Ignore instance */
void CancelAbilities(const FGameplayTagContainer* WithTags=nullptr, const FGameplayTagContainer* WithoutTags=nullptr, UGameplayAbility* Ignore=nullptr);

/** Cancels all abilities regardless of tags. Will not cancel the ignore instance */
void CancelAllAbilities(UGameplayAbility* Ignore=nullptr);

/** Cancels all abilities and kills any remaining instanced abilities */
virtual void DestroyActiveState();
```

**Note:** I have found that `CancelAllAbilities` doesn't seem to work right if you have a `Non-Instanced` `GameplayAbilities`. It seems to hit the `Non-Instanced` `GameplayAbility` and give up. `CancelAbilities` can handle `Non-Instanced` `GameplayAbilities` better and that is what the Sample Project uses (Jump is a non-instanced `GameplayAbility`). Your mileage may vary.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ga-definition-activeability"></a>
#### 4.6.6 Getting Active Abilities
Beginners often ask "How can I get the active ability?" perhaps to set variables on it or to cancel it. More than one `GameplayAbility` can be active at a time so there is no one "active ability". Instead, you must search through an `ASC's` list of `ActivatableAbilities` (granted `GameplayAbilities` that the `ASC` owns) and find the one matching the [`Asset` or `Granted` `GameplayTag`](#concepts-ga-tags) that you are looking for.

`UAbilitySystemComponent::GetActivatableAbilities()` returns a `TArray<FGameplayAbilitySpec>` for you to iterate over.

The `ASC` also has another helper function that takes in a `GameplayTagContainer` as a parameter to assist in searching instead of manually iterating over the list of `GameplayAbilitySpecs`. The `bOnlyAbilitiesThatSatisfyTagRequirements` parameter will only return `GameplayAbilitySpecs` that satisfy their `GameplayTag` requirements and could be activated right now. For example, you could have two basic attack `GameplayAbilities`, one with a weapon and one with bare fists, and the correct one activates depending on if a weapon is equipped setting the `GameplayTag` requirement. See Epic's comment on the function for more information.
```c++
UAbilitySystemComponent::GetActivatableGameplayAbilitySpecsByAllMatchingTags(const FGameplayTagContainer& GameplayTagContainer, TArray < struct FGameplayAbilitySpec* >& MatchingGameplayAbilities, bool bOnlyAbilitiesThatSatisfyTagRequirements = true)
```

Once you get the `FGameplayAbilitySpec` that you are looking for, you can call `IsActive()` on it.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ga-instancing"></a>
#### 4.6.7 Instancing Policy
A `GameplayAbility's` `Instancing Policy` determines if and how the `GameplayAbility` is instanced when activated.

| `Instancing Policy`     | Description                                                  | Example of when to use                                       |
| ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Instanced Per Actor     | Each `ASC` only has one instance of the `GameplayAbility` that is reused between activations. | This will probably be the `Instancing Policy` that you use the most. You can use it for any ability and provides persistence between activations. The designer is responsible for manually resetting any variables between activations that need it. |
| Instanced Per Execution | Every time a `GameplayAbility` is activated, a new instance of the `GameplayAbility` is created. | The benefit of these `GameplayAbilities` is that the variables are reset everytime you activate. These provide worse performance than `Instanced Per Actor` since they will spawn new `GameplayAbilities` every time they activate. The Sample Project does not use any of these. |
| Non-Instanced           | The `GameplayAbility` operates on its `ClassDefaultObject`. No instances are created. | This has the best performance of the three but is the most restrictive in what can be done with it. `Non-Instanced` `GameplayAbilities` cannot store state, meaning no dynamic variables and no binding to `AbilityTask` delegates. The best place to use them is for frequently used simple abilities like minion basic attacks in a MOBA or RTS. The Sample Project's Jump `GameplayAbility` is `Non-Instanced`. |

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ga-net"></a>
#### 4.6.8 Net Execution Policy
A `GameplayAbility's` `Net Execution Policy` determines who runs the `GameplayAbility` and in what order.

| `Net Execution Policy` | Description                                                  |
| ---------------------- | ------------------------------------------------------------ |
| `Local Only`           | The `GameplayAbility` is only run on the owning client. This could be useful for abilities that only make local cosmetic changes. Single player games should use `Server Only`. |
| `Local Predicted`      | `Local Predicted` `GameplayAbilities` activate first on the owning client and then on the server. The server's version will correct anything that the client predicted incorrectly. See [Prediction](#concepts-p). |
| `Server Only`          | The `GameplayAbility` is only run on the server. Passive `GameplayAbilities` will typically be `Server Only`. Single player games should use this. |
| `Server Initiated`     | `Server Initiated` `GameplayAbilities` activate first on the server and then on the owning client. I personally haven't used these much if any. |

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ga-tags"></a>
#### 4.6.9 Ability Tags
`GameplayAbilities` come with `GameplayTagContainers` with built-in logic. None of these `GameplayTags` are replicated.

| `GameplayTag Container`     | Description                                                  |
| --------------------------- | ------------------------------------------------------------ |
| `Ability Tags`              | `GameplayTags` that the `GameplayAbility` owns. These are just `GameplayTags` to describe the `GameplayAbility`. |
| `Cancel Abilities with Tag` | Other `GameplayAbilities` that have these `GameplayTags` in their `Ability Tags` will be canceled when this `GameplayAbility` is activated. |
| `Block Abilities with Tag`  | Other `GameplayAbilities` that have these `GameplayTags` in their `Ability Tags` are blocked from activating while this `GameplayAbility` is active. |
| `Activation Owned Tags`     | These `GameplayTags` are given to the `GameplayAbility's` owner while this `GameplayAbility` is active. Remember these are not replicated. |
| `Activation Required Tags`  | This `GameplayAbility` can only be activated if the owner has **all** of these `GameplayTags`. |
| `Activation Blocked Tags`   | This `GameplayAbility` cannot be activated if the owner has **any** of these `GameplayTags`. |
| `Source Required Tags`      | This `GameplayAbility` can only be activated if the `Source` has **all** of these `GameplayTags`. The `Source` `GameplayTags` are only set if the `GameplayAbility` is triggered by an event. |
| `Source Blocked Tags`       | This `GameplayAbility` cannot be activated if the `Source` has **any** of these `GameplayTags`. The `Source` `GameplayTags` are only set if the `GameplayAbility` is triggered by an event. |
| `Target Required Tags`      | This `GameplayAbility` can only be activated if the `Target` has **all** of these `GameplayTags`. The `Target` `GameplayTags` are only set if the `GameplayAbility` is triggered by an event. |
| `Target Blocked Tags`       | This `GameplayAbility` cannot be activated if the `Target` has **any** of these `GameplayTags`. The `Target` `GameplayTags` are only set if the `GameplayAbility` is triggered by an event. |

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ga-spec"></a>
#### 4.6.10 Gameplay Ability Spec
A `GameplayAbilitySpec` exists on the `ASC` after a `GameplayAbility` is granted and defines the activatable `GameplayAbility` - `GameplayAbility` class, level, input bindings, and runtime state that must be kept separate from the `GameplayAbility` class.

When a `GameplayAbility` is granted on the server, the server replicates the `GameplayAbilitySpec` to the owning client so that she may activate it.

Activating a `GameplayAbilitySpec` will create an instance (or not for `Non-Instanced` `GameplayAbilities`) of the `GameplayAbility` depending on its `Instancing Policy`.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ga-data"></a>
#### 4.6.11 Passing Data to Abilities
The general paradigm for `GameplayAbilities` is `Activate->Generate Data->Apply->End`. Sometimes you need to act on existing data. GAS provides a few options for getting external data into your `GameplayAbilities`:

| Method                                          | Description                                                  |
| ----------------------------------------------- | ------------------------------------------------------------ |
| Activate `GameplayAbility` by Event             | Activate a `GameplayAbility` with an event containing a payload of data. The event's payload is replicated from client to server for local predicted `GameplayAbilities`. Use the two `Optional Object` or the [`TargetData`](#concepts-targeting-data) variables for arbitrary data that does not fit any of the existing variables. The downside to this is that it prevents you from activating the ability with an input bind. To activate a `GameplayAbility` by event, the `GameplayAbility` must have its `Triggers` set up in the `GameplayAbility`. Assign a `GameplayTag` and pick an option for `GameplayEvent`. To send the event, use the function `UAbilitySystemBlueprintLibrary::SendGameplayEventToActor(AActor* Actor, FGameplayTag EventTag, FGameplayEventData Payload)`. |
| Use `WaitGameplayEvent` `AbilityTask`           | Use the `WaitGameplayEvent` `AbilityTask` to tell the `GameplayAbility` to listen for an event with payload data after it activates. The event payload and the process to send it is the same as activating `GameplayAbilities` by event. The downside to this is that events are not replicated by the `AbilityTask` and should only be used for `Local Only` and `Server Only` `GameplayAbilities`. You potentially could write your own `AbilityTask` that will replicate the event payload. |
| Use `TargetData`                                | A custom `TargetData` struct is a good way to pass arbitrary data between the client and server. |
| Store Data on the `OwnerActor` or `AvatarActor` | Use replicated variables stored on the `OwnerActor`, `AvatarActor`, or any other object that you can get a reference to. This method is the most flexible and will work with `GameplayAbilities` activated by input binds. However, it does not guarantee the data will be synchronized from replication at the time of use. You must ensure that ahead of time - meaning if you set a replicated variable and then immediately activate a `GameplayAbility` there is no guarantee the order that will happen on the receiver due to potential packet loss. |

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ga-commit"></a>
#### 4.6.12 Ability Cost and Cooldown
`GameplayAbilities` come with functionality for optional costs and cooldowns. Costs are predefined amounts of `Attributes` that the `ASC` must have in order to activate the `GameplayAbility` implemented with an `Instant` `GameplayEffect` ([`Cost GE`](#concepts-ge-cost)). Cooldowns are timers that prevent the reactivation of a `GameplayAbility` until it expires and is implemented with a `Duration` `GameplayEffect` ([`Cooldown GE`](#concepts-ge-cooldown)).

Before a `GameplayAbility` calls `UGameplayAbility::Activate()`, it calls `UGameplayAbility::CanActivateAbility()`. This function checks if the owning `ASC` can afford the cost (`UGameplayAbility::CheckCost()`) and ensures that the `GameplayAbility` is not on cooldown (`UGameplayAbility::CheckCooldown()`).

After a `GameplayAbility` calls `Activate()`, it can optionally commit the cost and cooldown at any time using `UGameplayAbility::CommitAbility()` which calls `UGameplayAbility::CommitCost()` and `UGameplayAbility::CommitCooldown()`. The designer may choose to call `CommitCost()` or `CommitCooldown()` separately if they shouldn't be committed at the same time. Committing cost and cooldown calls `CheckCost()` and `CheckCooldown()` one more time and is the last chance for the `GameplayAbility` to fail related to them. The owning `ASC's` `Attributes` could potentially change after a `GameplayAbility` is activated, failing to meet the cost at time of commit. Committing the cost and cooldown can be [locally predicted](#concepts-p) if the [prediction key](#concepts-p-key) is valid at the time of commit.

See [`CostGE`](#concepts-ge-cost) and [`CooldownGE`](#concepts-ge-cooldown) for implementation details.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ga-leveling"></a>
#### 4.6.13 Leveling Up Abilities
There are two common methods for leveling up an ability:

| Level Up Method                            | Description                                                  |
| ------------------------------------------ | ------------------------------------------------------------ |
| Ungrant and Regrant at the New Level       | Ungrant (remove) the `GameplayAbility` from the `ASC` and regrant it back at the next level on the server. This terminates the `GameplayAbility` if it was active at the time. |
| Increase the `GameplayAbilitySpec's` Level | On the server, find the `GameplayAbilitySpec`, increase its level, and mark it dirty so that replicates to the owning client. This method does not terminate the `GameplayAbility` if it was active at the time. |

The main difference between the two methods is if you want active `GameplayAbilities` to be canceled at the time of level up. You will most likely use both methods depending on your `GameplayAbilities`. I recommend adding a `bool` to your `UGameplayAbility` subclass specifying which method to use.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ga-sets"></a>
#### 4.6.14 Ability Sets
`GameplayAbilitySets` are convenience `UDataAsset` classes for holding input bindings and lists of startup `GameplayAbilities` for Characters with logic to grant the `GameplayAbilities`. Subclasses can also include extra logic or properties. Paragon had a `GameplayAbilitySet` per hero that included all of their given `GameplayAbilities`.

I find this class to be unnecessary at least given what I've seen of it so far. The Sample Project handles all of the functionality of `GameplayAbilitySets` inside of the `GDCharacterBase` and its subclasses.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ga-batching"></a>
#### 4.6.15 Ability Batching
Traditional `Gameplay Ability` lifecycle involves a minimum of two or three RPCs from the client to the server.

1. `CallServerTryActivateAbility()`
1. `ServerSetReplicatedTargetData()` (Optional)
1. `ServerEndAbility()`

If a `GameplayAbility` performs all of these actions in one atomic grouping in a frame, we can optimize this workflow to batch (combine) all two or three RPCs into one RPC. `GAS` refers to this RPC optimization as `Ability Batching`. The common example of when to use `Ability Batching` is for hitscan guns. Hitscan guns activate, do a line trace, send the [`TargetData`](#concepts-targeting-data) to the server, and end the ability all in one atomic group in one frame. The [GASShooter](https://github.com/tranek/GASShooter) sample project demonstrates this technique for its hitscan guns.

Semi-Automatic guns are the best case scenario and batch the `CallServerTryActivateAbility()`, `ServerSetReplicatedTargetData()` (the bullet hit result), and `ServerEndAbility()` into one RPC instead of three RPCs.

Full-Automatic/Burst guns batch `CallServerTryActivateAbility()` and `ServerSetReplicatedTargetData()` for the first bullet into one RPC instead of two RPCs. Each subsequent bullet is its own `ServerSetReplicatedTargetData()` RPC. Finally, `ServerEndAbility()` is sent as a separate RPC when the gun stops firing. This is a worst case scenario where we only save one RPC on the first bullet instead of two. This scenario could have also been implemented with activating the ability via a [`Gameplay Event`](#concepts-ga-data) which would send the bullet's `TargetData` in with the `EventPayload` to the server from the client. The downside of the latter approach is that the `TargetData` would have to be generated externally to the ability whereas the batching approach generates the `TargetData` inside of the ability.

`Ability Batching` is disabled by default on the [`ASC`](#concepts-asc). To enable `Ability Batching`, override `ShouldDoServerAbilityRPCBatch()` to return true:

```c++
virtual bool ShouldDoServerAbilityRPCBatch() const override { return true; }
```

Now that `Ability Batching` is enabled, before activating abilities that you want batched, you must create a `FScopedServerAbilityRPCBatcher` struct beforehand. This special struct will try to batch any abilities following it within its scope. Once the `FScopedServerAbilityRPCBatcher` falls out of scope, any abilities activated will not try to batch. `FScopedServerAbilityRPCBatcher` works by having special code in each of the functions that can be batched that intercepts the call from sending the RPC and instead packs the message into a batch struct. When `FScopedServerAbilityRPCBatcher` falls out of scope, it automatically RPCs this batch struct to the server in `UAbilitySystemComponent::EndServerAbilityRPCBatch()`. The server receives the batch RPC in `UAbilitySystemComponent::ServerAbilityRPCBatch_Internal(FServerAbilityRPCBatch& BatchInfo)`. The `BatchInfo` parameter will contain flags for if the ability should end and if input was pressed at the time of activation and the `TargetData` if that was included. This is a good function to put a breakpoint on to confirm that your batching is working properly. Alternatively, use the cvar `AbilitySystem.ServerRPCBatching.Log 1` to enable special ability batching logging.

This mechanism can only be done in C++ and can only activate abilities by their `FGameplayAbilitySpecHandle`.

```c++
bool UGSAbilitySystemComponent::BatchRPCTryActivateAbility(FGameplayAbilitySpecHandle InAbilityHandle, bool EndAbilityImmediately)
{
	bool AbilityActivated = false;
	if (InAbilityHandle.IsValid())
	{
		FScopedServerAbilityRPCBatcher GSAbilityRPCBatcher(this, InAbilityHandle);
		AbilityActivated = TryActivateAbility(InAbilityHandle, true);

		if (EndAbilityImmediately)
		{
			FGameplayAbilitySpec* AbilitySpec = FindAbilitySpecFromHandle(InAbilityHandle);
			if (AbilitySpec)
			{
				UGSGameplayAbility* GSAbility = Cast<UGSGameplayAbility>(AbilitySpec->GetPrimaryInstance());
				GSAbility->ExternalEndAbility();
			}
		}

		return AbilityActivated;
	}

	return AbilityActivated;
}
```

GASShooter reuses the same batched `GameplayAbility` for semi-automatic and full-automatic guns which never directly call `EndAbility()` (it is handled outside of the ability by a local-only ability that manages player input and the call to the batched ability based on the current firemode). Since all of the RPCs must happen within the scope of the `FScopedServerAbilityRPCBatcher`, I provide the `EndAbilityImmediately` parameter so that the controlling/managing local-only can specify whether this ability should batch the `EndAbility()` call (semi-automatic), or not batch the `EndAbility()` call (full-automatic) and the `EndAbility()` call will happen sometime later in its own RPC.

GASShooter exposes a Blueprint node to allow batching abilities which the aforementioned local-only ability uses to trigger the batched ability.

![Activate Batched Ability](https://github.com/tranek/GASDocumentation/raw/master/Images/batchabilityactivate.png)

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-ga-netsecuritypolicy"></a>
#### 4.6.16 Net Security Policy
A `GameplayAbility`'s `NetSecurityPolicy` determines where should an ability execute on the network. It provides protection from clients attempting to execute restricted abilities.

| `NetSecurityPolicy`     | Description                                                  |
| ----------------------- | ------------------------------------------------------------ |
| `ClientOrServer`        | No security requirements. Client or server can trigger execution and termination of this ability freely. |
| `ServerOnlyExecution`   | A client requesting execution of this ability will be ignored by the server. Clients can still request that the server cancel or end this ability. |
| `ServerOnlyTermination` | A client requesting cancellation or ending of this ability will be ignored by the server. Clients can still request execution of the ability. |
| `ServerOnly`            | Server controls both execution and termination of this ability. A client making any requests will be ignored. |

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-at"></a>
### 4.7 Ability Tasks

<a name="concepts-at-definition"></a>
### 4.7.1 Ability Task Definition
`GameplayAbilities` only execute in one frame. This does not allow for much flexibility on its own. To do actions that happen over time or require responding to delegates fired at some point later in time we use latent actions called `AbilityTasks`.

GAS comes with many `AbilityTasks` out of the box:
* Tasks for moving Characters with `RootMotionSource`
* A task for playing animation montages
* Tasks for responding to `Attribute` changes
* Tasks for responding to `GameplayEffect` changes
* Tasks for responding to player input
* and more

The `UAbilityTask` constructor enforces a hardcoded game-wide maximum of 1000 concurrent `AbilityTasks` running at the same time. Keep this in mind when designing `GameplayAbilities` for games that can have hundreds of characters in the world at the same time like RTS games.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-at-definition"></a>
### 4.7.2 Custom Ability Tasks
Often you will be creating your own custom `AbilityTasks` (in C++). The Sample Project comes with two custom `AbilityTasks`:
1. `PlayMontageAndWaitForEvent` is a combination of the default `PlayMontageAndWait` and `WaitGameplayEvent` `AbilityTasks`. This allows animation montages to send gameplay events from `AnimNotifies` back to the `GameplayAbility` that started them. Use this to trigger actions at specific times during animation montages.
1. `WaitReceiveDamage` listens for the `OwnerActor` to receive damage. The passive armor stacks `GameplayAbility` removes a stack of armor when the hero receives an instance of damage.

`AbilityTasks` are composed of:
* A static function that creates new instances of the `AbilityTask`
* Delegates that are broadcasted on when the `AbilityTask` completes its purpose
* An `Activate()` function to start its main job, bind to external delegates, etc.
* An `OnDestroy()` function for cleanup, including external delegates that it bound to
* Callback functions for any external delegates that it bound to
* Member variables and any internal helper functions

**Note:** `AbilityTasks` can only declare one type of output delegate. All of your output delegates must be of this type, regardless if they use the parameters or not. Pass default values for unused delegate parameters.

`AbilityTasks` only run on the Client or Server that is running the owning `GameplayAbility`; however, `AbilityTasks` can be set to run on simulated clients by setting `bSimulatedTask = true;` in the `AbilityTask` constructor, overriding `virtual void InitSimulatedTask(UGameplayTasksComponent& InGameplayTasksComponent);`, and setting any member variables to be replicated. This is only useful in rare situations like movement `AbilityTasks` where you don't want to replicate every movement change but instead simulate the entire movement `AbilityTask`. All of the `RootMotionSource` `AbilityTasks` do this. See `AbilityTask_MoveToLocation.h/.cpp` as an example.

`AbilityTasks` can `Tick` if you set `bTickingTask = true;` in the `AbilityTask` constructor and override `virtual void TickTask(float DeltaTime);`. This is useful when you need to lerp values smoothly across frames. See `AbilityTask_MoveToLocation.h/.cpp` as an example.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-at-using"></a>
### 4.7.3 Using Ability Tasks
To create and activate an `AbilityTask` in C++ (From `GDGA_FireGun.cpp`):
```c++
UGDAT_PlayMontageAndWaitForEvent* Task = UGDAT_PlayMontageAndWaitForEvent::PlayMontageAndWaitForEvent(this, NAME_None, MontageToPlay, FGameplayTagContainer(), 1.0f, NAME_None, false, 1.0f);
Task->OnBlendOut.AddDynamic(this, &UGDGA_FireGun::OnCompleted);
Task->OnCompleted.AddDynamic(this, &UGDGA_FireGun::OnCompleted);
Task->OnInterrupted.AddDynamic(this, &UGDGA_FireGun::OnCancelled);
Task->OnCancelled.AddDynamic(this, &UGDGA_FireGun::OnCancelled);
Task->EventReceived.AddDynamic(this, &UGDGA_FireGun::EventReceived);
Task->ReadyForActivation();
```

In Blueprint, we just use the Blueprint node that we create for the `AbilityTask`. We don't have to call `ReadyForActivation()`. That is automatically called by `Engine/Source/Editor/GameplayTasksEditor/Private/K2Node_LatentGameplayTaskCall.cpp`. `K2Node_LatentGameplayTaskCall` also automatically calls `BeginSpawningActor()` and `FinishSpawningActor()` if they exist in your `AbilityTask` class (see `AbilityTask_WaitTargetData`). To reiterate, `K2Node_LatentGameplayTaskCall` only does automagic sorcery for Blueprint. In C++, we have to manually call `ReadyForActivation()`, `BeginSpawningActor()`, and `FinishSpawningActor()`.

![Blueprint WaitTargetData AbilityTask](https://github.com/tranek/GASDocumentation/raw/master/Images/abilitytask.png)

To manually cancel an `AbilityTask`, just call `EndTask()` on the `AbilityTask` object in Blueprint (called `Async Task Proxy`) or in C++.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-at-rms"></a>
### 4.7.4 Root Motion Source Ability Tasks
GAS comes with `AbilityTasks` for moving `Characters` over time for things like knockbacks, complex jumps, pulls, and dashes using `Root Motion Sources` hooked into the `CharacterMovementComponent`.

**Note:** Predicting `RootMotionSource` `AbilityTasks` works up to engine version 4.19 and 4.25+. Prediction is bugged for engine versions 4.20-4.24; however, the `AbilityTasks` still perform their function in multiplayer with minor net corrections and work perfectly in single player. It is possible to cherry pick the [prediction fix](https://github.com/EpicGames/UnrealEngine/commit/94107438dd9f490e7b743f8e13da46927051bf33#diff-65f6196f9f28f560f95bd578e07e290c) from 4.25 into a custom 4.20-4.24 engine.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-gc"></a>
### 4.8 Gameplay Cues

<a name="concepts-gc-definition"></a>
#### 4.8.1 Gameplay Cue Definition
`GameplayCues` (`GC`) execute non-gameplay related things like sound effects, particle effects, camera shakes, etc. `GameplayCues` are typically replicated (unless explicitly `Executed`, `Added`, or `Removed` locally) and predicted.

We trigger `GameplayCues` by sending a corresponding `GameplayTag` with the **mandatory parent name of `GameplayCue.`** and an event type (`Execute`, `Add`, or `Remove`) to the `GameplayCueManager` via the `ASC`. `GameplayCueNotify` objects and other `Actors` that implement the `IGameplayCueInterface` can subscribe to these events based on the `GameplayCue's` `GameplayTag` (`GameplayCueTag`).

**Note:** Just to reiterate, `GameplayCue` `GameplayTags` need to start with the parent `GameplayTag` of `GameplayCue`. So for example, a valid `GameplayCue` `GameplayTag` might be `GameplayCue.A.B.C`.

There are two classes of `GameplayCueNotifies`, `Static` and `Actor`. They respond to different events and different types of `GameplayEffects` can trigger them. Override the corresponding event with your logic.

| `GameplayCue` Class                                          | Event             | `GameplayEffect` Type    | Description                                                  |
| ------------------------------------------------------------ | ----------------- | ------------------------ | ------------------------------------------------------------ |
| [`GameplayCueNotify_Static`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayCueNotify_Static/index.html) | `Execute`         | `Instant` or `Periodic`  | Static `GameplayCueNotifies` operate on the `ClassDefaultObject` (meaning no instances) and are perfect for one-off effects like hit impacts. |
| [`GameplayCueNotify_Actor`](https://docs.unrealengine.com/en-US/BlueprintAPI/GameplayCueNotify/index.html) | `Add` or `Remove` | `Duration` or `Infinite` | Actor `GameplayCueNotifies` spawn a new instance when `Added`. Because these are instanced, they can do actions over time until they are `Removed`. These are good for looping sounds and particle effects that will be removed when the backing `Duration` or `Infinite` `GameplayEffect` is removed or by manually calling remove. These also come with options to manage how many are allowed to be `Added` at the same time so that multiple applications of the same effect only start the sounds or particles once. |

`GameplayCueNotifies` technically can respond to any of the events but this is typically how we use them.

**Note:** When using `GameplayCueNotify_Actor`, check `Auto Destroy on Remove` otherwise subsequent calls to `Add` that `GameplayCueTag` won't work.

When using an `ASC` [Replication Mode](#concepts-asc-rm) other than `Full`, `Add` and `Remove` `GC` events will fire twice on Server players (listen server) - once for applying the `GE` and again from the "Minimal" `NetMultiCast` to the clients. However, `WhileActive` events will still only fire once. All events will only fire once on clients.

The Sample Project includes a `GameplayCueNotify_Actor` for stun and sprint effects. It also has a `GameplayCueNotify_Static` for the FireGun's projectile impact. These `GCs` can be optimized further by [triggering them locally](#concepts-gc-local) instead of replicating them through a `GE`. I opted for showing the beginner way of using them in the Sample Project.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-gc-trigger"></a>
#### 4.8.2 Triggering Gameplay Cues

From inside of a `GameplayEffect` when it is successfully applied (not blocked by tags or immunity), fill in the `GameplayTags` of all the `GameplayCues` that should be triggered.

![GameplayCue Triggered from a GameplayEffect](https://github.com/tranek/GASDocumentation/raw/master/Images/gcfromge.png)

`UGameplayAbility` offers Blueprint nodes to `Execute`, `Add`, or `Remove` `GameplayCues`.

![GameplayCue Triggered from a GameplayAbility](https://github.com/tranek/GASDocumentation/raw/master/Images/gcfromga.png)

In C++, you can call functions directly on the `ASC` (or expose them to Blueprint in your `ASC` subclass):

```c++
/** GameplayCues can also come on their own. These take an optional effect context to pass through hit result, etc */
void ExecuteGameplayCue(const FGameplayTag GameplayCueTag, FGameplayEffectContextHandle EffectContext = FGameplayEffectContextHandle());
void ExecuteGameplayCue(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);

/** Add a persistent gameplay cue */
void AddGameplayCue(const FGameplayTag GameplayCueTag, FGameplayEffectContextHandle EffectContext = FGameplayEffectContextHandle());
void AddGameplayCue(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);

/** Remove a persistent gameplay cue */
void RemoveGameplayCue(const FGameplayTag GameplayCueTag);
	
/** Removes any GameplayCue added on its own, i.e. not as part of a GameplayEffect. */
void RemoveAllGameplayCues();
```

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-gc-local"></a>
#### 4.8.3 Local Gameplay Cues
The exposed functions for firing `GameplayCues` from `GameplayAbilities` and the `ASC` are replicated by default. Each `GameplayCue` event is a multicast RPC. This can cause a lot of RPCs. GAS also enforces a maximum of two of the same `GameplayCue` RPCs per net update. We avoid this by using local `GameplayCues` where we can. Local `GameplayCues` only `Execute`, `Add`, or `Remove` on the individual client.

Scenarios where we can use local `GameplayCues`:
* Projectile impacts
* Melee collision impacts
* `GameplayCues` fired from animation montages

Local `GameplayCue` functions that you should add to your `ASC` subclass:

```c++
UFUNCTION(BlueprintCallable, Category = "GameplayCue", Meta = (AutoCreateRefTerm = "GameplayCueParameters", GameplayTagFilter = "GameplayCue"))
void ExecuteGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);

UFUNCTION(BlueprintCallable, Category = "GameplayCue", Meta = (AutoCreateRefTerm = "GameplayCueParameters", GameplayTagFilter = "GameplayCue"))
void AddGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);

UFUNCTION(BlueprintCallable, Category = "GameplayCue", Meta = (AutoCreateRefTerm = "GameplayCueParameters", GameplayTagFilter = "GameplayCue"))
void RemoveGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);
```

```c++
void UPAAbilitySystemComponent::ExecuteGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters & GameplayCueParameters)
{
	UAbilitySystemGlobals::Get().GetGameplayCueManager()->HandleGameplayCue(GetOwner(), GameplayCueTag, EGameplayCueEvent::Type::Executed, GameplayCueParameters);
}

void UPAAbilitySystemComponent::AddGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters & GameplayCueParameters)
{
	UAbilitySystemGlobals::Get().GetGameplayCueManager()->HandleGameplayCue(GetOwner(), GameplayCueTag, EGameplayCueEvent::Type::OnActive, GameplayCueParameters);
	UAbilitySystemGlobals::Get().GetGameplayCueManager()->HandleGameplayCue(GetOwner(), GameplayCueTag, EGameplayCueEvent::Type::WhileActive, GameplayCueParameters);
}

void UPAAbilitySystemComponent::RemoveGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters & GameplayCueParameters)
{
	UAbilitySystemGlobals::Get().GetGameplayCueManager()->HandleGameplayCue(GetOwner(), GameplayCueTag, EGameplayCueEvent::Type::Removed, GameplayCueParameters);
}
```

If a `GameplayCue` was `Added` locally, it should be `Removed` locally. If it was `Added` via replication, it should be `Removed` via replication.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-gc-parameters"></a>
#### 4.8.4 Gameplay Cue Parameters
`GameplayCues` receive a `FGameplayCueParameters` structure containing extra information for the `GameplayCue` as a parameter. If you manually trigger the `GameplayCue` from a function on the `GameplayAbility` or the `ASC`, then you must manually fill in the `GameplayCueParameters` structure that is passed to the `GameplayCue`. If the `GameplayCue` is triggered by a `GameplayEffect`, then the following variables are automatically filled in on the `GameplayCueParameters` structure:

* AggregatedSourceTags
* AggregatedTargetTags
* GameplayEffectLevel
* AbilityLevel
* [EffectContext](#concepts-ge-context)
* Magnitude (if the `GameplayEffect` has an `Attribute` for magnitude selected in the dropdown above the `GameplayCue` tag container and a corresponding `Modifier` that affects that `Attribute`)

The `SourceObject` variable in the `GameplayCueParameters` structure is potentially a good place to pass arbitrary data to the `GameplayCue` when triggering the `GameplayCue` manually.

**Note:** Some of the variables in the parameters structure like `Instigator` might already exist in the `EffectContext`. The `EffectContext` can also contain a `FHitResult` for location of where to spawn the `GameplayCue` in the world. Subclassing `EffectContext` is potentially a good way to pass more data to `GameplayCues`, especially those triggered by a `GameplayEffect`.

See the 3 functions in [`UAbilitySystemGlobals`](#concepts-asg) that populate the `GameplayCueParameters` structure for more information. They are virtual so you can override them to autopopulate more information.

```c++
/** Initialize GameplayCue Parameters */
virtual void InitGameplayCueParameters(FGameplayCueParameters& CueParameters, const FGameplayEffectSpecForRPC &Spec);
virtual void InitGameplayCueParameters_GESpec(FGameplayCueParameters& CueParameters, const FGameplayEffectSpec &Spec);
virtual void InitGameplayCueParameters(FGameplayCueParameters& CueParameters, const FGameplayEffectContextHandle& EffectContext);
```

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-gc-manager"></a>
#### 4.8.5 Gameplay Cue Manager
By default, the `GameplayCueManager` will scan the entire game directory for `GameplayCueNotifies` and load them into memory on play. We can change the path where the `GameplayCueManager` scans by setting it in the `DefaultGame.ini`.

```
[/Script/GameplayAbilities.AbilitySystemGlobals]
GameplayCueNotifyPaths="/Game/GASDocumentation/Characters"
```

We do want the `GameplayCueManager` to scan and find all of the `GameplayCueNotifies`; however, we don't want it to async load every single one on play. This will put every `GameplayCueNotify` and all of their referenced sounds and particles into memory regardless if they're even used in a level. In a large game like Paragon, this can be hundreds of megabytes of unneeded assets in memory and cause hitching and game freezes on startup.

An alternative to async loading every `GameplayCue` on startup is to only async load `GameplayCues` as they're triggered in-game. This mitigates the unnecessary memory usage and potential game hard freezes while async loading every `GameplayCue` in exchange for potentially delayed effects for the first time that a specific `GameplayCue` is triggered during play. This potential delay is nonexistent for SSDs. I have not tested on a HDD. If using this option in the UE Editor, there may be slight hitches or freezes during the first load of GameplayCues if the Editor needs to compile particle systems. This is not an issue in builds as the particle systems will already be compiled.

First we must subclass `UGameplayCueManager` and tell the `AbilitySystemGlobals` class to use our `UGameplayCueManager` subclass in `DefaultGame.ini`.

```
[/Script/GameplayAbilities.AbilitySystemGlobals]
GlobalGameplayCueManagerClass="/Script/ParagonAssets.PBGameplayCueManager"
```

In our `UGameplayCueManager` subclass, override `ShouldAsyncLoadRuntimeObjectLibraries()`.

```c++
virtual bool ShouldAsyncLoadRuntimeObjectLibraries() const override
{
	return false;
}
```

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-gc-prevention"></a>
#### 4.8.6 Prevent Gameplay Cues from Firing
Sometimes we don't want `GameplayCues` to fire. For example if we block an attack, we may not want to play the hit impact attached to the damage `GameplayEffect` or play a custom one instead. We can do this inside of [`GameplayEffectExecutionCalculations`](#concepts-ge-ec) by calling `OutExecutionOutput.MarkGameplayCuesHandledManually()` and then manually sending our `GameplayCue` event to the `Target` or `Source's` `ASC`.

If you never want any `GameplayCues` to fire on a specific `ASC`, you can set `AbilitySystemComponent->bSuppressGameplayCues = true;`.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-gc-batching"></a>
#### 4.8.7 Gameplay Cue Batching
Each `GameplayCue` triggered is an unreliable NetMulticast RPC. In situations where we fire multiple `GCs` at the same time, there are a few optimization methods to condense them down into one RPC or save bandwidth by sending less data.

<a name="concepts-gc-batching-manualrpc"></a>
##### 4.8.7.1 Manual RPC
Say you have a shotgun that shoots eight pellets. That's eight trace and impact `GameplayCues`. [GASShooter](https://github.com/tranek/GASShooter) takes the lazy approach of combining them into one RPC by stashing all of the trace information into the [`EffectContext`](#concepts-ge-ec) as [`TargetData`](#concepts-targeting-data). While this reduces the RPCs from eight to one, it still sends a lot of data over the network in that one RPC (~500 bytes). A more optimized approach is to send an RPC with a custom struct where you efficiently encode the hit locations or maybe you give it a random seed number to recreate/approximate the impact locations on the receiving side. The clients would then unpack this custom struct and turn back into [locally executed `GameplayCues`](#concepts-gc-local).

How this works:
1. Declare a `FScopedGameplayCueSendContext`. This suppresses `UGameplayCueManager::FlushPendingCues()` until it falls out of scope, meaning all `GameplayCues` will be queued up until the `FScopedGameplayCueSendContext` falls out of scope.
1. Override `UGameplayCueManager::FlushPendingCues()` to merge `GameplayCues` that can be batched together based on some custom `GameplayTag` into your custom struct and RPC it to clients.
1. Clients receive the custom struct and unpack it into locally executed `GameplayCues`.

This method can also be used when you need specific parameters for your `GameplayCues` that don't fit with what `GameplayCueParameters` offer and you don't want to add them to the `EffectContext` like damage numbers, crit indicator, broken shield indicator, was fatal hit indicator, etc.

https://forums.unrealengine.com/development-discussion/c-gameplay-programming/1711546-fscopedgameplaycuesendcontext-gameplaycuemanager

<a name="concepts-gc-batching-gcsonge"></a>
##### 4.8.7.2 Multiple GCs on one GE
All of the `GameplayCues` on a `GameplayEffect` are sent in one RPC already. By default, `UGameplayCueManager::InvokeGameplayCueAddedAndWhileActive_FromSpec()` will send the whole `GameplayEffectSpec` (but converted to `FGameplayEffectSpecForRPC`) in the unreliable NetMulticast regardless of the `ASC`'s `Replication Mode`. This could potentially be a lot of bandwidth depending on what is in the `GameplayEffectSpec`. We can potentially optimize this by setting the cvar `AbilitySystem.AlwaysConvertGESpecToGCParams 1`. This will convert `GameplayEffectSpecs` to `FGameplayCueParameter` structures and RPC those instead of the whole `FGameplayEffectSpecForRPC`. This potentially saves bandwidth but also has less information, depending on how the `GESpec` is converted to `GameplayCueParameters` and what your `GCs` need to know.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-gc-events"></a>
#### 4.8.8 Gameplay Cue Events
`GameplayCues` respond to specific `EGameplayCueEvents`:

| `EGameplayCueEvent` | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| `OnActive`          | Called when a `GameplayCue` is activated (added).            |
| `WhileActive`       | Called when `GameplayCue` is active, even if it wasn't actually just applied (Join in progress, etc). This is not `Tick`! It's called once just like `OnActive` when a `GameplayCueNotify_Actor` is added or becomes relevant. If you need `Tick()`, just use the `GameplayCueNotify_Actor`'s `Tick()`. It's an `AActor` after all. |
| `Removed`           | Called when a `GameplayCue` is removed. The Blueprint `GameplayCue` function that responds to this event is `OnRemove`. |
| `Executed`          | Called when a `GameplayCue` is executed: instant effects or periodic `Tick()`. The Blueprint `GameplayCue` function that responds to this event is `OnExecute`. |

Use `OnActive` for anything in your `GameplayCue` that happen at the start of the `GameplayCue` but is okay if late joiners miss. Use `WhileActive` for ongoing effects in the `GameplayCue` that you would want late joiners to see. For example, if you have a `GameplayCue` for a tower structure in a MOBA exploding, you would put the initial explosion particle system and explosion sound in `OnActive` and you would put any residual ongoing fire particles or sounds in the `WhileActive`. In this scenario, it wouldn't make sense for late joiners to replay the initial explosion from `OnActive`, but you would want them to see the persistent, looping fire effects on the ground after the explosion happened from `WhileActive`. `OnRemove` should clean up anything added in `OnActive` and `WhileActive`. `WhileActive` will be called every time an Actor enters the relevancy range of a `GameplayCueNotify_Actor`. `OnRemove` will be called every time an Actor leaves relevancy range of a `GameplayCueNotify_Actor`.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-gc-reliability"></a>
#### 4.8.9 Gameplay Cue Reliability

`GameplayCues` in general should be considered unreliable and thus unsuited for anything that directly affects gameplay.

**Executed `GameplayCues`:** These `GameplayCues` are applied via unreliable multicasts and are always unreliable.

**`GameplayCues` applied from `GameplayEffects`:**
* Autonomous proxy reliably receives `OnActive`, `WhileActive`, and `OnRemove`  
`FActiveGameplayEffectsContainer::NetDeltaSerialize()` calls `UAbilitySystemComponent::HandleDeferredGameplayCues()` to call `OnActive` and `WhileActive`. `FActiveGameplayEffectsContainer::RemoveActiveGameplayEffectGrantedTagsAndModifiers()` makes the call to `OnRemoved`.
* Simulated proxies reliably receive `WhileActive` and `OnRemove`  
`UAbilitySystemComponent::MinimalReplicationGameplayCues`'s replication calls `WhileActive` and `OnRemove`. The `OnActive` event is called by an unreliable multicast.

**`GameplayCues` applied without a `GameplayEffect`:**
* Autonomous proxy reliably receives `OnRemove`  
The `OnActive` and `WhileActive` events are called by an unreliable multicast.
* Simulated proxies reliably receive `WhileActive` and `OnRemove`  
`UAbilitySystemComponent::MinimalReplicationGameplayCues`'s replication calls `WhileActive` and `OnRemove`. The `OnActive` event is called by an unreliable multicast.

If you need something in a `GameplayCue` to be 'reliable', then apply it from a `GameplayEffect` and use `WhileActive` to add the FX and `OnRemove` to remove the FX.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-asg"></a>
### 4.9 Ability System Globals
The [`AbilitySystemGlobals`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UAbilitySystemGlobals/index.html) class holds global information about GAS. Most of the variables can be set from the `DefaultGame.ini`. Generally you won't have to interact with this class, but you should be aware of its existence. If you need to subclass things like the [`GameplayCueManager`](#concepts-gc-manager) or the [`GameplayEffectContext`](#concepts-ge-context), you have to do that through the `AbilitySystemGlobals`.

To subclass `AbilitySystemGlobals`, set the class name in the `DefaultGame.ini`:
```
[/Script/GameplayAbilities.AbilitySystemGlobals]
AbilitySystemGlobalsClassName="/Script/ParagonAssets.PAAbilitySystemGlobals"
```

<a name="concepts-asg-initglobaldata"></a>
#### 4.9.1 InitGlobalData()
Between UE 4.24 and 5.2, it is necessary to call `UAbilitySystemGlobals::Get().InitGlobalData()` to use [`TargetData`](#concepts-targeting-data), otherwise you will get errors related to `ScriptStructCache` and clients will be disconnected from the server. This function only needs to be called once in a project. Fortnite calls it from `UAssetManager::StartInitialLoading()` and Paragon called it from `UEngine::Init()`. I find that putting it in `UAssetManager::StartInitialLoading()` is a good place as shown in the Sample Project. I would consider this boilerplate code that you should copy into your project to avoid issues with `TargetData`. Starting in 5.3 it is called automatically.

If you run into a crash while using the `AbilitySystemGlobals` `GlobalAttributeSetDefaultsTableNames`, you may need to call `UAbilitySystemGlobals::Get().InitGlobalData()` later like Fortnite in the `AssetManager` or in the `GameInstance`.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-p"></a>
### 4.10 Prediction
GAS comes out of the box with support for client-side prediction; however, it does not predict everything. Client-side prediction in GAS means that the client does not have to wait for the server's permission to activate a `GameplayAbility` and apply `GameplayEffects`. It can "predict" the server giving it permission to do this and predict the targets that it would apply `GameplayEffects` to. The server then runs the `GameplayAbility` network latency-time after the client activates and tells the client if he was correct or not in his predictions. If the client was wrong in any of his predictions, he will "roll back" his changes from his "mispredictions" to match the server.

The definitive source for GAS-related prediction is `GameplayPrediction.h` in the plugin source code.

Epic's mindset is to only predict what you "can get away with". For example, Paragon and Fortnite do not predict damage. Most likely they use [`ExecutionCalculations`](#concepts-ge-ec) for their damage which cannot be predicted anyway. This is not to say that you can't try to predict certain things like damage. By all means if you do it and it works well for you then that's great.

> ... we are also not all in on a "predict everything: seamlessly and automatically" solution. We still feel player prediction is best kept to a minimum (meaning: predict the minimum amount of stuff you can get away with).

*Dave Ratti from Epic's comment from the new [Network Prediction Plugin](#concepts-p-npp)*

**What is predicted:**
> * Ability activation
> *	Triggered Events
> *	GameplayEffect application:
>    * Attribute modification (EXCEPTIONS: Executions do not currently predict, only attribute modifiers)
>    * GameplayTag modification
> * Gameplay Cue events (both from within predictive gameplay effect and on their own)
> * Montages
> * Movement (built into UE's UCharacterMovement)

**What is not predicted:**
> * GameplayEffect removal
> * GameplayEffect periodic effects (dots ticking)

*From `GameplayPrediction.h`*

While we can predict `GameplayEffect` application, we cannot predict `GameplayEffect` removal. One way that we can work around this limitation is to predict the inverse effect when we want to remove a `GameplayEffect`. Say we predict a movement speed slow of 40%. We can predictively remove it by applying a movement speed buff of 40%. Then remove both `GameplayEffects` at the same time. This is not appropriate for every scenario and support for predicting `GameplayEffect` removal is still needed. Dave Ratti from Epic has expressed desire to add it to a [future iteration of GAS](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89).

Because we cannot predict the removal of `GameplayEffects`, we cannot fully predict `GameplayAbility` cooldowns and there is no inverse `GameplayEffect` workaround for them. The server's replicated `Cooldown GE` will exist on the client and any attempts to bypass this (with `Minimal` replication mode for example) will be rejected by the server. This means clients with higher latencies take longer to tell the server to go on cooldown and to receive the removal of the server's `Cooldown GE`. This means players with higher latencies will have a lower rate of fire than players with lower latencies, giving them a disadvantage against lower latency players. Fortnite avoids this issue by using custom bookkeeping instead of `Cooldown GEs`.

Regarding predicting damage, I personally do not recommend it despite it being one of the first things that most people try when starting with GAS. I especially do not recommend trying to predict death. While you can predict damage, doing so is tricky. If you mispredict applying damage, the player will see the enemy's health jump back up. This can be especially awkward and frustrating if you try to predict death. Say you mispredict a `Character's` death and it starts ragdolling only to stop ragdolling and continue shooting at you when the server corrects it.

**Note:** `Instant`	`GameplayEffects` (like `Cost GEs`) that change `Attributes` can be predicted on yourself seamlessly, predicting `Instant` `Attribute` changes to other characters will show a brief anomaly or "blip" in their `Attributes`. Predicted `Instant` `GameplayEffects` are actually treated like `Infinite` `GameplayEffects` so that they can be rolled back if mispredicted. When the server's `GameplayEffect` is applied, there potentially exists two of the same `GameplayEffect's` causing the `Modifier` to be applied twice or not at all for a brief moment. It will eventually correct itself but sometimes the blip is noticeable to players.

Problems that GAS's prediction implementation is trying to solve:
> 1. "Can I do this?" Basic protocol for prediction.
> 2. "Undo" How to undo side effects when a prediction fails.
> 3. "Redo" How to avoid replaying side effects that we predicted locally but that also get replicated from the server.
> 4. "Completeness" How to be sure we /really/ predicted all side effects.
> 5. "Dependencies" How to manage dependent prediction and chains of predicted events.
> 6. "Override" How to override state predictively that is otherwise replicated/owned by the server.

*From `GameplayPrediction.h`*

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-p-key"></a>
#### 4.10.1 Prediction Key
GAS's prediction works on the concept of a `Prediction Key` which is an integer identifier that the client generates when he activates a `GameplayAbility`.

* Client generates a prediction key when it activates a `GameplayAbility`. This is the `Activation Prediction Key`.
* Client sends this prediction key to the server with `CallServerTryActivateAbility()`.
* Client adds this prediction key to all `GameplayEffects` that it applies while the prediction key is valid.
* Client's prediction key falls out of scope. Further predicted effects in the same `GameplayAbility` need a new [Scoped Prediction Window](#concepts-p-windows).


* Server receives the prediction key from the client.
* Server adds this prediction key to all `GameplayEffects` that it applies.
* Server replicates the prediction key back to the client.


* Client receives replicated `GameplayEffects` from the server with the prediction key used to apply them. If any of the replicated `GameplayEffects` match the `GameplayEffects` that the client applied with the same prediction key, they were predicted correctly. There will temporarily be two copies of the `GameplayEffect` on the target until the client removes its predicted one.
* Client receives the prediction key back from the server. This is the `Replicated Prediction Key`. This prediction key is now marked stale.
* Client removes **all** `GameplayEffects` that it created with the now stale replicated prediction key. `GameplayEffects` replicated by the server will persist. Any `GameplayEffects` that the client added and didn't receive a matching replicated version from the server were mispredicted.

Prediction keys are guaranteed to be valid during an atomic grouping of instructions "window" in `GameplayAbilities` starting with `Activation` from the activation prediction key. You can think of this as being only valid during one frame. Any callbacks from latent action `AbilityTasks` will no longer have a valid prediction key unless the `AbilityTask` has a built-in Synch Point which generates a new [Scoped Prediction Window](#concepts-p-windows).

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-p-windows"></a>
#### 4.10.2 Creating New Prediction Windows in Abilities
To predict more actions in callbacks from `AbilityTasks`, we need to create a new Scoped Prediction Window with a new Scoped Prediction Key. This is sometimes referred to as a Synch Point between the client and server. Some `AbilityTasks` like all of the input related ones come with built-in functionality to create a new scoped prediction window, meaning atomic code in the `AbilityTasks'` callbacks have a valid scoped prediction key to use. Other tasks like the `WaitDelay` task do not have built-in code to create a new scoped prediction window for its callback. If you need to predict actions after an `AbilityTask` that does not have built-in code to create a scoped prediction window like `WaitDelay`, we must manually do that using the `WaitNetSync` `AbilityTask` with the option `OnlyServerWait`. When the client hits a `WaitNetSync` with `OnlyServerWait`, it generates a new scoped prediction key based on the `GameplayAbility's` activation prediction key, RPCs it to the server, and adds it to any new `GameplayEffects` that it applies. When the server hits a `WaitNetSync` with `OnlyServerWait`, it waits until it receives the new scoped prediction key from the client before continuing. This scoped prediction key does the same dance as activation prediction keys - applied to `GameplayEffects` and replicated back to clients to be marked stale. The scoped prediction key is valid until it falls out of scope, meaning the scoped prediction window has closed. So again, only atomic operations, nothing latent, can use the new scoped prediction key.

You can create as many scoped prediction windows as you need.

If you would like to add the synch point functionality to your own custom `AbilityTasks`, look at how the input ones essentially inject the `WaitNetSync` `AbilityTask` code into them.

**Note:** When using `WaitNetSync`, this does block the server's `GameplayAbility` from continuing execution until it hears from the client. This could potentially be abused by malicious users who hack the game and intentionally delay sending their new scoped prediction key. While Epic uses the `WaitNetSync` sparingly, it recommends potentially building a new version of the `AbilityTask` with a delay that automatically continues without the client if this is a concern for you.

The Sample Project uses `WaitNetSync` in the Sprint `GameplayAbility` to create a new scoped prediction window every time we apply the stamina cost so that we can predict it. Ideally we want a valid prediction key when applying costs and cooldowns.

If you have a predicted `GameplayEffect` that is playing twice on the owning client, your prediction key is stale and you're experiencing the "redo" problem. You can usually solve this by putting a `WaitNetSync` `AbilityTask` with `OnlyServerWait` right before you apply the `GameplayEffect` to create a new scoped prediction key.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-p-spawn"></a>
#### 4.10.3 Predictively Spawning Actors
Spawning `Actors` predictively on clients is an advanced topic. GAS does not provide functionality to handle this out of the box (the `SpawnActor` `AbilityTask` only spawns the `Actor` on the server). The key concept is to spawn a replicated `Actor` on both the client and the server.

If the `Actor` is just cosmetic or doesn't serve any gameplay purpose, the simple solution is to override the `Actor's` `IsNetRelevantFor()` function to restrict the server from replicating to the owning client. The owning client would have his locally spawned version and the server and other clients would have the server's replicated version.
```c++
bool APAReplicatedActorExceptOwner::IsNetRelevantFor(const AActor * RealViewer, const AActor * ViewTarget, const FVector & SrcLocation) const
{
	return !IsOwnedBy(ViewTarget);
}
```

If the spawned `Actor` affects gameplay like a projectile that needs to predict damage, then you need advanced logic that is outside of the scope of this documentation. Look at how UnrealTournament predictively spawns projectiles on Epic Games' GitHub. They have a dummy projectile spawned only on the owning client that synchs up with the server's replicated projectile.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-p-future"></a>
#### 4.10.4 Future of Prediction in GAS
`GameplayPrediction.h` states in the future they could potentially add functionality for predicting `GameplayEffect` removal and periodic `GameplayEffects`.

Dave Ratti from Epic has [expressed interest](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89) in fixing the `latency reconciliation` problem for predicting cooldowns, disadvantaging players with higher latencies versus players with lower latencies.

The new [`Network Prediction` plugin](#concepts-p-npp) by Epic is expected to be fully interoperable with the GAS like the `CharacterMovementComponent` *was* before it.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-p-npp"></a>
#### 4.10.5 Network Prediction Plugin
Epic recently started an initiative to replace the `CharacterMovementComponent` with a new `Network Prediction` plugin. This plugin is still in its very early stages but is available to very early access on the Unreal Engine GitHub. It's too soon to tell which future version of the Engine that it will make its experimental beta debut in.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-targeting"></a>
### 4.11 Targeting

<a name="concepts-targeting-data"></a>
#### 4.11.1 Target Data
[`FGameplayAbilityTargetData`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/FGameplayAbilityTargetData/index.html) is a generic structure for targeting data meant to be passed across the network. `TargetData` will typically hold `AActor`/`UObject` references, `FHitResults`, and other generic location/direction/origin information. However, you can subclass it to put essentially anything that you want inside of them as a simple means to [pass data between the client and server in `GameplayAbilities`](#concepts-ga-data). The base struct `FGameplayAbilityTargetData` is not meant to be used directly but instead subclassed. `GAS` comes with a few subclassed `FGameplayAbilityTargetData` structs out of the box located in `GameplayAbilityTargetTypes.h`.

`TargetData` is typically produced by [`Target Actors`](#concepts-targeting-actors) or **created manually** and consumed by [`AbilityTasks`](#concepts-at) and [`GameplayEffects`](#concepts-ge) via the [`EffectContext`](#concepts-ge-context). As a result of being in the `EffectContext`, [`Executions`](#concepts-ge-ec), [`MMCs`](#concepts-ge-mmc), [`GameplayCues`](#concepts-gc), and the functions on the backend of the [`AttributeSet`](#concepts-as) can access the `TargetData`.

We don't typically pass around the `FGameplayAbilityTargetData` directly, instead we use a [`FGameplayAbilityTargetDataHandle`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/FGameplayAbilityTargetDataHandle/index.html) which has an internal TArray of pointers to `FGameplayAbilityTargetData`. This intermediate struct provides support for polymorphism of the `TargetData`.

An example of inheritting from `FGameplayAbilityTargetData`:
```c++
USTRUCT(BlueprintType)
struct MYGAME_API FGameplayAbilityTargetData_CustomData : public FGameplayAbilityTargetData
{
    GENERATED_BODY()
public:

    FGameplayAbilityTargetData_CustomData()
    { }

    UPROPERTY()
    FName CoolName = NAME_None;

    UPROPERTY()
    FPredictionKey MyCoolPredictionKey;

    // This is required for all child structs of FGameplayAbilityTargetData
    virtual UScriptStruct* GetScriptStruct() const override
    {
    	return FGameplayAbilityTargetData_CustomData::StaticStruct();
    }

	// This is required for all child structs of FGameplayAbilityTargetData
    bool NetSerialize(FArchive& Ar, class UPackageMap* Map, bool& bOutSuccess)
    {
	    // The engine already defined NetSerialize for FName & FPredictionKey, thanks Epic!
        CoolName.NetSerialize(Ar, Map, bOutSuccess);
        MyCoolPredictionKey.NetSerialize(Ar, Map, bOutSuccess);
        bOutSuccess = true;
        return true;
    }
}

template<>
struct TStructOpsTypeTraits<FGameplayAbilityTargetData_CustomData> : public TStructOpsTypeTraitsBase2<FGameplayAbilityTargetData_CustomData>
{
	enum
	{
        WithNetSerializer = true // This is REQUIRED for FGameplayAbilityTargetDataHandle net serialization to work
	};
};
```
For adding the target data to a handle:
```c++
UFUNCTION(BlueprintPure)
FGameplayAbilityTargetDataHandle MakeTargetDataFromCustomName(const FName CustomName)
{
	// Create our target data type, 
	// Handle's automatically cleanup and delete this data when the handle is destructed, 
	// if you don't add this to a handle then be careful because this deals with memory management and memory leaks so its safe to just always add it to a handle at some point in the frame!
	FGameplayAbilityTargetData_CustomData* MyCustomData = new FGameplayAbilityTargetData_CustomData();
	// Setup the struct's information to use the inputted name and any other changes we may want to do
	MyCustomData->CoolName = CustomName;
	
	// Make our handle wrapper for Blueprint usage
	FGameplayAbilityTargetDataHandle Handle;
	// Add the target data to our handle
	Handle.Add(MyCustomData);
	// Output our handle to Blueprint
	return Handle
}
```

For getting values it requires doing type safety checking, because the only way to get values from the handle's target data is by using generic C/C++ casting for it which is *NOT* type safe which can cause object slicing and crashes. For type checking there are multiple ways of doing this(however you want honestly) two common ways are:
- Gameplay Tag(s): You can use a subclass hierarchy where you know that anytime a certain code architecture's functionality occurs, you can cast for the base parent type and get its gameplay tag(s) and then compare against those for casting for inherited classes.
- Script Struct & Static Structs: You can instead do direct class comparison(which can involve a lot of IF statements or making some template functions), below is an example of doing this but basically you can get the script struct from any `FGameplayAbilityTargetData`(this is a nice advantage of it being a `USTRUCT` and requiring any inherited classes to specify the struct type in `GetScriptStruct`) and compare if its the type you're looking for. Below is an example of using these functions for type checking:
```c++
UFUNCTION(BlueprintPure)
FName GetCoolNameFromTargetData(const FGameplayAbilityTargetDataHandle& Handle, const int Index)
{   
    // NOTE, there is two versions of this '::Get(int32 Index)' function; 
    // 1) const version that returns 'const FGameplayAbilityTargetData*', good for reading target data values 
    // 2) non-const version that returns 'FGameplayAbilityTargetData*', good for modifying target data values
    FGameplayAbilityTargetData* Data = Handle.Get(Index); // This will valid check the index for you 
    
    // Valid check we have something to use, null data means nothing to cast for
    if(Data == nullptr)
    {
       	return NAME_None;
    }
    // This is basically the type checking pass, static_cast does not have type safety, this is why we do this check.
    // If we don't do this then it will object slice the struct and thus we have no way of making sure its that type.
    if(Data->GetScriptStruct() == FGameplayAbilityTargetData_CustomData::StaticStruct())
    {
        // Here is when you would do the cast because we know its the correct type already
        FGameplayAbilityTargetData_CustomData* CustomData = static_cast<FGameplayAbilityTargetData_CustomData*>(Data);    
        return CustomData->CoolName;
    }
    return NAME_None;
}
```

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-targeting-actors"></a>
#### 4.11.2 Target Actors
`GameplayAbilities` spawn [`TargetActors`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/AGameplayAbilityTargetActor/index.html) with the `WaitTargetData` `AbilityTask` to visualize and capture targeting information from the world. `TargetActors` may optionally use [`GameplayAbilityWorldReticles`](#concepts-targeting-reticles) to display current targets. Upon confirmation, the targeting information is returned as [`TargetData`](#concepts-targeting-data) which can then be passed into `GameplayEffects`.

`TargetActors` are based on `AActor` so they can have any kind of visible component to represent **where** and **how** they are targeting such as static meshes or decals. Static meshes may be used to visualize placement of an object that your character will build. Decals may be used to show an area of effect on the ground. The Sample Project uses [`AGameplayAbilityTargetActor_GroundTrace`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/AGameplayAbilityTargetActor_Grou-/index.html) with a decal on the ground to represent the damage area of effect for the Meteor ability. They also don't need to display anything either. For example it wouldn't make sense to display anything for a hitscan gun that instantly traces a line to its target as used in [GASShooter](https://github.com/tranek/GASShooter).

They capture targeting information using basic traces or collision overlaps and convert the results as `FHitResults` or `AActor` arrays to `TargetData` depending on the `TargetActor` implementation. The `WaitTargetData` `AbilityTask` determines when the targets are confirmed through its `TEnumAsByte<EGameplayTargetingConfirmation::Type> ConfirmationType` parameter. When **not** using `TEnumAsByte<EGameplayTargetingConfirmation::Type::Instant`, the `TargetActor` typically performs the trace/overlap on `Tick()` and updates its location to the `FHitResult` depending on its implementation. While this performs a trace/overlap on `Tick()`, it's generally not terrible since it's not replicated and you typically don't have more than one (although you could have more) `TargetActor` running at a time. Just be aware that it uses `Tick()` and some complex `TargetActors` might do a lot on it like the rocket launcher's secondary ability in GASShooter. While tracing on `Tick()` is very responsive to the client, you may consider lowering the tick rate on the `TargetActor` if the performance hit is too much. In the case of `TEnumAsByte<EGameplayTargetingConfirmation::Type::Instant`, the `TargetActor` immediately spawns, produces `TargetData`, and destroys. `Tick()` is never called. 

| `EGameplayTargetingConfirmation::Type` | When targets are confirmed                                   |
| -------------------------------------- | ------------------------------------------------------------ |
| `Instant`                              | The targeting happens instantly without special logic or user input deciding when to 'fire'. |
| `UserConfirmed`                        | The targeting happens when the user confirms the targeting when the [ability is bound to a `Confirm` input](#concepts-ga-input) or by calling `UAbilitySystemComponent::TargetConfirm()`. The `TargetActor` will also respond to a bound `Cancel` input or call to `UAbilitySystemComponent::TargetCancel()` to cancel targeting. |
| `Custom`                               | The GameplayTargeting Ability is responsible for deciding when the targeting data is ready by calling `UGameplayAbility::ConfirmTaskByInstanceName()`. The `TargetActor` will also respond to `UGameplayAbility::CancelTaskByInstanceName()` to cancel targeting. |
| `CustomMulti`                          | The GameplayTargeting Ability is responsible for deciding when the targeting data is ready by calling `UGameplayAbility::ConfirmTaskByInstanceName()`. The `TargetActor` will also respond to `UGameplayAbility::CancelTaskByInstanceName()` to cancel targeting. Should not end the `AbilityTask` upon data production. |

Not every EGameplayTargetingConfirmation::Type is supported by every `TargetActor`. For example, `AGameplayAbilityTargetActor_GroundTrace` does not support `Instant` confirmation.

The `WaitTargetData` `AbilityTask` takes in a `AGameplayAbilityTargetActor` class as a parameter and will spawn an instance on each activation of the `AbilityTask` and will destroy the `TargetActor` when the `AbilityTask` ends. The `WaitTargetDataUsingActor` `AbilityTask` takes in an already spawned `TargetActor`, but still destroys it when the `AbilityTask` ends. Both of these `AbilityTasks` are inefficient in that they either spawn or require a newly spawned `TargetActor` for each use. They're great for prototyping, but in production you might explore optimizing it if you have cases where you are constantly producing `TargetData` like in the case of an automatic rifle. GASShooter has a custom subclass of [`AGameplayAbilityTargetActor`](https://github.com/tranek/GASShooter/blob/master/Source/GASShooter/Public/Characters/Abilities/GSGATA_Trace.h) and a new [`WaitTargetDataWithReusableActor`](https://github.com/tranek/GASShooter/blob/master/Source/GASShooter/Public/Characters/Abilities/AbilityTasks/GSAT_WaitTargetDataUsingActor.h) `AbilityTask` written from scratch that allows you to reuse a `TargetActor` without destroying it.

`TargetActors` are not replicated by default; however, they can be made to replicate if that makes sense in your game to show other players where the local player is targeting. They do include default functionality to communicate with the server via RPCs on the `WaitTargetData` `AbilityTask`. If the `TargetActor`'s `ShouldProduceTargetDataOnServer` property is set to `false`, then the client will RPC its `TargetData` to the server on confirmation via `CallServerSetReplicatedTargetData()` in `UAbilityTask_WaitTargetData::OnTargetDataReadyCallback()`. If `ShouldProduceTargetDataOnServer` is `true`, the client will send a generic confirm event, `EAbilityGenericReplicatedEvent::GenericConfirm`, RPC to the server in `UAbilityTask_WaitTargetData::OnTargetDataReadyCallback()` and the server will do the trace or overlap check upon receiving the RPC to produce data on the server. If the client cancels the targeting, it will send a generic cancel event, `EAbilityGenericReplicatedEvent::GenericCancel`, RPC to the server in `UAbilityTask_WaitTargetData::OnTargetDataCancelledCallback`. As you can see, there are a lot of delegates on both the `TargetActor` and the `WaitTargetData` `AbilityTask`. The `TargetActor` responds to inputs to produce and broadcast `TargetData` ready, confirm, or cancel delegates. `WaitTargetData` listens to the `TargetActor`'s `TargetData` ready, confirm, and cancel delegates and relays that information back to the `GameplayAbility` and to the server. If you send `TargetData` to the server, you may want to do validation on the server to make sure the `TargetData` looks reasonable to prevent cheating. Producing the `TargetData` directly on the server avoids this issue entirely, but will potentially lead to mispredictions for the owning client.

Depending on the particular subclass of `AGameplayAbilityTargetActor` that you use, different `ExposeOnSpawn` parameters will be exposed on the `WaitTargetData` `AbilityTask` node. Some common parameters include:

| Common `TargetActor` Parameters | Definition                                                   |
| ------------------------------- | ------------------------------------------------------------ |
| Debug                           | If `true`, it will draw debug tracing/overlapping information whenever the `TargetActor` performs a trace in non-shipping builds. Remember, non-`Instant` `TargetActors` will perform a trace on `Tick()` so these debug draw calls will also happen on `Tick()`. |
| Filter                          | [Optional] A special struct for filtering out (removing) `Actors` from the targets when the trace/overlap happens. Typical use cases are to filter out the player's `Pawn`, require targets be of a specific class. See [Target Data Filters](#concepts-target-data-filters) for more advanced use cases. |
| Reticle Class                   | [Optional] Subclass of `AGameplayAbilityWorldReticle` that the `TargetActor` will spawn. |
| Reticle Parameters              | [Optional] Configure your Reticles. See [Reticles](#concepts-targeting-reticles). |
| Start Location                  | A special struct for where tracing should start from. Typically this will be the player's viewpoint, a weapon muzzle, or the `Pawn`'s location. |

With the default `TargetActor` classes, `Actors` are only valid targets when they are directly in the trace/overlap. If they leave the trace/overlap (they move or you look away), they are no longer valid. If you want the `TargetActor` to remember the last valid target(s), you will need to add this functionality to a custom `TargetActor` class. I refer to these as persistent targets as they will persist until the `TargetActor` receives confirmation or cancellation, the `TargetActor` finds a new valid target in its trace/overlap, or the target is no longer valid (destroyed). GASShooter uses persistent targets for its rocket launcher's secondary ability's homing rockets targeting.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-target-data-filters"></a>
#### 4.11.3 Target Data Filters
Using both the `Make GameplayTargetDataFilter` and `Make Filter Handle` nodes, you can filter out the player's `Pawn` or select only a specific class. If you need more advanced filtering, you can subclass `FGameplayTargetDataFilter` and override the `FilterPassesForActor` function. 
```c++
USTRUCT(BlueprintType)
struct GASDOCUMENTATION_API FGDNameTargetDataFilter : public FGameplayTargetDataFilter
{
	GENERATED_BODY()

	/** Returns true if the actor passes the filter and will be targeted */
	virtual bool FilterPassesForActor(const AActor* ActorToBeFiltered) const override;
};
```

However, this will not work directly into the `Wait Target Data` node as it requires a `FGameplayTargetDataFilterHandle`. A new custom `Make Filter Handle` must be made to accept the subclass:
```c++
FGameplayTargetDataFilterHandle UGDTargetDataFilterBlueprintLibrary::MakeGDNameFilterHandle(FGDNameTargetDataFilter Filter, AActor* FilterActor)
{
	FGameplayTargetDataFilter* NewFilter = new FGDNameTargetDataFilter(Filter);
	NewFilter->InitializeFilterContext(FilterActor);

	FGameplayTargetDataFilterHandle FilterHandle;
	FilterHandle.Filter = TSharedPtr<FGameplayTargetDataFilter>(NewFilter);
	return FilterHandle;
}
```

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-targeting-reticles"></a>
#### 4.11.4 Gameplay Ability World Reticles
[`AGameplayAbilityWorldReticles`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/AGameplayAbilityWorldReticle/index.html) (`Reticles`) visualize **who** you are targeting when targeting with non-`Instant` confirmed [`TargetActors`](#concepts-targeting-actors). `TargetActors` are responsible for the spawn and destroy lifetimes for all `Reticles`. `Reticles` are `AActors` so they can use any kind of visual component for representation. A common implementation as seen in [GASShooter](https://github.com/tranek/GASShooter) is to use a `WidgetComponent` to display a UMG Widget in screen space (always facing the player's camera). `Reticles` do not know which `AActor` that they're on, but you could subclass in that functionality on a custom `TargetActor`. `TargetActors` will typically update the `Reticle`'s location to the target's location on every `Tick()`.

GASShooter uses `Reticles` to show locked-on targets for the rocket launcher's secondary ability's homing rockets. The red indicator on the enemy is the `Reticle`. The similar white image is the rocket launcher's crosshair.
![Reticles in GASShooter](https://github.com/tranek/GASDocumentation/raw/master/Images/gameplayabilityworldreticle.png)

`Reticles` come with a handful of `BlueprintImplementableEvents` for designers (they're intended to be developed in Blueprints):

```c++
/** Called whenever bIsTargetValid changes value. */
UFUNCTION(BlueprintImplementableEvent, Category = Reticle)
void OnValidTargetChanged(bool bNewValue);

/** Called whenever bIsTargetAnActor changes value. */
UFUNCTION(BlueprintImplementableEvent, Category = Reticle)
void OnTargetingAnActor(bool bNewValue);

UFUNCTION(BlueprintImplementableEvent, Category = Reticle)
void OnParametersInitialized();

UFUNCTION(BlueprintImplementableEvent, Category = Reticle)
void SetReticleMaterialParamFloat(FName ParamName, float value);

UFUNCTION(BlueprintImplementableEvent, Category = Reticle)
void SetReticleMaterialParamVector(FName ParamName, FVector value);
```

`Reticles` can optionally use [`FWorldReticleParameters`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/FWorldReticleParameters/index.html) provided by the `TargetActor` for configuration. The default struct only provides one variable `FVector AOEScale`. While you can technically subclass this struct, the `TargetActor` will only accept the base struct. It seems a little short-sighted to not allow this to be subclassed with default `TargetActors`. However, if you make your own custom `TargetActor`, you can provide your own custom reticle parameters struct and manually pass it to your subclass of `AGameplayAbilityWorldReticles` when you spawn them.

`Reticles` are not replicated by default, but can be made replicated if it makes sense for your game to show other players who the local player is targeting.

`Reticles` will only display on the current valid target with the default `TargetActors`. For example, if you're using a `AGameplayAbilityTargetActor_SingleLineTrace` to trace for a target, the `Reticle` will only appear when the enemy is directly in the trace path. If you look away, the enemy is no longer a valid target and the `Reticle` will disappear. If you want the `Reticle` to stay on the last valid target, you will want to customize your `TargetActor` to remember the last valid target and keep the `Reticle` on them. I refer to these as persistent targets as they will persist until the `TargetActor` receives confirmation or cancellation, the `TargetActor` finds a new valid target in its trace/overlap, or the target is no longer valid (destroyed).  GASShooter uses persistent targets for its rocket launcher's secondary ability's homing rockets targeting.

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-targeting-containers"></a>
#### 4.11.5 Gameplay Effect Containers Targeting
[`GameplayEffectContainers`](#concepts-ge-containers) come with an optional, efficient means of producing [`TargetData`](#concepts-targeting-data). This targeting takes place instantly when the `EffectContainer` is applied on the client and the server. It's more efficient than [`TargetActors`](#concepts-targeting-actors) because it runs on the CDO of the targeting object (no spawning and destroying of `Actors`), but it lacks player input, happens instantly without needing confirmation, cannot be canceled, and cannot send data from the client to the server (produces data on both). It works well for instant traces and collision overlaps. Epic's [Action RPG Sample Project](https://www.unrealengine.com/marketplace/en-US/product/action-rpg) includes two example types of targeting with its containers - target the ability owner and pull `TargetData` from an event. It also implements one in Blueprint to do instant sphere traces at some offset (set by child Blueprint classes) from the player. You can subclass `URPGTargetType` in C++ or Blueprint to make your own targeting types.

**[⬆ Back to Top](#table-of-contents)**

<a name="cae"></a>
## 5. Commonly Implemented Abilities and Effects

<a name="cae-stun"></a>
### 5.1 Stun
Typically with stuns, we want to cancel all of a `Character's` active `GameplayAbilities`, prevent new `GameplayAbility` activations, and prevent movement throughout the duration of the stun. The Sample Project's Meteor `GameplayAbility` applies a stun on hit targets.

To cancel the target's active `GameplayAbilities`, we call `AbilitySystemComponent->CancelAbilities()` when the stun [`GameplayTag` is added](#concepts-gt-change).

To prevent new `GameplayAbilities` from activating while stunned, the `GameplayAbilities` are given the stun `GameplayTag` in their [`Activation Blocked Tags` `GameplayTagContainer`](#concepts-ga-tags).

To prevent movement while stunned, we override the `CharacterMovementComponent's` `GetMaxSpeed()` function to return 0 when the owner has the stun `GameplayTag`.

**[⬆ Back to Top](#table-of-contents)**

<a name="cae-sprint"></a>
### 5.2 Sprint
The Sample Project provides an example of how to sprint - run faster while `Left Shift` is held down.

The faster movement is handled predictively by the `CharacterMovementComponent` by sending a flag over the network to the server. See `GDCharacterMovementComponent.h/cpp` for details.

The `GA` handles responding to the `Left Shift` input, tells the `CMC` to begin and stop sprinting, and to predictively charge stamina while `Left Shift` is pressed. See `GA_Sprint_BP` for details.

**[⬆ Back to Top](#table-of-contents)**

<a name="cae-ads"></a>
### 5.3 Aim Down Sights
The Sample Project handles this the exact same way as sprinting but decreasing the movement speed instead of increasing it.

See `GDCharacterMovementComponent.h/cpp` for details on predictively decreasing the movement speed.

See `GA_AimDownSight_BP` for details on handling the input. There is no stamina cost for aiming down sights.

**[⬆ Back to Top](#table-of-contents)**

<a name="cae-ls"></a>
### 5.4 Lifesteal
I handle lifesteal inside of the damage [`ExecutionCalculation`](#concepts-ge-ec). The `GameplayEffect` will have a `GameplayTag` on it like `Effect.CanLifesteal`. The `ExecutionCalculation` checks if the `GameplayEffectSpec` has that `Effect.CanLifesteal` `GameplayTag`. If the `GameplayTag` exists, the `ExecutionCalculation` [creates a dynamic `Instant` `GameplayEffect`](#concepts-ge-dynamic) with the amount of health to give as the modifier and applies it back to the `Source's` `ASC`.

```c++
if (SpecAssetTags.HasTag(FGameplayTag::RequestGameplayTag(FName("Effect.Damage.CanLifesteal"))))
{
	float Lifesteal = Damage * LifestealPercent;

	UGameplayEffect* GELifesteal = NewObject<UGameplayEffect>(GetTransientPackage(), FName(TEXT("Lifesteal")));
	GELifesteal->DurationPolicy = EGameplayEffectDurationType::Instant;

	int32 Idx = GELifesteal->Modifiers.Num();
	GELifesteal->Modifiers.SetNum(Idx + 1);
	FGameplayModifierInfo& Info = GELifesteal->Modifiers[Idx];
	Info.ModifierMagnitude = FScalableFloat(Lifesteal);
	Info.ModifierOp = EGameplayModOp::Additive;
	Info.Attribute = UPAAttributeSetBase::GetHealthAttribute();

	SourceAbilitySystemComponent->ApplyGameplayEffectToSelf(GELifesteal, 1.0f, SourceAbilitySystemComponent->MakeEffectContext());
}
```

**[⬆ Back to Top](#table-of-contents)**

<a name="cae-random"></a>
### 5.5 Generating a Random Number on Client and Server
Sometimes you need to generate a "random" number inside of a `GameplayAbility` for things like bullet recoil or spread. The client and the server will both want to generate the same random numbers. To do this, we must set the `random seed` to be the same at the time of `GameplayAbility` activation. You will want to set the `random seed` each time you activate the `GameplayAbility` in case the client mispredicts activation and its random number sequence becomes out of synch with the server's.

| Seed Setting Method                                          | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Use the activation prediction key                            | The `GameplayAbility` activation prediction key is an int16 guaranteed to be synchronized and available in both the client and server in the `Activation()`. You can set this as the `random seed` on both the client and the server. The downside to this method is that the prediction key always starts at zero each time the game starts and consistently increments the value to use between generating keys. This means each match will have the exact same random number sequence. This may or may not be random enough for your needs. |
| Send a seed through an event payload when you activate the `GameplayAbility` | Activate your `GameplayAbility` by event and send the randomly generated seed from the client to the server via the replicated event payload. This allows for more randomness but the client could easily hack their game to only send the same seed value every time. Also activating `GameplayAbilities` by event will prevent them from activating from the input bind. |

If your random deviation is small, most players won't notice that the sequence is the same every game and using the activation prediction key as the `random seed` should work for you. If you're doing something more complex that needs to be hacker proof, perhaps using a `Server Initiated` `GameplayAbility` would work better where the server can create the prediction key or generate the `random seed` to send via an event payload.

**[⬆ Back to Top](#table-of-contents)**

<a name="cae-crit"></a>
### 5.6 Critical Hits
I handle critical hits inside of the damage [`ExecutionCalculation`](#concepts-ge-ec). The `GameplayEffect` will have a `GameplayTag` on it like `Effect.CanCrit`. The `ExecutionCalculation` checks if the `GameplayEffectSpec` has that `Effect.CanCrit` `GameplayTag`. If the `GameplayTag` exists, the `ExecutionCalculation` generates a random number corresponding to the critical hit chance (`Attribute` captured from the `Source`) and adds the critical hit damage (also an `Attribute` captured from the `Source`) if it succeeded. Since I don't predict damage, I don't have to worry about synchronizing the random number generators on the client and server since the `ExecutionCalculation` will only run on the server. If you tried to do this predictively using an `MMC` to do your damage calculation, you would have to get a reference to the `random seed` from the `GameplayEffectSpec->GameplayEffectContext->GameplayAbilityInstance`.

See how [GASShooter](https://github.com/tranek/GASShooter) does headshots. It's the same concept except that it does not rely on a random number for chance and instead checks the `FHitResult` bone name.

**[⬆ Back to Top](#table-of-contents)**

<a name="cae-nonstackingge"></a>
### 5.7 Non-Stacking Gameplay Effects but Only the Greatest Magnitude Actually Affects the Target
Slow effects in Paragon did not stack. Each slow instance applied and kept track of their lifetimes as normal, but only the greatest magnitude slow effect actually affected the `Character`. GAS provides for this scenario out of the box with `AggregatorEvaluateMetaData`. See [`AggregatorEvaluateMetaData()`](#concepts-as-onattributeaggregatorcreated) for details and implementation.

**[⬆ Back to Top](#table-of-contents)**

<a name="cae-paused"></a>
### 5.8 Generate Target Data While Game is Paused
If you need to pause the game while waiting to generate [`TargetData`](#concepts-targeting-data) from a `WaitTargetData` `AbilityTask` from your player, I suggest instead of pausing to use `slomo 0`.

**[⬆ Back to Top](#table-of-contents)**

<a name="cae-onebuttoninteractionsystem"></a>
### 5.9 One Button Interaction System
[GASShooter](https://github.com/tranek/GASShooter) implements a one button interaction system where the player can press or hold 'E' to interact with interactable objects like reviving a player, opening a weapon chest, and opening or closing a sliding door.

**[⬆ Back to Top](#table-of-contents)**

<a name="debugging"></a>
## 6. Debugging GAS
Often when debugging GAS related issues, you want to know things like:
> * "What are the values of my attributes?"
> * "What gameplay tags do I have?"
> * "What gameplay effects do I currently have?"
> * "What abilities do I have granted, which ones are running, and which ones are blocked from activating?".

GAS comes with two techniques for answering these questions at runtime - [`showdebug abilitysystem`](#debugging-sd) and hooks in the [`GameplayDebugger`](#debugging-gd).

**Tip:** Unreal Engine likes to optimize C++ code which makes it hard to debug some functions. You will encounter this rarely when tracing deep into your code. If setting your Visual Studio solution configuration to `DebugGame Editor` still prevents tracing code or inspecting variables, you can disable all optimizations by wrapping the optimized function with the `UE_DISABLE_OPTIMIZATION` and `UE_ENABLE_OPTIMIZATION` macros or the ship variations defined in CoreMiscDefines.h. This cannot be used on the plugin code unless you rebuild the plugin from source. This may or may not work on inline functions depending on what they do and where they are. Be sure to remove the macros when you're done debugging!

```c++
UE_DISABLE_OPTIMIZATION
void MyClass::MyFunction(int32 MyIntParameter)
{
	// My code
}
UE_ENABLE_OPTIMIZATION
```

**[⬆ Back to Top](#table-of-contents)**

<a name="debugging-sd"></a>
### 6.1 showdebug abilitysystem
Type `showdebug abilitysystem` in the in-game console. This feature is split into three "pages". All three pages will show the `GameplayTags` that you currently have. Type `AbilitySystem.Debug.NextCategory` into the console to cycle between the pages.

The first page shows the `CurrentValue` of all of your `Attributes`:
![First Page of showdebug abilitysystem](https://github.com/tranek/GASDocumentation/raw/master/Images/showdebugpage1.png)

The second page shows all of the `Duration` and `Infinite` `GameplayEffects` on you, their number of stacks, what `GameplayTags` they give, and what `Modifiers` they give.
![Second Page of showdebug abilitysystem](https://github.com/tranek/GASDocumentation/raw/master/Images/showdebugpage2.png)

The third page shows all of the `GameplayAbilities` that have been granted to you, whether they are currently running, whether they are blocked from activating, and the status of currently running `AbilityTasks`.
![Third Page of showdebug abilitysystem](https://github.com/tranek/GASDocumentation/raw/master/Images/showdebugpage3.png)

To cycle between targets (denoted by a green rectangular prism around the Actor), use the `PageUp` key or `NextDebugTarget` console command to go to the next target and the `PageDown` key or `PreviousDebugTarget` console command to go to the previous target.

**Note:** In order for the ability system information to update based on the currently selected debug Actor, you need to set `bUseDebugTargetFromHud=true` in the `AbilitySystemGlobals` like so in the `DefaultGame.ini`:
```
[/Script/GameplayAbilities.AbilitySystemGlobals]
bUseDebugTargetFromHud=true
```

**Note:** For `showdebug abilitysystem` to work an actual HUD class must be selected in the GameMode. Otherwise the command is not found and "Unknown Command" is returned.

**[⬆ Back to Top](#table-of-contents)**

<a name="debugging-gd"></a>
### 6.2 Gameplay Debugger
GAS adds functionality to the Gameplay Debugger. Access the Gameplay Debugger with the Apostrophe (') key. Enable the Abilities category by pressing 3 on your numpad. The category may be different depending on what plugins you have. If your keyboard doesn't have a numpad like a laptop, then you can change the keybindings in the project settings.

Use the Gameplay Debugger when you want to see the `GameplayTags`, `GameplayEffects`, and `GameplayAbilities` on **other** `Characters`. Unfortunately it does not show the `CurrentValue` of the target's `Attributes`. It will target whatever `Character` is in the center of your screen. You can change targets by selecting them in the World Outliner in the Editor or by looking at a different `Character` and press Apostrophe (') again. The currently inspected `Character` has the largest red circle above it.

![Gameplay Debugger](https://github.com/tranek/GASDocumentation/raw/master/Images/gameplaydebugger.png)

**[⬆ Back to Top](#table-of-contents)**

<a name="debugging-log"></a>
### 6.3 GAS Logging
The GAS source code contains a lot of logging statements produced at varying verbosity levels. You will most likely see these as `ABILITY_LOG()` statements. The default verbosity level is `Display`. Anything higher will not be displayed in the console by default.

To change the verbosity level of a log category, type into your console:

```
log [category] [verbosity]
```

For example, to turn on `ABILITY_LOG()` statements, you would type into your console:
```
log LogAbilitySystem VeryVerbose
```

To reset it back to default, type:
```
log LogAbilitySystem Display
```

To display all log categories, type:
```
log list
```

Notable GAS related logging categories:

| Logging Category          | Default Verbosity Level |
| ------------------------- | ----------------------- |
| LogAbilitySystem          | Display                 |
| LogAbilitySystemComponent | Log                     |
| LogGameplayCueDetails     | Log                     |
| LogGameplayCueTranslator  | Display                 |
| LogGameplayEffectDetails  | Log                     |
| LogGameplayEffects        | Display                 |
| LogGameplayTags           | Log                     |
| LogGameplayTasks          | Log                     |
| VLogAbilitySystem         | Display                 |

See the [Wiki on Logging](https://unrealcommunity.wiki/logging-lgpidy6i) for more information.

**[⬆ Back to Top](#table-of-contents)**

<a name="optimizations"></a>
## 7. Optimizations

<a name="optimizations-abilitybatching"></a>
### 7.1 Ability Batching
[`GameplayAbilities`](#concepts-ga) that activate, optionally send `TargetData` to the server, and end all in one frame can be [batched to condense two-three RPCs into one RPC](#concepts-ga-batching). These types of abilities are commonly used for hitscan guns.

<a name="optimizations-gameplaycuebatching"></a>
### 7.2 Gameplay Cue Batching
If you're sending many [`GameplayCues`](#concepts-gc) at the same time, consider [batching them into one RPC](#concepts-gc-batching). The goal is to reduce the number of RPCs (`GameplayCues` are unreliable NetMulticasts) and send as little data as possible.

<a name="optimizations-ascreplicationmode"></a>
### 7.3 AbilitySystemComponent Replication Mode
By default, the [`ASC`](#concepts-asc) is in [`Full Replication Mode`](#concepts-asc-rm). This will replicate all [`GameplayEffects`](#concepts-ge) to every client (which is fine for a single player game). In a multiplayer game, set the player owned `ASCs` to `Mixed Replication Mode` and AI controlled characters to `Minimal Replication Mode`. This will replicate `GEs` applied on a player character to only replicate to the owner of that character and `GEs` applied on AI controlled characters will never replicate `GEs` to clients. [`GameplayTags`](#concepts-gt) will still replicate and [`GameplayCues`](#concepts-gc) will still be unreliable NetMulticast to all clients, regardless of the `Replication Mode`. This will cut down on network data from `GEs` being replicated when all clients don't need to see them.

<a name="optimizations-attributeproxyreplication"></a>
### 7.4 Attribute Proxy Replication
In large games with many players like Fortnite Battle Royale (FNBR), there will be a lot of [`ASCs`](#concepts-asc) living on always-relevant `PlayerStates` replicating a lot of [`Attributes`](#concepts-a). To optimize this bottleneck, Fortnite disables the `ASC` and its [`AttributeSets`](#concepts-as) from replicating altogether on **simulated player-controlled proxies** in the `PlayerState::ReplicateSubobjects()`. Autonomous proxies and AI controlled `Pawns` still fully replicate according to their [`Replication Mode`](#concepts-asc-rm). Instead of replicating `Attributes` on the `ASC` on the always-relevant `PlayerStates`, FNBR uses a replicated proxy structure on the player's `Pawn`. When `Attributes` change on the server's `ASC`, they are changed on the proxy struct too. The client receives the replicated `Attributes` from the proxy struct and pushes the changes back into its local `ASC`. This allows `Attribute` replication to use the `Pawn`'s relevancy and `NetUpdateFrequency`. This proxy struct also replicates a small white-listed set of `GameplayTags` in a bitmask. This optimization reduces the amount of data over the network and allows us to take advantage of pawn relevancy. AI controlled `Pawns` have their `ASC` on the `Pawn` which already uses its relevancy so this optimization is not needed for them.

> I’m not sure if it is still necessary with other server side optimizations that have been done since then (Replication Graph, etc) and it is not the most maintainable pattern.

*Dave Ratti from Epic's answer to [community questions #3](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89)*

<a name="optimizations-asclazyloading"></a>
### 7.5 ASC Lazy Loading
Fortnite Battle Royale (FNBR) has a lot of damageable `AActors` (trees, buildings, etc) in the world, each with an [`ASC`](#concepts-asc). This can add up in memory cost. FNBR optimizes this by lazily loading `ASCs` only when they're needed (when they first take damage by a player). This reduces overall memory usage since some `AActors` may never be damaged in a match.

**[⬆ Back to Top](#table-of-contents)**

<a name="qol"></a>
## 8. Quality of Life Suggestions

<a name="qol-gameplayeffectcontainers"></a>
### 8.1 Gameplay Effect Containers
[GameplayEffectContainers](#concepts-ge-containers) combine [`GameplayEffectSpecs`](#concepts-ge-spec), [`TargetData`](#concepts-targeting-data), [simple targeting](#concepts-targeting-containers), and related functionality into easy to use structures. These are great for transfering `GameplayEffectSpecs` to projectiles spawned from an ability that will then apply them on collision at a later time.

<a name="qol-asynctasksascdelegates"></a>
### 8.2 Blueprint AsyncTasks to Bind to ASC Delegates
To increase designer-friendly iteration times, especially when designing UMG Widgets for UI, create Blueprint AsyncTasks (in C++) to bind to the common change delegates on the `ASC` directly from your UMG Blueprint graphs. The only caveat is that they must be manually destroyed (like when the widget is destroyed) otherwise they will live in memory forever. The Sample Project includes three Blueprint AsyncTasks.

Listen for `Attribute` changes:

![Listen for Attributes Changes BP Node](https://github.com/tranek/GASDocumentation/raw/master/Images/attributeschange.png)

Listen for cooldown changes:

![Listen for Cooldown Change BP Node](https://github.com/tranek/GASDocumentation/raw/master/Images/cooldownchange.png)

Listen for `GE` stack changes:

![Listen for GameplayEffect Stack Change BP Node](https://github.com/tranek/GASDocumentation/raw/master/Images/gestackchange.png)

**[⬆ Back to Top](#table-of-contents)**

<a name="troubleshooting"></a>
## 9. Troubleshooting

<a name="troubleshooting-notlocal"></a>
### 9.1 `LogAbilitySystem: Warning: Can't activate LocalOnly or LocalPredicted ability %s when not local!`
You need to [initialize the `ASC` on the client](#concepts-asc-setup).

**[⬆ Back to Top](#table-of-contents)**

<a name="troubleshooting-scriptstructcache"></a>
### 9.2 `ScriptStructCache` errors
You need to call [`UAbilitySystemGlobals::InitGlobalData()`](#concepts-asg-initglobaldata).

**[⬆ Back to Top](#table-of-contents)**

<a name="troubleshooting-replicatinganimmontages"></a>
### 9.3 Animation Montages are not replicating to clients
Make sure that you're using the `PlayMontageAndWait` Blueprint node instead of `PlayMontage` in your [GameplayAbilities](#concepts-ga). This [AbilityTask](#concepts-at) replicates the montage through the `ASC` automatically whereas the `PlayMontage` node does not.

**[⬆ Back to Top](#table-of-contents)**

<a name="troubleshooting-duplicatingblueprintactors"></a>
### 9.4 Duplicating Blueprint Actors is setting AttributeSets to nullptr
There is a [bug in Unreal Engine](https://issues.unrealengine.com/issue/UE-81109) that will set `AttributeSet` pointers on your classes to nullptr for Blueprint Actor classes that are duplicated from existing Blueprint Actor classes. There are a few workarounds for this. I've had success not creating bespoke `AttributeSet` pointers on my classes (no pointer in the .h, not calling `CreateDefaultSubobject` in the constructor) and instead just directly adding `AttributeSets` to the `ASC` in `PostInitializeComponents()` (not shown in the Sample Project). The replicated `AttributeSets` will still live in the `ASC's` `SpawnedAttributes` array. It would look something like this:

```c++
void AGDPlayerState::PostInitializeComponents()
{
	Super::PostInitializeComponents();

	if (AbilitySystemComponent)
	{
		AbilitySystemComponent->AddSet<UGDAttributeSetBase>();
		// ... any other AttributeSets that you may have
	}
}
```

In this scenario, you would read and set the values in the `AttributeSet` using the functions on the `ASC` instead of [calling functions on the `AttributeSet` made from the macros](#concepts-as-attributes).

```c++
/** Returns current (final) value of an attribute */
float GetNumericAttribute(const FGameplayAttribute &Attribute) const;

/** Sets the base value of an attribute. Existing active modifiers are NOT cleared and will act upon the new base value. */
void SetNumericAttributeBase(const FGameplayAttribute &Attribute, float NewBaseValue);
```

So the `GetHealth()` would look something like:

```c++
float AGDPlayerState::GetHealth() const
{
	if (AbilitySystemComponent)
	{
		return AbilitySystemComponent->GetNumericAttribute(UGDAttributeSetBase::GetHealthAttribute());
	}

	return 0.0f;
}
```

Setting (initializing) the health `Attribute` would look something like:

```c++
const float NewHealth = 100.0f;
if (AbilitySystemComponent)
{
	AbilitySystemComponent->SetNumericAttributeBase(UGDAttributeSetBase::GetHealthAttribute(), NewHealth);
}
```

As a reminder, the `ASC` only ever expects at most one `AttributeSet` object per `AttributeSet` class.

**[⬆ Back to Top](#table-of-contents)**

<a name="troubleshooting-unresolvedexternalsymbolmarkpropertydirty"></a>
### 9.5 Unresolved external symbol UEPushModelPrivate::MarkPropertyDirty(int,int)

If you get a compiler error like:

```
error LNK2019: unresolved external symbol "__declspec(dllimport) void __cdecl UEPushModelPrivate::MarkPropertyDirty(int,int)" (__imp_?MarkPropertyDirty@UEPushModelPrivate@@YAXHH@Z) referenced in function "public: void __cdecl FFastArraySerializer::IncrementArrayReplicationKey(void)" (?IncrementArrayReplicationKey@FFastArraySerializer@@QEAAXXZ)
```

This is from trying to call `MarkItemDirty()` on a `FFastArraySerializer`. I've encountered this from updating an `ActiveGameplayEffect` such as when updating the cooldown duration.

```c++
ActiveGameplayEffects.MarkItemDirty(*AGE);
```

What's happening is that `WITH_PUSH_MODEL` is getting defined in more than one place. `PushModelMacros.h` is defining it as 0 while it's defined as 1 in multiple places. `PushModel.h` is seeing it as 1 but `PushModel.cpp` is seeing it as 0.

The solution is to add `NetCore` to your project's `PublicDependencyModuleNames` in the `Build.cs`.

**[⬆ Back to Top](#table-of-contents)**

<a name="troubleshooting-enumnamesarenowpathnames"></a>
### 9.6 Enum names are now represented by path name

If you get a compiler warning like:

```
warning C4996: 'FGameplayAbilityInputBinds::FGameplayAbilityInputBinds': Enum names are now represented by path names. Please use a version of FGameplayAbilityInputBinds constructor that accepts FTopLevelAssetPath. Please update your code to the new API before upgrading to the next release, otherwise your project will no longer compile.
```

UE 5.1 deprecated using `FString` in the constructor for `BindAbilityActivationToInputComponent()`. Instead, we must pass in an `FTopLevelAssetPath`.

Old, deprecated way:
```c++
AbilitySystemComponent->BindAbilityActivationToInputComponent(InputComponent, FGameplayAbilityInputBinds(FString("ConfirmTarget"),
	FString("CancelTarget"), FString("EGDAbilityInputID"), static_cast<int32>(EGDAbilityInputID::Confirm), static_cast<int32>(EGDAbilityInputID::Cancel)));
```

New way:
```c++
FTopLevelAssetPath AbilityEnumAssetPath = FTopLevelAssetPath(FName("/Script/GASDocumentation"), FName("EGDAbilityInputID"));
AbilitySystemComponent->BindAbilityActivationToInputComponent(InputComponent, FGameplayAbilityInputBinds(FString("ConfirmTarget"),
	FString("CancelTarget"), AbilityEnumAssetPath, static_cast<int32>(EGDAbilityInputID::Confirm), static_cast<int32>(EGDAbilityInputID::Cancel)));
```

See `Engine\Source\Runtime\CoreUObject\Public\UObject\TopLevelAssetPath.h` for more info.

**[⬆ Back to Top](#table-of-contents)**

<a name="acronyms"></a>
## 10. Common GAS Acronyms

| Name                                                         | Acronyms            |
| ------------------------------------------------------------ | ------------------- |
| AbilitySystemComponent                                       | ASC                 |
| AbilityTask                                                  | AT                  |
| [Action RPG Sample Project by Epic](https://www.unrealengine.com/marketplace/en-US/product/action-rpg) | ARPG, ARPG Sample   |
| CharacterMovementComponent                                   | CMC                 |
| GameplayAbility                                              | GA                  |
| GameplayAbilitySystem                                        | GAS                 |
| GameplayCue                                                  | GC                  |
| GameplayEffect                                               | GE                  |
| GameplayEffectExecutionCalculation                           | ExecCalc, Execution |
| GameplayTag                                                  | Tag, GT             |
| ModifierMagnitudeCalculation                                 | ModMagCalc, MMC     |

**[⬆ Back to Top](#table-of-contents)**

<a name="resources"></a>
## 11. Other Resources
* [Official Documentation](https://docs.unrealengine.com/en-US/Gameplay/GameplayAbilitySystem/index.html)
* Source Code!
   * Especially `GameplayPrediction.h`
* [Lyra Sample Project by Epic](https://unrealengine.com/marketplace/en-US/learn/lyra)
* [Action RPG Sample Project by Epic](https://www.unrealengine.com/marketplace/en-US/product/action-rpg)
* [Unreal Slackers Discord](https://unrealslackers.org/) has a text channel dedicated to GAS `#gameplay-ability-system`
   * Check pinned messages
* [GitHub repository of resources by Dan 'Pan'](https://github.com/Pantong51/GASContent)
* [YouTube Videos by SabreDartStudios](https://www.youtube.com/channel/UCCFUhQ6xQyjXDZ_d6X_H_-A)

<a name="resources-daveratti"></a>
### 11.1 Q&A With Epic Game's Dave Ratti

<a name="resources-daveratti-community1"></a>
#### 11.1.1 Community Questions 1
[Dave Ratti responses to the Unreal Slackers Discord Server community questions about GAS](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89):

1. How can we create scoped prediction windows on demand outside or irrespective of `GameplayAbilities`? For example, how can a fire and forget projectile locally predict a damage `GameplayEffect` when it hits an enemy?

> The PredictionKey system is not really meant to do this. Fundamentally this systems works by a client initiating a predictive action, telling the server about it with a key, and then both client and server running the same thing and associating predictive side effects with the given prediction key. For example, “I am predictively activating an ability” or “I have produced target data and am going to predictively run the part of the ability graph after the WaitTargetData task”.
>
> With this pattern, the PredictionKey “bounces” off the server and comes back to the client via UAbilitySystemComponent::ReplicatedPredictionKeyMap (replicated property). Once the key is replicated back from the server, the client is able to undo all of the locally predictive side effects (GameplayCues, GameplayEffects): the replicated versions *will be there* and if they aren’t then it was a misprediction. Knowing exactly when to undo the predictive side effects is crucial here: if you are too early you will see gaps, if you are too late you will have “double”. (Note this is referring to stateful prediction, like a looping GameplayCue of a duration based Gameplay Effect. “Burst” GameplayCues and instant Gameplay Effects are never “undone” or rolled back. They are just skipped on the client if there is a prediction key associated with them).
>
> To further hit home the point: it’s crucial that predictive action is something the server does not do on their own, but only does so when the client tells them to. So having a generic “Create a key on demand and tell the server so I can run something” does not work unless that “something” is something the server will only do once told to by the client.
>
> Backing up to the original question: something like a fire and forget projectile. Both Paragon and Fornite have projectile actor classes that use GameplayCues. However we do not use the Prediction Key system to do these. Instead we have a concept on Non-Replicated GameplayCues. GameplayCues that just fire off locally and are skipped by the server completely. Essentially all these are direct calls to UGameplayCueManager::HandleGameplayCue. They do not route through the UAbilitySystemComponent so no prediction key checks / early returns are made.
>
> The downside with non replicated GameplayCues is that, well, they are not replicated. So its up to the projectile class/blueprint to make sure the code paths that call these functions are running on everyone. We have for cues startup (called in BeginPlay), explosion, hit wall/character, etc.
>
> These type of events are already generated client side, so calling into a non replicated gameplay cue was no big deal. Complicated blueprints can be tricky, and are up to the author to make sure they understand what is running where.

2. When using a `WaitNetSync` `AbilityTask` with `OnlyServerWait` to create a scoped prediction window in a locally predicted `GameplayAbility`, could players potentially cheat by delaying their packets to the Server to control `GameplayAbility` timing since the Server is waiting for their RPC withtheir prediction key? Was this ever an issue in Paragon or Fortnite, and if so, what did Epic do to remedy it?

> Yes, this is a valid concern. Any ability blueprint running on the server that is waiting for a client “signal” is potentially vulnerable to lag switch type exploits.
>
> Paragon had a custom targeting task similar to UAbilityTask_WaitTargetData. In this task we had timeouts, or a “max delay” that we would wait on the client for instantaneous targeting modes. If the targeting mode was waiting for user confirmation (button press) then it would be ignored since the user is allowed to take his time. But for abilities that instantly confirmed targeting we would only wait a certain amount of time before either A) generating the target data server side or B) canceling the ability.
>
> We never had such mechanisms for WaitNetSync, which we used pretty sparingly.
>
> I don’t believe Fortnite makes use of anything like this though. The weapon abilities in Fortnite are special cased batched to a single fortnite-specific RPC: one RPC to activate the ability, provide target data, and end the ability. So weapon abilities are intrinsically not vulnerable to this in Battle Royale.
>
> My take is that this is something that could probably be solved system wide but I don’t see us making the change ourselves anytime soon. Spot fixing WaitNetSync to include a max delay for the case you mention is probably a reasonable task, but again - unlikely we will do this on our end in the immediate future.


3. Which `EGameplayEffectReplicationMode` did Paragon and Fortnite use and what are Epic’s recommendations for when to use each?

> Both games essentially use Mixed mode for their player controlled characters and Minimal for AI controlled (AI minions, jungle creeps, AI Husks, etc). This is what I would recommend most people using the system in a multiplayer game. The sooner into your project you set these, the better.
>
> Fortnite goes a few steps further with its optimizations. It actually does not replicate the UAbilitySystemComponent at all for simulated proxies. The component and attribute subobjects are skipped inside ::ReplicateSubobjects() on the owning fortnite player state class. We do push the bare minimum replicated data from the ability system component to a structure on the pawn itself (basically, a subset of attribute values and a white list subset of tags that we replicate down in a bitmask). We call this a “proxy”. On the receiving side we take the proxy data, replicated on the pawn, and push it back into ability system component on the player state. So you do have an ASC for each player in FNBR, it just doesn’t directly replicate: instead it replicates data via a minimal proxy struct on the pawn and then routes back to the ASC on receiving side. This is advantage since its A) a more minimal set of data B) takes advantage of pawn relevancy.
>
> I’m not sure if it is still necessary with other server side optimizations that have been done since then (Replication Graph, etc) and it is not the most maintainable pattern.


4. Since we cannot predict the removal of `GameplayEffects` as per `GameplayPrediction.h`, are there any strategies for mitigating the effects of latency on removing `GameplayEffects`? For example, when removing a movement speed slow, we currently have to wait for the Server to replicate the `GameplayEffect` removal resulting in a snap of the player’s character position.

> This is a tough one and I don’t have a good answer. We generally skirted around these problems with tolerances and smoothing. I totally agree that ability system and precise synchronization with the character movement system is not in a good place and something we do want to fix.
>
> I had a shelf of allowing predictive removal of GEs but could never work out all edge cases before having to move on. This doesn’t solve everything though since character movement still has an internal saved move buffer that does not know anything about the ability system and possible movement speed modifiers, etc. It is still possible to get into correction feedback loops even outside of not being able to predict the removal of GEs.
>
> If you think you have a case that is truly desperate, you are able to predictively add a GE that would inhibit your movement speed GEs. I’ve never done this myself but have theorized about it before. It may be able to help with a certain class of problem.


5. We know that the `AbilitySystemComponent` lives on the `PlayerState` in Paragon and Fortnite and on the `Character` in the Action RPG Sample. What are Epic’s internal rules, guidelines, or recommendations for where the AbilitySystemComponent should live, and what should its `Owner` be?

> In general I would say anything that does not need to respawn should have the Owner and Avatar actor be the same thing. Anything like AI enemies, buildings, world props, etc.
>
> Anything that does respawn should have the Owner and Avatar be different so that the Ability System Component does not need to be saved off / recreated / restored after a respawn. PlayerState is the logical choice it is replicated to all clients (where as PlayerController is not). The downside is PlayerStates are always relevant so you can run into problems in 100 player games (See notes on what FN did in question #3).


6. Is it viable to have several `AbilitySystemComponents` which have the same owner but different avatars (e.g. on pawn and weapon/items/projectiles with `Owner` set to `PlayerState`)?

> The first problem I see there would be implementing the IGameplayTagAssetInterface and IAbilitySystemInterface on the owning actor. The former may be possible: just aggregate the tags from all ASCs (but watch out - HasAllMatchingGameplayTags may be met only via cross ASC aggregation. It wouldn't be enough to just forward that calls to each ASC and OR the results together). But the later is even trickier: which ASC is the authoritative one? If someone wants to apply a GE - which one should receive it? Maybe you can work these out but this side of the problem will be the hardest: owners will multiple ASCs beneath them.
>
> Separate ASCs on the pawn and the weapon can make sense on its own though. E.g, distinguishing between tags the describe the weapon vs those that describe the owning pawn. Maybe it does make sense that tags granted to the weapon also “apply” to the owner and nothing else (E.g, attributes and GEs are independent but the owner will aggregate the owned tags like I describe above). This could work out, I am sure. But having multiple ASCs with the same owner may get dicey.


7. Is there a way to stop the Server from overwriting the cooldown duration of locally predicted abilities on the Owning Client? In scenarios of high latency, this would let the Owning Client "try" to activate the ability again when its local cooldown expires but it is still on cooldown on the Server. By the time the Owning Client's activation request reaches the Server over the network, the Server may be off cooldown or the Server might be able to queue the activation request for the remaining milliseconds that it has left. Otherwise as is, clients with higher latency have a longer delay before when they can reactivate an ability versus those with less latency. This is most apparent with very low cooldown abilities like a basic attack that can be less than one second of cooldown. If there isn't a way to stop the Server from overwriting the cooldown duration of locally predicted abilities, what is Epic's strategy for mitigating the effects of high latency on reactivating abilities? To word it another example-based way, how did Epic design Paragon's basic attacks and other abilities so that high latency players could attack or activate at the same speed as low latency players with local prediction?

> The short answer there is not a way to prevent this and Paragon definitely had the problem. Higher latency connections would have a lower ROF with basic attacks.
>
> I attempted to fix this by adding “GE reconciliation” where latency was taken into account when calculating GE duration. Essentially allowing the server to eat some of the total GE time so that the effective time of the GE client side would be 100% consistent with any amount of latency (though fluctuations could still cause issues). However I never got this working in a state that could ship and the project moved fast and we just never fully addressed it.
>
> Fortnite does its own bookkeeping for weapon firing rates: it does not use GEs for cooldowns on weapons. I would recommend this if this is a critical problem for your game.


8. What is Epic’s roadmap for the GameplayAbilitySystem plugin? Which features does Epic plan to add in 2019 and beyond?

> We feel that overall the system is pretty stable at this point and we don’t have anyone working on major new features. Bug fixes and small improvements occasionally are made for Fortnite or from UDN/pull requests, but that is it right now.
>
> Longer term, I think we will eventually do a “V2” or some big changes. We learned a lot from writing this system and feel we got a lot right and a lot wrong. I would love a chance to correct those mistakes and improve some of the fatal flaws that were pointed out above.
>
> If a V2 was to ever come, providing an upgrade path would be of utmost importance. We would never make a V2 and leave Fortnite on V1 forever: there would be some path or procedures that would automatically migrate as much as possible, though there would still almost certainly be some manual remaking required.
>
> The high priority fixes would be:
> * Better interoperability with the character movement system. Unifying client prediction.
> * GE removal prediction (question #4)
> * GE latency reconciliation (question #7)
> * Generalized network optimizations such as batching RPCs and proxy structures. Mostly the stuff that we’ve done for Fortnite but find ways to break it down into more generalized form, at least so that games can write their own game specific optimizations more easily.
>
> The more general refactor type of changes I would consider making:
> * I would like to look at fundamentally moving away from having GEs reference spreadsheet values directly, instead they would be able to emit parameters and those parameters could be filled by some higher level object that is bound to spreadsheet values. The problem with the current model is that GEs become unsharable due to their tight coupling with the curve table rows. I think a generalized system for parameterization could be written and be the underpinning of a V2 system.
> * Reduce number of “policies” on UGameplayAbility. I would remove ReplicationPolicy and InstancingPolicy. Replication is, imo, almost never actually needed and causes confusion. InstancingPolicy should be replaced instead by making FGameplayAbilitySpec a UObject that can be subclassed. This should have been the “non instantiated ability object” that has events and is blueprintable. The UGameplayAbility should be the “instanced per execution” object. It could be optional if you need to actually instantiate: instead “non instanced” abilities would be implemented via the new UGameplayAbilitySpec object. 
> * The system should provide more “middle level” constructs such as “filtered GE application container” (data drive what GEs to apply to which actors with higher level gameplay logic), “Overlapping volume support” (apply the “Filtered GE application container” based on collision primitive overlap events), etc. These are building blocks that every project ends up implementing in their own way. Getting them right is non trivial so I think we should do a better job providing some basic implementations. 
> * In general, reducing boilerplate needed to get your project up and running. Possibly a separate module “Ex library” or whatever that could provide things like passive abilities or basic hitscan weapons out of the box. This module would be optional but would get you up and running quickly.
> * I would like to move GameplayCues to a separate module that is not coupled with the ability system. I think there are a lot of improvements that could be made here.


> This is only my personal opinion and not a commitment from anyone. I think the most realistic course of action will be as new engine tech initiatives come through, the ability system will need to be updated and that will be a time to do this sort of thing. These initiatives could be related to scripting, networking, or physics/character movement. This is all very far looking ahead though so I cannot give commitments or estimates on timelines.

**[⬆ Back to Top](#table-of-contents)**

<a name="resources-daveratti-community2"></a>
#### 11.1.2 Community Questions 2
Community member [iniside](https://github.com/iniside)'s Q&A with Dave Ratti:

1. Is the support for decoupled fixed ticking planned? I'd like to
have Game Thread be fixed (like 30/60fps) and let the rendering thread
run wild. I ask if this is something we should expect in future or
not, to make some assumptions about how gameplay should work.
I ask mainly because there is now a fixed async tick for physics and
this poses a question how the rest of the system might work in the
future. I do not hide that having the ability to have fixed tick game
thread without also fixing tick rate of the rest of the engine would
be beyond awesome.

> There are no plans to decouple rendering frame rate and game thread tick frame rate. I think the ship has sailed on this ever happening due to the complexity of these systems and the requirement to preserve backwards compatibility with previous engine versions.
>
> Instead, the direction we've gone is to have an asynchronous "Physics Thread" which runs at a fixed tick rate, independent of the game thread. Things that need to run at a fixed rate can run here and the game thread / rendering can operate how they always have.
>
> It's worth clarifying that Network Prediction supports what it calls Independent Ticking and Fixed Ticking modes. My long term plan is to keep Independent Ticking roughly how it is today in Network Prediction where it runs on the game thread at variable frame rate and there is no "group/world" prediction, it's just the classic "clients predict their own pawn and owned actors" model. And Fixed Ticking would be what uses the async physics stuff and allows you to predict non client controlled/owned actors like physics objects and other clients/pawns/vehicles/etc.


2. Is there any plan on how the integration of Network Prediction will
look with the Ability System? Like for example, fixed frame ability
activation (so the server gets frames in which abilities were
activated and tasks executed instead of prediction keys)?

> Yes, the plan is to rewrite/remove the Ability System's prediction keys and replace them with Network Prediction constructs. The MockAbility examples in NetworkPredictionExtras show how this might work but they are more "hard coded" than what GAS will require. 
>
> The main idea would be that we remove the explicit client->server Prediction Key exchange in the ASC's RPCs. There would no longer be prediction windows or scoped prediction keys. Instead everything would be anchored around NetworkPrediction frames. The important thing is that client and server agree on when things happen. Examples would be:
>
> * When abilities were activated/ended/cancelled
> * When Gameplay Effects were applied/removed
> * Attribute values (what an attributes value was at frame X)
>
> I think this could be done generically at the ability system level. But actually making the user-defined logic inside a UGameplayAbility completely rollback-able would still take more work. We may end up having a subclass of UGameplayAbility that is fully rollbackable and has access to a more limited set of functionality or only Ability Tasks that are marked as rollback-friendly. Something like that. There are also many implications to animation events and root motion and how those are processed.
>
> Wish I had a more clear answer but it's really important we get the foundation right before touching GAS again. Movement and physics have to be solid before the higher level systems can be changed.


3. Is there a plan to move Network Prediction development toward the
main branch? Not gonna lie, I'd really like to check the latest code.
Regardless of it's state.

> We are working towards it. The system work is still all being done in NetworkPrediction (see NetworkPhysics.h) and the underlying async physics stuff should be all available (RewindData.h etc). But we also have use cases in Fortnite that we have been focused on that obviously can't be made public. We are working through bugs, performance optimizations, etc.
>
> For more context: when working on the early versions of this system, we were very focused on the "front end" of things - how state and simulations were defined and written. We learned a lot there. But as the async physics stuff has come online, we've been much more focused on just getting something real to work in this system, at the expense of throwing out some of our early abstractions. The goal here is to circle back when the real thing is working and reunifying things. E.g, get back to the "front end" and make the final version of that on top of the core pieces of tech we are working on now.


4. For some time on main branch there was a plugin for sending Gameplay
Messages (Looked like Event/Message Bus), but it was removed. Any
plans to restore it? With the Game Features/Modular Gameplay plugins,
having a generic Event Bus Dispatcher would be extremely useful.

> I think you are referring to the GameplayMessages plugin. This will probably come back at some point - the API isn't really finalized yet and the author didn't mean for it to be public yet. I agree it should be useful for modular gameplay design. But it's not really my area so I don't have much more information. 


5. I've been playing recently with async fixed physics and the results
are promising, though if there is going to be NP update in the future
I will probably just play around and wait, since to get it working I
still need to get entire engine into fixed tick and on the other hand
I try to keep physics at 33ms. Which does not make for a good
experience if everything is at 30 fps (:.

I have noticed there was some work on Async
CharacterMovementComponent, but not sure if this will be using Network
Prediction, or it is a separate effort?

Since I noticed it, I also went ahead and tried to implement my custom
async movement at fixed tick rate, which worked okay, but on top of it
I also needed to add a separate update for interpolation. The setup
was to run simulation tick on separate worker threads at fixed 33ms
update, do calculations, save result, and interpolate it at the game
thread to match current frame rate. Not perfect, but it got the job
done.

My question is, if this is something that might be easier to set up in
the future, as there is just quite a bit of boilerplate code to write,
(the interpolation part) and it's not particularly efficient to
interpolate each moving object individually.

The async stuff is really interesting, because it would allow you to
really run game simulation at fixed update rate (which would make
fixed thread unneeded) and have more predictable results. Is this
something that is intended going forward, or more of a benefit to
select systems? As far as I remember actor transforms are not updated
async and blueprints are not entirely thread safe. In other words is
it something that is planned to be supported at more of a framework
level or something that each game has to solve on it's own?

> Async CharacterMovementComponent
>
> This is basically an early prototype/experiment of porting CMC as it is to the physics thread. I don't view it as the future of CMC yet, but it could evolve into that. Right now there is no networking support so it's not something I would really follow. The people doing it are mostly concerned with measuring input latency that this system would add and how that could be mitigated.
>
> I still need to get entire engine into fixed tick and on the other hand I try to keep physics at 33ms. Which does not make for a good experience if everything is at 30 fps (:.
>
> The async stuff is really interesting, because it would allow you to really run game simulation at fixed update rate (which would make fixed thread unneeded) 
>
> Yes. The goal here is that with async physics enabled, you can run the engine at variable tick rate while the physics and "core" gameplay simulations can run at the fixed rate (such as character movement, vehicles, GAS, etc).
>
> These are the cvars that need to be set to enable this now: (I think you've figured this out)  
> `p.DefaultAsyncDt=0.03333`  
> `p.RewindCaptureNumFrames=64`
>
> Chaos does provide interpolation for the physics state (e.g, the transforms that get pushed back to the UPrimitiveComponent and are visible to the game code). There is a cvar now, `p.AsyncInterpolationMultiplier`, which controls that if you want to look at it. You should see smooth continuous motion of physics bodies without having to write any extra code. 
>
> If you want to interpolate non physics state, it is still up to you to do that right now. The example would be like a cool-down that you want to update (tick) on the async physics thread but see smooth continuous interpolation on the game thread so that every render frame the cool-down visualization is updated. We will get to this eventually but don't have examples yet.
>
> there is just quite a bit of boilerplate code to write,
>
> Yeah, so that has been a big general problem with the system up until now. We want to provide an interface that experienced programmers can use to maximize performance and safety (the ability to write gameplay code that "just works" predictively without tons of hazards and things you could-do-but-better-not). So something like CharacterMoverment might do a bunch of custom stuff to maximize its performance - e.g, writing templated code and doing batch updating, going wide, breaking the update loop into distinct phases etc. We want to provide a good "low level" interface into the async thread and rollback systems for this use case. And in this case too - it's still reasonable that the character movement system itself is extendable in its own way. For example providing a way to blueprint a custom movement mode and providing a blueprint API that is thread safe.
>
> But we recognize this is not acceptable for simpler gameplay objects that don't really need their own "system". Something more inline with Unreal is what is needed. E.g, using the reflection system, having general blueprint support, etc. There are examples of blueprints being used on other threads (see BlueprintThreadSafe keyword and what the animation system has been working towards). So I think there will be some form of this one day. But again, we aren't there yet.
>
> I realize you were just asking about interpolation but that is the general answer: right now we have you do everything manually like NetSerialize, ShouldReconcile, Interpolate, etc but eventually we'll have a way that is like "if you want to just use the reflection system, you don't have to manually write this stuff". We just don't want to *force* everyone to use the reflection system since that imposes other limitations that we think we don't want to take on the lowest levels of the system. 
>
> And then just to tie this back to what I said earlier - right now we are really focused on getting a few very specific examples working and performant and then we will turn attention back to the front end and making things friendly to use and iterate on, reducing boilerplate, etc for everybody else to use. 

**[⬆ Back to Top](#table-of-contents)**

<a name="changelog"></a>
## 12. GAS Changelog

This is a list of notable changes (fixes, changes, and new features) to GAS compiled from the official Unreal Engine upgrade changelog and from undocumented changes that I've encountered. If you've found something that isn't listed here, please make an issue or pull request.

<a name="changelog-5.3"></a>
### 5.3
* Crash Fix: Fixed a crash when trying to apply Gameplay Cues after a seamless travel.
* Crash Fix: Fixed a crash caused by GlobalAbilityTaskCount when using Live Coding.
* Crash Fix: Fixed UAbilityTask::OnDestroy to not crash if called recursively for cases like UAbilityTask_StartAbilityState.
* Bug Fix: It is now safe to call `Super::ActivateAbility` in a child class. Previously, it would call `CommitAbility`.
* Bug Fix: Added support for properly replicating different types of FGameplayEffectContext.
* Bug Fix: FGameplayEffectContextHandle will now check if data is valid before retrieving "Actors".
* Bug Fix: Retain rotation for Gameplay Ability System Target Data LocationInfo.
* Bug Fix: Gameplay Ability System now stops searching for PC only if a valid PC is found.
* Bug Fix: Use existing GameplayCueParameters if it exists instead of default parameters object in RemoveGameplayCue_Internal.
* Bug Fix: GameplayAbilityWorldReticle now faces towards the source Actor instead of the TargetingActor.
* Bug Fix: Cache trigger event data if it was passed in with GiveAbilityAndActivateOnce and the ability list was locked.
* Bug Fix: Support has been added for the FInheritedGameplayTags to update its CombinedTags immediately rather than waiting until a Save.
* Bug Fix: Moved ShouldAbilityRespondToEvent from client-only code path to both server and client.
* Bug Fix: Fixed FAttributeSetInitterDiscreteLevels from not working in Cooked Builds due to Curve Simplification.
* Bug Fix: Set CurrentEventData in GameplayAbility.
* Bug Fix: Ensure MinimalReplicationTags are set up correctly before potentially executing callbacks.
* Bug Fix: Fixed ShouldAbilityRespondToEvent from not getting called on the instanced GameplayAbility.
* Bug Fix: Gameplay Cue Notify Actors executing on Child Actors no longer leak memory when gc.PendingKill is disabled.
* Bug Fix: Fixed an issue in GameplayCueManager where GameplayCueNotify_Actors could be 'lost' due to hash collisions.
* Bug Fix: WaitGameplayTagQuery will now respect its Query even if we have no Gameplay Tags on the Actor.
* Bug Fix: PostAttributeChange and AttributeValueChangeDelegates will now have the correct OldValue.
* Bug Fix: Fixed FGameplayTagQuery from not showing a proper Query Description if the struct was created by native code.
* Bug Fix: Ensure that the UAbilitySystemGlobals::InitGlobalData is called if the Ability System is in use. Previously if the user did not call it, the Gameplay Ability System did not function correctly.
* Bug Fix: Fixed issue when linking/unlinking anim layers from UGameplayAbility::EndAbility.
* Bug Fix: Updated Ability System Component function to check the Spec's ability pointer before use.
* New: Added a GameplayTagQuery field to FGameplayTagRequirements to enable more complex requirements to be specified.
* New: Introduced FGameplayEffectQuery::SourceAggregateTagQuery to augment SourceTagQuery.
* New: Extended the functonality to execute and cancel Gameplay Abilities & Gameplay Effects from a console command.
* New: Added the ability to perform an "Audit" on Gameplay Ability Blueprints that will show information on how they're developed and intended to be used.
* Change: OnAvatarSet is now called on the primary instance instead of the CDO for instanced per Actor Gameplay Abilities.
* Change: Allow both Activate Ability and Activate Ability From Event in the same Gameplay Ability Graph.
* Change: AnimTask_PlayMontageAndWait now has a toggle to allow Completed and Interrupted after a BlendOut event.
* Change: ModMagnitudeCalc wrapper functions have been declared const.
* Change: FGameplayTagQuery::Matches now returns false for empty queries.
* Change: Updated FGameplayAttribute::PostSerialize to mark the contained attribute as a searchable name.
* Change: Updated GetAbilitySystemComponent to default parameter to Self.
* Change: Marked functions as virtual in AbilityTask_WaitTargetData.
* Change: Removed unused function FGameplayAbilityTargetData::AddTargetDataToGameplayCueParameters.
* Change: Removed vestigial GameplayAbility::SetMovementSyncPoint.
* Change: Removed unused replication flag from Gameplay tasks & Ability system components.
* Change: Moved some gameplay effect functionality into optional components. All existing content will automatically update to use components during PostCDOCompiled, if necessary.

https://docs.unrealengine.com/5.3/en-US/unreal-engine-5.3-release-notes/

<a name="changelog-5.2"></a>
### 5.2
* Bug Fix: Fixed a crash in the `UAbilitySystemBlueprintLibrary::MakeSpecHandle` function.
* Bug Fix: Fixed logic in the Gameplay Ability System where a non-Controlled Pawn would be considered remote, even if it was spawned locally on the server (e.g. Vehicles).
* Bug Fix: Correctly set activation info on predicted instanced abilities that were rejected by the server.
* Bug Fix: Fixed a bug that would cause GameplayCues to get stuck on remote instances.
* Bug Fix: Fixed a memory stomp when chaining calls to WaitGameplayEvent.
* Bug Fix: Calling the AbilitySystemComponent `GetOwnedGameplayTags()` function in Blueprint no longer retains the previous call's return values when the same node is executed multiple times.
* Bug Fix: Fixed an issue with GameplayEffectContext replicating a reference to a dynamic object that would never be replicated.
  * This prevented GameplayEffect from calling `Owner->HandleDeferredGameplayCues(this)` as `bHasMoreUnmappedReferences` would always be true.
* New: The [Gameplay Targeting System](https://docs.unrealengine.com/en-US/gameplay-targeting-system-in-unreal-engine/) is a way to create data-driven targeting requests.
* New: Added custom serialization support for GameplayTag Queries.
* New: Added support for replicating derived FGameplayEffectContext types.
* New: Gameplay Attributes in assets are now registered as searchable names on save, allowing for references to attributes to be seen in the reference viewer.
* New: Added some basic unit tests for the AbilitySystemComponent.
* New: Gameplay Ability System Attributes now respect Core Redirects. This means you can now rename Attribute Sets and their Attributes in code and have them load properly in assets saved with the old names by adding redirect entries to DefaultEngine.ini.
* Change: Allow changing the evaluation channel of a Gameplay Effect Modifier from code.
* Change: Removed previously unused variable `FGameplayModifierInfo::Magnitude` from the Gameplay Abilities Plugin.
* Change: Removed the synchronization logic between the ability system component and Smart Object instance tags.

https://docs.unrealengine.com/5.2/en-US/unreal-engine-5.2-release-notes/

<a name="changelog-5.1"></a>
### 5.1
* Bug Fix: Fixed issue where replicated loose gameplay tags were not replicating to the owner.
* Bug Fix: Fixed AbilityTask bug where abilities could be blocked from timely garbage-collection.
* Bug Fix: Fixed an issue when a gameplay ability listening to activate based on a tag would fail to be activated. This would happen if there were more than one Gameplay Ability listening to this tag, and the first one in the list was invalid or didn't have authority to activate.
* Bug Fix: Fixed GameplayEffects that use Data Registries correctly from warning on load and improved the warning text.
* Bug Fix: Removed code from UGameplayAbility that was incorrectly only registering the last instanced ability with the Blueprint debugger for breakpoints.
* Bug Fix: Fixed Gameplay Ability System Ability getting stuck if EndAbility was called during the lock inside ApplyGameplayEffectSpecToTarget.
* New: Added support for Gameplay Effects to add blocked ability tags.
* New: Added WaitGameplayTagQuery nodes. One is based off of the UAbilityTask and the other is of UAbilityAsync. This node specifies a TagQuery, and will trigger its output pin when the query becomes true or false, based on configuration.
* New: Modified AbilityTask debugging in Console Variables to enable debug recording and printing to log by default in non-shipping builds (with ability to hotfix on/off as needed).
* New: You can now set AbilitySystem.AbilityTask.Debug.RecordingEnabled to 0 to disable, 1 to enable in non-shipping builds, and 2 to enable all builds (including shipping).
* New: You can use AbilitySystem.AbilityTask.Debug.AbilityTaskDebugPrintTopNResults to only print the top N results in log (to avoid log spam).
* New: STAT_AbilityTaskDebugRecording can be used to test perf impact from these on-by-default debugging changes.
* New: Added a debug command to filter GameplayCue events.
* New: Added new debug commandsAbilitySystem.DebugAbilityTags, AbilitySystem.DebugBlockedTags, andAbilitySystem.DebugAttribute to the Gameplay Ability System.
* New: Added a Blueprint function to get a debug string representation of a Gameplay Attribute.
* New: Added a new Gameplay Task resource overlap policy to cancel existing tasks.
* Change: Now Ability Tasks should make sure to call Super::OnDestroy only after they do anything needed to the Ability pointer, as it will be nulled out after calling it.
* Change: Converted FGameplayAbilitySpec/Def::SourceObject to be a weak reference.
* Change: Made a Ability System Component reference in the Ability Task a weak pointer so Garbage Collection can delete it.
* Change: Removed redundant enum EWaitGameplayTagQueryAsyncTriggerCondition.
* Change: GameplayTasksComponent and AbilitySystemComponent now support the registered subobject API.
* Change: Added better logging to indicate why Gameplay Abilities failed to be activated.
* Change: Removed AbilitySystem.Debug.NextTarget and PrevTarget commands in favor of global HUD NextDebugTarget and PrevDebugTarget commands.

https://docs.unrealengine.com/5.1/en-US/unreal-engine-5.1-release-notes/

<a name="changelog-5.0"></a>
### 5.0

https://docs.unrealengine.com/5.0/en-US/unreal-engine-5.0-release-notes/

<a name="changelog-4.27"></a>
### 4.27
* Crash Fix: Fixed a root motion source issue where a networked client could crash when an Actor finishes executing an ability that uses a constant force root motion task with a strength-over-time modifier.
* Bug Fix: Fixed a regression in Editor loading time when using GameplayCues.
* Bug Fix: GameplayEffectsContainer's `SetActiveGameplayEffectLevel` method will no longer dirty FastArray if setting the same EffectLevel.
* Bug Fix: Fixed an edge case in GameplayEffect mixed replication mode where Actors not explicitly owned by the net connection but who utilize that connection from `GetNetConnection` will not received mixed replication updates.
* Bug Fix: Fixed an endless recursion occuring in GameplayAbility's class method `EndAbility` which was called by calling `EndAbility` again from `K2_OnEndAbility`.
* Bug Fix: GameplayTags Blueprint pins will no longer be silently cleared if they are loaded before tags are registered. They now work the same as GameplayTag variables, and the behavior for both can be changed with the ClearInvalidTags option in the Project Settings.
* Bug Fix: Improved thread safety of GameplayTag operations.
* New: Exposed SourceObject to GameplayAbility's `K2_CanActivateAbility` method.
* New: Native GameplayTags. Introducing a new `FNativeGameplayTag`, these make it possible to do one off native tags that are correctly registered and unregistered when the module is loaded and unloaded.
* New: Updated `GiveAbilityAndActivateOnce` to pass in FGameplayEventData parameter.
* New: Improved ScalableFloats in the GameplayAbilities plugin to support dynamic lookup of curve tables from the new Data Registry System. Added a ScalableFloat header for easier reuse of the generic struct outside the abilities plugin.
* New: Added code support for using the GameplayTag UI in other Editor customizations via GameplayTagsEditorModule.
* New: Modified UGameplayAbility's PreActivate method to optionally take in trigger event data.
* New: Added more support to filter GameplayTags in the Editor using a project-specific filter. `OnFilterGameplayTag` supplies the referencing property and the tag source, so you can filter tags based on what asset is requesting the tag.
* New: Added option to preserve the original captured SourceTags when GameplayEffectSpec's class method `SetContext` is called after initialization.
* New: Improved UI for registering GameplayTags from specific plugins. The new tag UI now lets you select a plugin location on disk for newly added GameplayTag sources.
* New: A new track has been added to Sequencer to allow for triggering notify states on Actors built using the GameplayAbiltiySystem. Like notifies, the GameplayCueTrack can utilize range-based events or trigger-based events.
* Change: Changed the GameplayCueInterface to pass GameplayCueParameters struct by reference.
* Optimization: Made several performance improvements to loading and regenerating the GameplayTag table were implemented so that this option would be optimized.

https://docs.unrealengine.com/en-US/WhatsNew/Builds/ReleaseNotes/4_27/

<a name="changelog-4.26"></a>
### 4.26
* GAS plugin is no longer flagged as beta.
* Crash Fix: Fixed a crash when adding a gameplay tag without a valid tag source selection.
* Crash Fix: Added the path string arg to a message to fix a crash in UGameplayCueManager::VerifyNotifyAssetIsInValidPath.
* Crash Fix: Fixed an access violation crash in AbilitySystemComponent_Abilities when using a ptr without checking it.
* Bug Fix: Fixed a bug where stacking GEs that did not reset the duration on additional instances of the effect being applied.
* Bug Fix: Fixed an issue that caused CancelAllAbilities to only cancel non-instanced abilities.
* New: Added optional tag parameters to gameplay ability commit functions.
* New: Added StartTimeSeconds to PlayMontageAndWait ability task and improved comments.
* New: Added tag container "DynamicAbilityTags" to FGameplayAbilitySpec. These are optional ability tags that are replicated with the spec. They are also captured as source tags by applied gameplay effects.
* New: GameplayAbility IsLocallyControlled and HasAuthority functions are now callable from Blueprint.
* New: Visual logger will now only collect and store info about instant GEs if we're currently recording visual logging data.
* New: Added support for redirectors on gameplay attribute pins in blueprint nodes.
* New: Added new functionality for when root motion movement related ability tasks end they will return the movement component's movement mode to the movement mode it was in before the task started.

https://docs.unrealengine.com/en-US/WhatsNew/Builds/ReleaseNotes/4_26/

<a name="changelog-4.25.1"></a>
### 4.25.1
* Fixed! UE-92787 Crash saving blueprint with a Get Float Attribute node and the attribute pin is set inline
* Fixed! UE-92810 Crash spawning actor with instance editable gameplay tag property that was changed inline

<a name="changelog-4.25"></a>
### 4.25
* Fixed prediction of `RootMotionSource` `AbilityTasks`
* [`GAMEPLAYATTRIBUTE_REPNOTIFY()`](#concepts-as-attributes) now additionally takes in the old `Attribute` value. We must supply that as the optional parameter to our `OnRep` functions. Previously, it was reading the attribute value to try to get the old value. However, if called from a replication function, the old value had already been discarded before reaching SetBaseAttributeValueFromReplication so we'd get the new value instead.
* Added [`NetSecurityPolicy`](#concepts-ga-netsecuritypolicy) to `UGameplayAbility`.
* Crash Fix: Fixed a crash when adding a gameplay tag without a valid tag source selection.
* Crash Fix: Removed a few ways for attackers to crash a server through the ability system.
* Crash Fix: We now make sure we have a GameplayEffect definition before checking tag requirements.
* Bug Fix: Fixed an issue with gameplay tag categories not applying to function parameters in Blueprints if they were part of a function terminator node.
* Bug Fix: Fixed an issue with gameplay effects' tags not being replicated with multiple viewports.
* Bug Fix: Fixed a bug where a gameplay ability spec could be invalidated by the InternalTryActivateAbility function while looping through triggered abilities.
* Bug Fix: Changed how we handle updating gameplay tags inside of tag count containers. When deferring the update of parent tags while removing gameplay tags, we will now call the change-related delegates after the parent tags have updated. This ensures that the tag table is in a consistent state when the delegates broadcast.
* Bug Fix: We now make a copy of the spawned target actor array before iterating over it inside when confirming targets because some callbacks may modify the array.
* Bug Fix: Fixed a bug where stacking GameplayEffects that did not reset the duration on additional instances of the effect being applied and with set by caller durations would only have the duration correctly set for the first instance on the stack. All other GE specs in the stack would have a duration of 1 second. Added automation tests to detect this case.
* Bug Fix: Fixed a bug that could occur if handling gameplay event delegates modified the list of gameplay event delegates.
* Bug Fix: Fixed a bug causing GiveAbilityAndActivateOnce to behave inconsistently.
* Bug Fix: Reordered some operations inside FGameplayEffectSpec::Initialize to deal with a potential ordering dependency.
* New: UGameplayAbility now has an OnRemoveAbility function. It follows the same pattern as OnGiveAbility and is only called on the primary instance of the ability or the class default object.
* New: When displaying blocked ability tags, the debug text now includes the total number of blocked tags.
* New: Renamed UAbilitySystemComponent::InternalServerTryActiveAbility to UAbilitySystemComponent::InternalServerTryActivateAbility.Code that was calling InternalServerTryActiveAbility should now call InternalServerTryActivateAbility.
* New: Continue to use the filter text for displaying gameplay tags when a tag is added or deleted. The previous behavior cleared the filter.
* New: Don't reset the tag source when we add a new tag in the editor.
* New: Added the ability to query an ability system component for all active gameplay effects that have a specified set of tags. The new function is called GetActiveEffectsWithAllTags and can be accessed through code or blueprints.
* New: When root motion movement related ability tasks end they now return the movement component's movement mode to the movement mode it was in before the task started.
* New: Made SpawnedAttributes transient so it won't save data that can become stale and incorrect. Added null checks to prevent any currently saved stale data from propagating. This prevents problems related to bad data getting stored in SpawnedAttributes.
* API Change: AddDefaultSubobjectSet has been deprecated. AddAttributeSetSubobject should be used instead.
* New: Gameplay Abilities can now specify the Anim Instance on which to play a montage.

https://docs.unrealengine.com/en-US/WhatsNew/Builds/ReleaseNotes/4_25/

<a name="changelog-4.24"></a>
### 4.24
* Fixed blueprint node `Attribute` variables resetting to `None` on compile.
* Need to call [`UAbilitySystemGlobals::InitGlobalData()`](#concepts-asg-initglobaldata) to use [`TargetData`](#concepts-targeting-data) otherwise you will get `ScriptStructCache` errors and clients will be disconnected from the server. My advice is to always call this in every project now whereas before 4.24 it was optional.
* Fixed crash when copying a `GameplayTag` setter to a blueprint that didn't have the variable previously defined.
* `UGameplayAbility::MontageStop()` function now properly uses the `OverrideBlendOutTime` parameter.
* Fixed `GameplayTag` query variables on components not being modified when edited.
* Added the ability for `GameplayEffectExecutionCalculations` to support scoped modifiers against "temporary variables" that aren't required to be backed by an attribute capture.
  * Implementation basically enables `GameplayTag`-identified aggregators to be created as a means for an execution to expose a temporary value to be manipulated with scoped modifiers; you can now build formulas that want manipulatable values that don't need to be captured from a source or target.
  * To use, an execution has to add a tag to the new member variable `ValidTransientAggregatorIdentifiers`; those tags will show up in the calculation modifier array of scoped mods at the bottom, marked as temporary variables—with updated details customizations accordingly to support feature
* Added restricted tag quality-of-life improvements. Removed the default option for restricted `GameplayTag` source. We no longer reset the source when adding restricted tags to make it easier to add several in a row. 
* `APawn::PossessedBy()` now sets the owner of the `Pawn` to the new `Controller`. Useful because [Mixed Replication Mode](#concepts-asc-rm) expects the owner of the `Pawn` to be the `Controller` if the `ASC` lives on the `Pawn`.
* Fixed bug with POD (Plain Old Data) in `FAttributeSetInitterDiscreteLevels`.

https://docs.unrealengine.com/en-US/WhatsNew/Builds/ReleaseNotes/4_24/

**[⬆ Back to Top](#table-of-contents)**