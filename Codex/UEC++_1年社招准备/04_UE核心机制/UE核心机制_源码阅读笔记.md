# UE 核心机制源码阅读笔记：UE 5.7

## 使用方式
- 这份文档是 UE C++ 机制的长期源码阅读索引，不是 API 功能清单。
- 每个机制按“问题 -> 源码入口 -> 证据 -> 结论 -> 项目用法 -> 面试表达”的方式阅读。
- 源码以本机 `D:\soft\UE_5.7\Engine` 为准，具体 API、签名、宏、模块边界以本机源码为最终依据。
- 不要求一次读完整个源码文件。先按问题定位，再读相关函数、注释、宏、断言和调用链。
- 读完一个机制后，至少留下三句话：这个机制解决什么问题、项目中怎么用、面试时怎么讲。

## 源码阅读读什么
- 读声明：类继承关系、`UCLASS`、`UPROPERTY`、`UFUNCTION`、`*_API`、访问权限。
- 读生命周期：构造、初始化、注册、BeginPlay、Tick、销毁、反注册等入口。
- 读边界：哪些函数只能运行时用，哪些只能编辑器用，哪些不能在构造函数里用。
- 读参数和返回值：默认参数、`Outer`、`Name`、`Flags`、`SpawnParameters`、`WorldContextObject` 等。
- 读注释和断言：注释说明设计意图，`check` / `ensure` / `AssertIf...` 往往说明禁止用法。
- 读调用链：这个函数由谁触发、触发后又调用谁、Blueprint 事件和 C++ virtual 函数如何衔接。
- 读模块边界：文件在 `Runtime`、`Editor`、`Developer` 还是 `Plugins`，对应是否能进打包游戏。

## 问题驱动阅读流程
1. 先写问题：不要写“读 `Actor.h`”，要写“`BeginPlay` 是谁调用的，和构造函数有什么边界？”
2. 找入口：用 `rg` 搜类名、函数名、宏名，只打开相关区域。
3. 抓证据：记录文件路径、行号、函数签名、关键注释、断言或调用链。
4. 得结论：用自己的话解释机制的职责、边界和常见坑。
5. 对项目：写一句“我在项目里什么时候用，什么时候不用”。
6. 对面试：写一句可以直接说出口的表达。

## 阅读模板
```md
### 阅读问题
- 这个机制解决 UE 的什么问题？
- 它由谁创建、谁持有、谁调用、什么时候结束？
- 哪些场景能用，哪些场景不能用？

### 源码证据
- 文件：
- 搜索词：
- 关键签名：
- 关键注释/断言：
- 调用链：

### 机制结论
- 职责：
- 边界：
- 常见坑：

### 项目用法
- 我会在什么场景使用：
- 我会避免什么写法：

### 面试表达
- 一句话版本：
- 展开版本：
```

## 和直接看功能总结的区别
- 功能总结告诉你“这个类能做什么”，源码阅读告诉你“这个类为什么这样设计、什么时候能用、什么时候不能用”。
- 功能总结容易停在结论，源码阅读能给出证据：函数签名、宏、注释、断言、调用顺序。
- 功能总结适合快速入门，源码阅读适合解决真实项目问题、定位编译错误、回答追问和形成自己的判断。
- 面试里只背功能容易被追问打断；能说出源码入口、生命周期边界和项目取舍，表达会更像真正用过 UE。

## 读懂的判断标准
- 能说清这个机制解决什么问题，而不是只背类名和函数名。
- 能指出一个源码入口，例如某个头文件、函数签名、关键注释或断言。
- 能说出至少一个边界：什么时候该用，什么时候不该用。
- 能把机制落到项目写法上，例如代码应该放在构造函数、`BeginPlay`、组件初始化还是 `EndPlay`。
- 能回答面试追问：如果不用这个机制会怎样，常见错误会导致什么后果。

## 每次阅读后的最小产物
- 一个具体问题。
- 两到三个源码证据。
- 一个机制结论。
- 一个项目用法。
- 一个常见坑。
- 一段可以直接用于面试的表达。

## 机制 01：`UObject` 是什么

### 为什么重要
- `UObject` 是 UE 反射、序列化、GC、蓝图、编辑器属性系统的基础。
- 你要做 UE C++ 客户端，必须知道哪些对象应该进入 `UObject` 系统，哪些只是普通 C++ 对象。

### 源码入口
- 本地源码：
  - `D:\soft\UE_5.7\Engine\Source\Runtime\CoreUObject\Public\UObject\Object.h`
  - `D:\soft\UE_5.7\Engine\Source\Runtime\CoreUObject\Public\UObject\Class.h`
- 官方文档：
  - https://dev.epicgames.com/documentation/unreal-engine/objects-in-unreal-engine
- 阅读建议：
  - 在 `Object.h` 中搜索 `class UObject`。
  - 在 `Class.h` 中搜索 `class UClass`。

### 核心讲解
- 普通 C++ 对象只有语言层面的构造、析构、类型和内存管理。
- `UObject` 额外接入 UE 的对象系统：
  - 反射：运行时能知道类、属性、函数元数据。
  - 序列化：属性可以保存、加载、复制。
  - GC：UE 可以追踪对象引用并回收不可达对象。
  - 编辑器和蓝图：属性和函数可以暴露给工具链。
- 不是所有东西都应该继承 `UObject`：
  - 轻量值类型更适合 `USTRUCT` 或普通 struct。
  - 需要世界位置和生命周期的对象通常是 `AActor`。
  - 可复用行为片段通常是 `UActorComponent`。

### 最小例子
```cpp
UCLASS(BlueprintType)
class UMySkillConfig : public UObject
{
    GENERATED_BODY()

public:
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    float Cooldown = 3.0f;
};
```

这个类进入 UE 对象系统后，`Cooldown` 可以被反射、编辑器识别，也能被 GC 追踪引用。

### 常见误区
- 误区：`UObject` 只是 UE 里的普通基类。
  - 更准确：它是 UE 对象系统入口。
- 误区：所有游戏数据都要继承 `UObject`。
  - 更准确：只在需要反射、GC、蓝图、序列化等能力时才需要。

### 面试表达
- `UObject` 是 UE 反射和 GC 对象体系的基础。继承 `UObject` 的类型可以被 UHT 生成元数据，从而支持属性反射、序列化、编辑器显示、蓝图访问和 GC 引用追踪。普通 C++ 类型没有这些能力，所以我会根据是否需要 UE 对象系统能力来决定是否使用 `UObject`。

### 练习题
- 一个技能运行时临时计算数据，是否一定要继承 `UObject`？为什么？
  不需要，因为这些临时数据不需要被反射，序列化，GC，他们本就是临时数据，随着计算的结束而销毁

- `UObject`、`USTRUCT`、普通 C++ struct 的适用场景分别是什么？

  UObject：当需要使用到UE的反射，序列化，GC时使用，并且能接UFUCTION的一些功能

  USTRUCT：比UObject更轻量级但不能使用UFUNCTION，可以作为一些变量的集合需要使用反射，序列化和GC的功能
  普通 C++ struct：更轻量级，不需要UE的反射来完成一些功能时候适合

### 练习评价与详解
- 总体评价：方向是对的。你已经抓住了核心判断标准：是否需要进入 UE 对象系统。如果只是一次计算过程中的临时数据，普通 C++ 类型或局部变量就够了，不需要为了“看起来像 UE”而继承 `UObject`。
- 需要修正的一点：`USTRUCT` 本身不是 GC 对象，不能像 `UObject` 一样被 GC 回收。更准确的说法是：`USTRUCT` 可以进入反射系统，结构体里的 `UPROPERTY` 成员可以被序列化、编辑器识别、蓝图识别；如果结构体作为某个 `UObject` 的 `UPROPERTY` 成员存在，并且里面也有 `UPROPERTY` 标记的 `UObject` 引用，那么这些引用可以被 GC 追踪。
- `UObject` 适合有“对象身份”和 UE 系统能力的东西：需要反射、GC、序列化、蓝图函数、编辑器资产、运行时对象引用关系时使用。它可以有 `UFUNCTION`，可以被 `NewObject` 创建，有 `Outer`、`UClass`、CDO 等对象系统概念。
- `USTRUCT` 适合轻量数据和值类型：比如属性配置、表格行、命中结果、技能参数、存档片段。它比 `UObject` 轻，没有独立对象身份，不能写 `UFUNCTION`，也没有 UObject 生命周期。
- 普通 C++ `struct/class` 适合完全不需要 UE 反射系统的内部实现：临时计算、算法状态、纯 C++ 工具类、局部缓存等。

更稳的面试表达：
- 临时计算数据不一定要继承 `UObject`。如果它不需要反射、蓝图、编辑器、序列化或 GC 引用追踪，用普通 C++ 类型更轻。需要 UE 对象系统能力时才使用 `UObject`；需要可反射的轻量值类型时用 `USTRUCT`；完全内部的算法数据就用普通 C++ 类型。

## 机制 02：`UCLASS`、`UPROPERTY`、`UFUNCTION` 和 UHT

### 为什么重要
- 这是 UE C++ 面试第一批高频问题。
- 你之前答卷里对 UHT 和反射的边界还不稳，需要彻底理清。

### 源码入口
- 本地源码：
  - `D:\soft\UE_5.7\Engine\Source\Runtime\CoreUObject\Public\UObject\ObjectMacros.h`
  - `D:\soft\UE_5.7\Engine\Source\Runtime\CoreUObject\Public\UObject\Class.h`
- 官方文档：
  - https://dev.epicgames.com/documentation/unreal-engine/unreal-engine-uproperties
  - https://dev.epicgames.com/documentation/unreal-engine/ufunctions-in-unreal-engine
  - https://dev.epicgames.com/documentation/unreal-engine/unreal-header-tool-for-unreal-engine
- 阅读建议：
  - 在 `ObjectMacros.h` 搜 `UCLASS`、`UPROPERTY`、`UFUNCTION`、`GENERATED_BODY`。

### 核心讲解
- C++ 本身没有 UE 需要的完整运行时反射能力。
- UE 使用宏标记需要反射的类、属性和函数。
- UHT 在 C++ 编译前扫描头文件，生成 `.generated.h` 相关代码和元数据。
- `UCLASS` 让类进入 UE 反射系统。
- `UPROPERTY` 让属性可以被反射、编辑器显示、蓝图访问、序列化、GC 追踪、网络复制等系统识别。
- `UFUNCTION` 让函数可以被反射、蓝图调用、RPC、事件系统、动态委托等使用。
- `.generated.h` 必须最后 include，因为它依赖前面声明，并会注入生成代码。

### 最小例子
```cpp
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "MyTrainingActor.generated.h"

UCLASS(Blueprintable)
class AMyTrainingActor : public AActor
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Stats")
    float MaxHealth = 100.0f;

    UFUNCTION(BlueprintCallable, Category="Stats")
    void ApplyDamage(float Damage);
};
```

### 常见误区
- 误区：`UPROPERTY` 只是让变量在蓝图里看见。
  - 更准确：它还能接入序列化、GC、编辑器、复制等系统，具体取决于 specifier。
- 误区：`UFUNCTION` 只是让函数蓝图可调用。
  - 更准确：RPC、动态委托、事件等也依赖它。
- 误区：宏是运行时执行的。
  - 更准确：很多宏主要是给 UHT 和编译期生成代码使用。

### 面试表达
- UE 通过 `UCLASS`、`UPROPERTY`、`UFUNCTION` 标记需要反射的类型、属性和函数。UHT 会在 C++ 编译前扫描这些宏并生成元数据和胶水代码，使它们能被编辑器、蓝图、序列化、GC 和网络复制等系统识别。

### 练习题
- 为什么动态委托绑定的函数通常必须是 `UFUNCTION`？

  因为动态委托是BlueprintAssignable，值这个事件可以在蓝图中实现，所以需要绑定的函数被UFUNCTION标记能被反射(感觉答得不对希望补充)

- `UPROPERTY(EditAnywhere, BlueprintReadWrite)` 和 `UPROPERTY()` 的效果有什么不同？
  前者可以在编辑器显示且修改，并且能在蓝图中调用，而后置只是被反射和GC功能没有这两个独特的功能

### 练习评价与详解
- 总体评价：你第二题答得比较接近；第一题你自己感觉“不对”是很敏锐的，问题主要在于把“动态委托”和 `BlueprintAssignable` 绑定得太死了。
- 动态委托要求绑定函数通常是 `UFUNCTION`，核心原因不是“这个事件可以在蓝图中实现”，而是动态委托走的是 UE 反射系统。源码里 `AddDynamic` 会把函数名传进去，`BindUFunction` 也是按 `UObject + FName` 绑定函数；这意味着目标函数必须能被反射系统找到，所以通常需要 `UFUNCTION`。
- `BlueprintAssignable` 是属性层面的 specifier，常用于把动态多播委托暴露给蓝图绑定；但即使不从蓝图绑定，只要你使用 `AddDynamic` / 动态委托那套反射式绑定，目标函数也应该是 `UFUNCTION`。
- 普通 C++ 委托和动态委托的区别可以这样记：普通委托更偏 C++，可以绑定普通成员函数、lambda 等；动态委托能序列化、能暴露给蓝图、能通过反射按名字调用，所以限制更多、性能也通常更低。
- `UPROPERTY(EditAnywhere, BlueprintReadWrite)` 和 `UPROPERTY()` 的区别，你答的方向是对的。`UPROPERTY()` 不是“没功能”，它仍然会让属性进入反射系统，常见作用包括序列化、GC 引用追踪、复制系统识别等；`EditAnywhere` 增加编辑器可编辑性，`BlueprintReadWrite` 增加蓝图读写权限。

更稳的面试表达：
- 动态委托依赖 UE 反射系统保存和调用目标函数，绑定时通常会记录 `UObject` 和函数名，因此目标函数需要用 `UFUNCTION` 暴露给反射系统。`BlueprintAssignable` 是把委托属性暴露给蓝图绑定的能力，不等同于动态委托本身的全部原因。
- `UPROPERTY()` 已经能让成员进入 UE 属性系统；`EditAnywhere` 决定编辑器可编辑，`BlueprintReadWrite` 决定蓝图可读写。是否显示、是否蓝图可访问、是否被 GC/序列化识别，是不同层面的能力。

## 机制 03：CDO 和构造函数边界

### 为什么重要
- CDO 是 UE C++ 非常容易被问穿的机制。
- 如果不理解 CDO，就容易把运行时逻辑写进构造函数，导致编辑器、蓝图默认值、运行时状态混乱。

### 源码入口
- 本地源码：
  - `D:\soft\UE_5.7\Engine\Source\Runtime\CoreUObject\Public\UObject\Class.h`
  - `D:\soft\UE_5.7\Engine\Source\Runtime\CoreUObject\Public\UObject\UObjectGlobals.h`
- 官方文档：
  - https://dev.epicgames.com/documentation/unreal-engine/creating-objects-in-unreal-engine
- 阅读建议：
  - 在 `Class.h` 搜 `GetDefaultObject`。
  - 在 `UObjectGlobals.h` 搜 `NewObject`。

### 核心讲解
- CDO 是 Class Default Object，类默认对象。
- 每个 `UClass` 会有一个默认对象，保存类的默认属性。
- 新实例会基于 CDO 的默认值初始化。
- 构造函数不仅在运行时创建实例时可能执行，也会用于创建 CDO，并可能在编辑器阶段发生。
- 所以构造函数适合：
  - 创建默认子对象：`CreateDefaultSubobject`。
  - 设置默认属性值。
  - 设置组件默认附加关系。
- 构造函数不适合：
  - 查找运行时 Actor。
  - 访问玩家输入。
  - 依赖 `World` 的 Gameplay 逻辑。
  - 播放特效、生成运行时对象、发起网络逻辑。

### 最小例子
```cpp
AMyCharacter::AMyCharacter()
{
    PrimaryActorTick.bCanEverTick = true;

    CameraBoom = CreateDefaultSubobject<USpringArmComponent>(TEXT("CameraBoom"));
    CameraBoom->SetupAttachment(RootComponent);

    FollowCamera = CreateDefaultSubobject<UCameraComponent>(TEXT("FollowCamera"));
    FollowCamera->SetupAttachment(CameraBoom);

    MaxHealth = 100.0f;
}

void AMyCharacter::BeginPlay()
{
    Super::BeginPlay();
    CurrentHealth = MaxHealth;
}
```

### 常见误区
- 误区：构造函数就是游戏开始时调用。
  - 更准确：构造函数和对象默认值、CDO、编辑器构建都有关系。
- 误区：只要构造函数里能拿到指针，就可以做 Gameplay 初始化。
  - 更准确：运行时逻辑应优先放在 `BeginPlay`、初始化函数或明确的生命周期回调中。

### 面试表达
- CDO 是 UE 每个 `UClass` 对应的类默认对象，保存默认属性，新对象会从 CDO 拷贝默认值。构造函数会参与 CDO 构建，也可能在编辑器阶段执行，所以我只在构造函数里创建默认组件和设置默认值，把依赖运行时世界状态的 Gameplay 逻辑放到 `BeginPlay` 或后续初始化流程。

### 练习题
- 为什么在构造函数里 `GetWorld()` 相关逻辑可能不可靠？

  不可靠，因为有些情况比如你在一个`Level`中放置了一个`Actor`蓝图，然后你在他的构造函数里写`GetWorld`显而易见会崩溃，因为这时候`Level`还没有加载完全，也许`Level`和这个`Actor`是在同时进行构造的，你去`GetWorld`显然可能得到一个`nullptr`值

- `CreateDefaultSubobject` 和 `NewObject` 分别应该在什么场景用？
  `CreateDefaultSubobject`好像只能在构造函数中对在头文件中声明的一些`Component`或者subobject使用，而`NewObject`则是在游戏运行过程中，比如`BeginPlay`,`Tick`,`EndPlay`，看你情况，然后我看了NewObject好像也能被使用在构造函数但是需要你提供Name参数(可以补充全一点)

### 练习评价与详解
- 总体评价：你已经抓到“构造函数不是正常 Gameplay 阶段”这个重点了。需要再精确一点：问题不只是 Level 是否加载完成，而是构造函数可能在 CDO 创建、蓝图默认对象重建、编辑器预览、Actor 生成流程等多个阶段执行，这些阶段不一定有有效的运行时 `UWorld`，也不一定应该执行 Gameplay 逻辑。
- `GetWorld()` 在构造函数中可能不可靠，因为当前对象可能是 CDO，CDO 是类默认对象，不代表世界里的真实实例；也可能是编辑器阶段为了展示默认值、构造脚本或蓝图重建而创建的对象。即使某些情况下能拿到 World，也不代表玩家、GameMode、其他 Actor、网络状态都已经准备好。
- 构造函数适合做“默认结构”：创建默认组件、设置默认属性、设置默认附加关系。运行时依赖世界状态的逻辑，例如查找 Actor、绑定玩家输入、启动 Timer、播放特效、发起网络请求，应放在 `BeginPlay`、`PostInitializeComponents` 或更明确的初始化流程里。
- `CreateDefaultSubobject` 的重点是“默认子对象”，通常在构造函数中使用，常见对象是组件，但不只限于组件。它创建出来的子对象会进入类默认对象和实例默认结构，成为这个类默认拥有的一部分。
- `NewObject` 的重点是“运行时创建普通 `UObject`”。源码里 `NewObject` 在构造函数里有 `AssertIfInConstructor` 提示：不要用空名字的 `NewObject` 在 UObject 派生类构造函数里创建默认子对象，应使用 `CreateDefaultSubobject`。你提到“提供 Name 参数”能绕过部分断言，这个观察很细，但项目规则上仍应避免用 `NewObject` 来承担默认子对象职责。
- 更准确的边界是：默认组件/默认子对象用 `CreateDefaultSubobject`；运行时临时或动态的非 Actor `UObject` 用 `NewObject`；如果需要创建运行时组件，则通常 `NewObject` 后还要设置 Outer、注册组件、附加到合适组件，并用 `UPROPERTY` 或组件注册关系保证生命周期清晰。

更稳的面试表达：
- 构造函数会参与 CDO 构建和编辑器阶段对象构造，不等于游戏开始。`GetWorld()` 即使有时能拿到，也不代表运行时世界状态可用，所以构造函数只做默认组件和默认值；依赖 World、玩家、其他 Actor 或网络状态的逻辑放到 `BeginPlay` 或初始化回调里。
- `CreateDefaultSubobject` 用于构造默认子对象，尤其是默认组件；`NewObject` 用于运行时创建非 Actor 的 `UObject`。不要用 `NewObject` 代替默认子对象创建。

## 机制 04：对象创建：`CreateDefaultSubobject`、`NewObject`、`SpawnActor`

### 为什么重要
- 你做 Demo 时会创建组件、运行时对象、Actor。三种创建方式必须分清。

### 源码入口
- 本地源码：
  - `D:\soft\UE_5.7\Engine\Source\Runtime\CoreUObject\Public\UObject\UObjectGlobals.h`
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Classes\Engine\World.h`
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Classes\GameFramework\Actor.h`
- 阅读建议：
  - 在 `UObjectGlobals.h` 搜 `NewObject`。
  - 在 `World.h` 搜 `SpawnActor`。
  - 在 `Object.h` 或相关头中搜 `CreateDefaultSubobject`。

### 核心讲解
- `CreateDefaultSubobject<T>`：
  - 用于构造函数中创建默认子对象。
  - 常见：组件。
  - 会成为类默认对象/实例默认结构的一部分。
- `NewObject<T>`：
  - 用于运行时创建非 Actor 的 `UObject`。
  - 需要合适的 Outer。
  - 如果要长期持有，需要 `UPROPERTY` 引用或其他 GC 可见引用。
- `SpawnActor<T>`：
  - 用于运行时在世界中生成 Actor。
  - 需要 World、Class、Transform 等。
  - Actor 有世界生命周期、网络复制、Tick、碰撞等能力。

### 最小例子
```cpp
// 构造函数中创建默认组件
HealthComponent = CreateDefaultSubobject<UMyHealthComponent>(TEXT("HealthComponent"));

// 运行时创建 UObject
UMyRuntimeSkill* Skill = NewObject<UMyRuntimeSkill>(this);

// 运行时生成 Actor
AActor* Spawned = GetWorld()->SpawnActor<AActor>(ActorClass, SpawnTransform);
```

### 常见误区
- 误区：运行时添加组件也用 `CreateDefaultSubobject`。
  - 更准确：`CreateDefaultSubobject` 是构造函数默认子对象创建；运行时组件创建还涉及 `NewObject`、注册和附加。
- 误区：所有 UObject 创建后都会自动不被 GC。
  - 更准确：如果没有可达引用，GC 仍可能回收。

### 面试表达
- 我会按对象类型和生命周期选创建方式。默认组件在构造函数用 `CreateDefaultSubobject`；运行时非 Actor 的 UObject 用 `NewObject` 并设置合适 Outer 和引用；需要进入世界、有 Transform、Tick、复制或碰撞的对象用 `SpawnActor`。

### 练习题
- 运行时创建一个技能对象并保存到角色上，如何避免它被 GC？
  这个对象如果需要附加到角色身上那一般是`Component`，我会用`NewObject`进行创建后去`AttachToComponent`到角色的Mesh上依次避免GC，如果是Actor，我则会`SpawnActor`后`AttachToActor`到角色上
- 为什么 Actor 不能用 `NewObject` 当作正常生成方式？
  因为Actor需要生成在场景中，且相比于object多了一些功能需要初始化所以不能直接通过`NewObject`直接生成，

### 练习评价与详解
- 总体评价：你第二题方向正确，已经知道 Actor 不只是普通 `UObject`。第一题需要重点修正：避免 GC 的关键不是 `AttachToComponent` 或 `AttachToActor`，而是对象必须在 GC 可达引用链上。
- 如果“技能对象”是普通 `UObject`，推荐写法通常是角色里保存一个 `UPROPERTY()` 成员，例如 `UPROPERTY() TObjectPtr<UMySkill> CurrentSkill;`，然后 `CurrentSkill = NewObject<UMySkill>(this);`。这里 `Outer = this` 表示对象归属关系，`UPROPERTY` 成员引用让 GC 能追踪到它。
- `Outer` 很重要，但不要把它理解成万能保命符。更稳的项目写法是：合适的 Outer + `UPROPERTY` 强引用。如果只是局部变量接住 `NewObject` 返回值，函数结束后没有可达引用，后续 GC 仍可能回收。
- 如果技能其实是组件，运行时创建组件不是只 `AttachToComponent` 就结束，通常还要 `NewObject<UMyComponent>(Owner)`、必要时 `RegisterComponent()`、设置附加关系，并让 Actor/组件系统或 `UPROPERTY` 持有它。附加关系解决的是场景层级/Transform，不是所有 UObject 的生命周期管理。
- 如果技能是 Actor，则应该用 `GetWorld()->SpawnActor` 创建，再根据需要 `AttachToActor` 或 `AttachToComponent`。`SpawnActor` 会走 `UWorld` 的 Actor 生成流程，处理 Transform、Level、生命周期回调、组件初始化、网络等 Actor 专属流程。
- Actor 不能用 `NewObject` 当正常生成方式，是因为 `AActor` 虽然继承自 `UObject`，但它还需要被放入 `UWorld` / Level 的 Actor 管理体系，走 `SpawnActor` 的初始化链。源码入口上也能看到 `SpawnActor` 是 `UWorld` 的接口，而不是普通 UObject 全局创建函数。

更稳的面试表达：
- 运行时创建技能 UObject 后，我会用合适的 Outer 创建，并用角色上的 `UPROPERTY() TObjectPtr<UMySkill>` 保存它，让 GC 能从角色追踪到技能对象。如果是组件，还要注册和附加；如果是 Actor，则用 `SpawnActor`，因为 Actor 需要进入 World/Level 的生命周期和初始化流程，不能把 `NewObject` 当正常生成方式。

## 机制 05：UE GC、`UPROPERTY`、`TObjectPtr`

### 为什么重要
- UE C++ 崩溃和悬挂引用常与 GC 认知有关。
- 你答卷里知道 `UPROPERTY` 和 GC 有关，但需要更准确。

### 源码入口
- 本地源码：
  - `D:\soft\UE_5.7\Engine\Source\Runtime\CoreUObject\Public\UObject\Object.h`
  - `D:\soft\UE_5.7\Engine\Source\Runtime\CoreUObject\Public\UObject\ObjectPtr.h`
  - `D:\soft\UE_5.7\Engine\Source\Runtime\CoreUObject\Public\UObject\GarbageCollection.h`
- 官方文档：
  - https://dev.epicgames.com/documentation/unreal-engine/unreal-object-handling-in-unreal-engine

### 核心讲解
- UE GC 从根集合出发，遍历能被反射系统识别的 `UObject` 引用。
- `UPROPERTY` 让成员引用进入 UE 的反射/引用追踪系统。
- UE5 推荐成员 UObject 引用写成 `UPROPERTY() TObjectPtr<UObjectType>`。
- 裸 `UObject*` 可用于局部变量和函数参数，但长期作为成员保存会让 GC 不可见。
- `TWeakObjectPtr` 是非拥有引用，对象销毁后可以变无效，使用前要判断。
- `TSharedPtr`/`TUniquePtr` 不用于管理 UObject。

### 最小例子
```cpp
UCLASS()
class AMyActor : public AActor
{
    GENERATED_BODY()

private:
    UPROPERTY()
    TObjectPtr<UMyRuntimeSkill> CurrentSkill;
};
```

### 常见误区
- 误区：只要是指针，GC 都能知道。
  - 更准确：GC 只能追踪它能识别的 UObject 引用。
- 误区：`UPROPERTY` 只负责蓝图显示。
  - 更准确：即使没有蓝图 specifier，`UPROPERTY()` 也能让 GC 追踪引用。
- 误区：`TWeakObjectPtr` 会让对象不被回收。
  - 更准确：弱引用不拥有对象。

### 面试表达
- UE GC 会从 Root 出发遍历 UObject 引用链。成员 UObject 引用如果没有通过 `UPROPERTY` 暴露给反射系统，GC 可能不知道这条引用，导致对象被回收后留下悬垂指针。UE5 中我会用 `UPROPERTY() TObjectPtr<T>` 保存 UObject 成员引用，用 `TWeakObjectPtr` 表达非拥有引用。

### 练习题
- `UPROPERTY()` 没有 `BlueprintReadWrite`，还对 GC 有意义吗？有因为`BlueprintReadWrite`只是让蓝图能读写，但GC是UPROPERTY()标记后这个变量就已经被加入GC的图表里面了
- `TWeakObjectPtr` 适合保存什么关系？为什么使用前要 `IsValid()`？适合保存一些被临时需要去观察的Obejct就比如我想要记住上一个杀死我的对象时候就可以使用，因为TWeakObjectPtr不保证对象的存活所以需要使用IsVaild或者Pin

### 练习评价与详解
- 总体评价：第一个问题答得很稳，已经把 `BlueprintReadWrite` 和 GC 追踪区分开了。`BlueprintReadWrite` 是蓝图访问权限，GC 关心的是 UObject 引用是否进入 UE 的属性/引用收集系统。
- 更准确地说，`UPROPERTY()` 即使没有 `BlueprintReadWrite`、`EditAnywhere`，也仍然是 UE 反射属性。只要它是成员 UObject 引用，例如 `UPROPERTY() TObjectPtr<UMyObject> Obj;`，GC 就可以通过反射生成的引用信息追踪到这个对象。
- 但要注意：不是所有 `UPROPERTY` 都和 GC 有直接关系。`int32`、`float` 这类值类型也可以是 `UPROPERTY`，它们能被序列化、编辑器/蓝图识别，但不存在“引用的 UObject 被 GC 保活”这个问题。GC 重点关心的是 UObject 引用链。
- `TWeakObjectPtr` 的理解方向也对：它适合“观察关系”或“非拥有关系”，比如上一个攻击者、当前锁定目标、临时交互对象、缓存的 UI/Actor 引用等。它不会阻止对象被销毁或 GC。
- 使用前要判断有效性，是因为弱引用指向的对象可能已经销毁、Pending Kill 或被 GC 标记。普通写法是 `IsValid()` 后 `Get()`；UE 5.7 里 `TWeakObjectPtr` 也有 `Pin()` / `TryPin()`，可以临时得到 `TStrongObjectPtr`，用于你确实需要在一小段作用域内保证对象不被回收的场景。
- 小修正：`Object` 拼写注意；`IsValid` 不只是判断空指针，还会考虑 UObject 是否已经进入不可用状态。

更稳的面试表达：
- `UPROPERTY()` 不等于蓝图可见，它首先让成员进入 UE 属性系统。对于 UObject 成员引用，即使没有 `BlueprintReadWrite`，也能让 GC 通过反射引用链追踪对象。`BlueprintReadWrite` 只是额外决定蓝图是否能读写这个属性。
- `TWeakObjectPtr` 用来表达非拥有引用，不会让对象保活，所以访问前要 `IsValid()` / `Get()`，需要短时间强持有时可以考虑 `Pin()`。

## 机制 06：Actor 生命周期

### 为什么重要
- Gameplay 代码每天都在和 Actor 生命周期打交道。
- 不理解生命周期，就容易在错误时机访问组件、绑定事件、启动 Timer、执行网络逻辑。

### 源码入口
- 本地源码：
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Classes\GameFramework\Actor.h`
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Private\Actor.cpp`
- 官方文档：
  - https://dev.epicgames.com/documentation/unreal-engine/actors-in-unreal-engine
- 阅读建议：
  - 在 `Actor.h` 搜 `BeginPlay`、`EndPlay`、`Tick`、`PreInitializeComponents`、`PostInitializeComponents`。

### 阅读问题
- `Actor` 的构造函数、组件初始化、`BeginPlay`、`Tick`、`EndPlay` 分别适合放什么逻辑？
- `BeginPlay` 是不是等同于构造函数？`Tick` 是不是默认开启？
- 哪些生命周期函数只在 Gameplay 阶段调用，哪些会受到编辑器构造逻辑影响？

### 源码证据
- `Actor.h` 中 `BeginPlay` 附近可以看到 `ReceiveBeginPlay` 是 `BlueprintImplementableEvent`，而 `BeginPlay` 是 C++ 可重写的 native virtual 函数；下面还有 `DispatchBeginPlay`，说明真正触发顺序由引擎分发控制。
- `Actor.h` 中 `Tick(float DeltaSeconds)` 的注释明确写着 Tick 默认关闭，需要设置 `PrimaryActorTick.bCanEverTick = true` 才能启用。
- `Actor.h` 中 `PreInitializeComponents` 和 `PostInitializeComponents` 的注释都强调只在 Gameplay 阶段调用，适合区分编辑器构造逻辑和运行时初始化逻辑。
- `Actor.h` 中 `OnConstruction(const FTransform& Transform)` 是构造相关入口，和运行时 `BeginPlay` 不是同一个阶段。

### 核心讲解
- 简化顺序：
  - 构造函数：默认组件和默认值。
  - OnConstruction：构造脚本/编辑器相关构造逻辑。
  - PreInitializeComponents：组件初始化前。
  - PostInitializeComponents：组件初始化后。
  - BeginPlay：游戏开始或 Actor 进入播放状态。
  - Tick：每帧更新，需开启。
  - EndPlay：销毁、关卡切换、PIE 停止等。
- 实际流程更复杂，但面试和项目中先掌握职责边界最重要。

### 最小例子
```cpp
AMyActor::AMyActor()
{
    PrimaryActorTick.bCanEverTick = true;
}

void AMyActor::BeginPlay()
{
    Super::BeginPlay();
    UE_LOG(LogTemp, Log, TEXT("BeginPlay"));
}

void AMyActor::Tick(float DeltaSeconds)
{
    Super::Tick(DeltaSeconds);
}

void AMyActor::EndPlay(const EEndPlayReason::Type EndPlayReason)
{
    Super::EndPlay(EndPlayReason);
    UE_LOG(LogTemp, Log, TEXT("EndPlay"));
}
```

### 常见误区
- 误区：`BeginPlay` 一定紧跟构造函数。
  - 更准确：中间还有注册、初始化、关卡加载等流程。
- 误区：Tick 默认开启。
  - 更准确：Actor 需要 `PrimaryActorTick.bCanEverTick = true`。
- 误区：`EndPlay` 只在手动 Destroy 时调用。
  - 更准确：关卡切换、PIE 结束等也可能触发。

### 面试表达
- 我会把 Actor 生命周期分成默认构建、组件初始化、游戏开始、运行中 Tick、结束清理几个阶段。构造函数只放默认组件和值，`BeginPlay` 放运行时初始化，`EndPlay` 做委托解绑、Timer 清理和运行时引用清理。

### 练习题
- 你会在 `BeginPlay` 做哪些事，不会做哪些事？BeginPlay我会做一些初始化的操作，一些委托的绑定，设置生命周期，不会做组件的CreateDefaultSubobject(不知道还能不做啥，感觉BeginPlay基本能做所有事情)
- 为什么 `EndPlay` 里要解绑委托或清理 Timer？要不然可能导致内存泄漏吧（不确定）Timer和委托毕竟不在Actor上不随Actor的销毁而销毁，比如Timer是World有个TimerManager可能导致残留

### 练习评价与详解
- 总体评价：你对 `BeginPlay` 的定位基本正确：它适合运行时初始化、绑定事件、启动运行时逻辑，不适合创建默认子对象。你对 Timer 的判断也很接近，`TimerManager` 确实属于 World 侧管理，不能简单认为 Actor 销毁后所有回调都自然安全。
- `BeginPlay` 适合做的事：读取运行时配置、初始化当前状态、缓存运行时依赖、绑定输入/委托、启动 Timer、根据关卡状态查找必要对象、通知组件进入游戏状态等。此时 Actor 和组件通常已经完成初始化，世界上下文也比构造函数可靠。
- `BeginPlay` 不适合做的事：创建默认组件或默认子对象、设置类默认值、依赖构造脚本的编辑器逻辑、做非常重的同步资源加载、每个 Actor 都做大范围 `GetAllActorsOfClass` 搜索、假设所有其他 Actor 的 `BeginPlay` 都已经按你希望的顺序完成。
- “BeginPlay 基本能做所有事情”这个感觉要收一下：它是运行时入口，但不是万能入口。比如默认组件结构应该在构造函数；组件注册后的初始化可以考虑 `PostInitializeComponents`；需要网络同步的逻辑还要区分 Authority、客户端、Possession、RepNotify 等时机。
- `EndPlay` 里清理委托和 Timer 的核心不是单纯“内存泄漏”，更常见的问题是悬挂回调、重复绑定、对象已结束后仍被回调访问、下一次 PIE 或重新生成时状态残留。
- Timer 的确由 `World` 的 `FTimerManager` 管理。源码里 `SetTimer` 会把对象方法包装进 Timer 委托，`TimerManager` 也提供 `ClearAllTimersForObject`。所以如果 Actor 持有 `FTimerHandle`，在 `EndPlay` 中 `ClearTimer` 或 `ClearAllTimersForObject(this)` 是很常见的收尾动作。
- 委托也类似：如果你绑定到别人的事件上，别人可能比你活得更久。`EndPlay` 中解绑可以避免你的对象结束后仍被事件系统间接调用，也能避免再次进入关卡时重复绑定。

更稳的面试表达：
- 我会把 `BeginPlay` 当作运行时初始化入口，用来绑定委托、启动 Timer、初始化运行时状态和获取可靠的 World 依赖；但默认组件创建、默认值设置仍放在构造函数，组件级初始化或网络相关逻辑还要看更具体的生命周期。
- `EndPlay` 中我会清理 Timer、解绑委托、释放运行时引用，因为 Timer/委托可能由 World 或其他对象持有。清理的重点是避免对象结束后仍有回调触发、重复绑定或状态残留。

## 机制 07：ActorComponent 生命周期和组件化

### 为什么重要
- 你的 Demo 会大量用组件拆生命值、技能、战斗、交互。
- 组件化能让系统更好讲，也更接近 UE 客户端岗位实际开发。

### 源码入口
- 本地源码：
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Classes\Components\ActorComponent.h`
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Private\Components\ActorComponent.cpp`
- 官方文档：
  - https://dev.epicgames.com/documentation/unreal-engine/components-in-unreal-engine

### 核心讲解
- `UActorComponent` 是可挂在 Actor 上的行为模块。
- 它没有 Transform；需要 Transform 用 `USceneComponent`。
- 组件生命周期通常跟随 Owner。
- 默认组件通常在 Actor 构造函数中 `CreateDefaultSubobject`。
- 运行时创建组件要注意注册、附加和引用保存。
- 组件化适合：
  - HealthComponent
  - CombatComponent
  - SkillComponent
  - InteractionComponent
  - InventoryComponent

### 最小例子
```cpp
UCLASS(ClassGroup=(Custom), meta=(BlueprintSpawnableComponent))
class UMyHealthComponent : public UActorComponent
{
    GENERATED_BODY()

public:
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category="Health")
    float MaxHealth = 100.0f;

    UFUNCTION(BlueprintCallable, Category="Health")
    void ApplyDamage(float Damage);
};
```

### 常见误区
- 误区：组件只是为了蓝图拖拽方便。
  - 更准确：组件是行为复用和职责拆分的重要方式。
- 误区：组件可以随意访问 Owner 的所有细节。
  - 更准确：组件应尽量通过清晰接口和事件通信，避免强耦合。

### 面试表达
- 我会用组件拆分可复用 Gameplay 能力，比如生命、战斗、技能、交互。Actor 负责组合组件，组件负责单一职责。这样系统边界更清楚，也便于复用、测试和面试讲解。

### 练习题
- HealthComponent 应该知道 UI Widget 吗？为什么？不应该，避免组件之间的强耦合，或者说HealthComponent就应该只管理Health相关的，而不是UI，如果UI需要血量，只需要血量提供一个接口
- SkillComponent 和 CombatComponent 应该如何通信？我说2种做法，一种他们深度耦合，Skill一定从Combat出发，那么可以在Combat里面加一个TweakObjectPtr保存Skill组件，在他们的Owner初始化之后让Owner调用Combat的初始化就好了，然后Skill怎么传递消息给Combat呢，Skill不应该传递消息给Combat，Combat算一个上级管所有战斗相关的Component，Skill通过一些委托告诉Combat需要其他Component进行配合,第二种，他们属于有很独立的功能，只是其中某一个功能需要相互通信按照ECS的方法我需要一个单例我可能会在他们的上层中加一个东西，比如我想先Combat什么功能再调用Skill我会通过这个上层一点的粘合剂或者Manager完成这个功能，你最好能给我一个他们需要相互通信的例子我来试着能不能用这个方式解决，反正有第一种作为保底

### 练习评价与详解
- 总体评价：这两题方向不错。你已经抓住组件化最重要的两个原则：单一职责和降低耦合。`HealthComponent` 不应该知道具体 UI；`SkillComponent` 和 `CombatComponent` 通信前要先判断职责关系，而不是默认互相硬调。
- `HealthComponent` 这题基本正确。更稳的说法是：HealthComponent 负责维护血量、处理伤害/治疗、暴露查询接口，并在血量变化时广播事件，例如 `OnHealthChanged`。UI Widget 订阅这个事件并更新显示。这样换 UI、敌人血条、Boss 血条或网络同步显示时，HealthComponent 都不用改。
- “如果 UI 需要血量，只需要血量提供一个接口”这句话还可以再补一层：只提供 Getter 会让 UI 主动轮询；更 UE 的方式是 Getter + 委托事件。Getter 适合初始化显示，委托适合变化时更新。
- `SkillComponent` 和 `CombatComponent` 这题的好点是：你没有直接说“互相拿指针调用”，而是在想主从关系和上层协调者。这个方向是对的。
- 需要修正的是：不要太快引入“单例”。同一个 Actor 上的组件通信，优先考虑 Owner/Character 作为组合根，或者用接口、委托、明确的初始化函数。全局单例会扩大依赖范围，后期更难测试和替换。
- 拼写注意：是 `TWeakObjectPtr`，不是 `TweakObjectPtr`。如果只是缓存同 Owner 上另一个组件，并且不表达所有权，用 `TWeakObjectPtr` 可以；如果组件由同一个 Actor 稳定拥有，也可以通过初始化时传入普通指针/`TObjectPtr`，关键是依赖方向要清楚。
- 一个具体例子：玩家按技能键时，输入层或 Character 调 `SkillComponent->TryActivateSkill(SkillId)`；SkillComponent 检查冷却和消耗后广播 `OnSkillCommit` 或生成 `FSkillRequest`；CombatComponent 负责命中检测、攻击窗口和伤害结算；HealthComponent 接收最终伤害并广播 `OnHealthChanged`；UI 只监听血量或冷却变化事件。
- 这个例子里的职责边界是：Skill 管技能规则，Combat 管战斗结算，Health 管生命值，UI 管显示，Character/Owner 负责装配组件。这样每个组件都能讲清楚为什么存在。

更稳的面试表达：
- 我不会让 HealthComponent 直接依赖 Widget。HealthComponent 只维护生命值并广播血量变化事件，UI 通过绑定事件或查询接口更新显示。
- SkillComponent 和 CombatComponent 的通信要看职责边界。如果技能释放属于战斗流程，可以让 Character 或 CombatComponent 作为协调者，SkillComponent 只负责技能条件和释放请求，CombatComponent 负责命中和伤害结算。组件之间优先用接口、委托或 Owner 装配，避免互相硬引用和全局单例。

## 机制 08：委托和事件通信

### 为什么重要
- 你答卷里提到 UI 和 Gameplay 用委托通信，这是一个很好的方向。
- 委托能帮助你减少 UI、角色、组件之间的强耦合。

### 源码入口
- 本地源码：
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Core\Public\Delegates`
  - `D:\soft\UE_5.7\Engine\Source\Runtime\CoreUObject\Public\UObject\ScriptDelegates.h`
- 阅读建议：
  - 搜 `DECLARE_DYNAMIC_MULTICAST_DELEGATE`。

### 核心讲解
- 普通 C++ 委托：不暴露给蓝图，性能更轻。
- Dynamic 委托：可被反射和蓝图使用，需要 `UFUNCTION` 绑定。
- Multicast 委托：多个监听者。
- UI 常见模式：
  - HealthComponent 维护血量。
  - 血量变化时 Broadcast。
  - Widget 监听事件更新显示。
- 解绑很重要，尤其是对象销毁时。

### 最小例子
```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(FOnHealthChanged, float, Current, float, Max);

UPROPERTY(BlueprintAssignable)
FOnHealthChanged OnHealthChanged;

void UMyHealthComponent::ApplyDamage(float Damage)
{
    CurrentHealth = FMath::Max(0.0f, CurrentHealth - Damage);
    OnHealthChanged.Broadcast(CurrentHealth, MaxHealth);
}
```

### 常见误区
- 误区：所有委托都要 Dynamic。
  - 更准确：只有需要蓝图/反射时才用 Dynamic。
- 误区：绑定了就不用解绑。
  - 更准确：对象生命周期复杂时，解绑能避免悬挂调用或重复绑定。

### 面试表达
- 我会用委托做 Gameplay 和 UI 的解耦。例如血量组件只负责维护血量并广播变化，UI 订阅变化并更新显示。这样组件不需要知道具体 Widget，也方便替换 UI 实现。

### 练习题
- Dynamic 委托为什么要求绑定函数是 `UFUNCTION`？
- 血量组件广播事件，UI 订阅事件，销毁时应该注意什么？

## 机制 09：Timer 和 Tick

### 为什么重要
- Tick 滥用是性能和代码结构常见问题。
- 技能冷却、Buff 持续时间、延迟触发都可能用 Timer。

### 源码入口
- 本地源码：
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Public\TimerManager.h`
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Private\TimerManager.cpp`
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Classes\GameFramework\Actor.h`

### 核心讲解
- Tick 适合每帧都需要更新的逻辑。
- Timer 适合延迟执行、周期执行但不需要每帧的逻辑。
- 技能冷却可以用时间戳，也可以用 Timer，看系统设计。
- `EndPlay` 时要考虑清理 Timer，避免回调访问已销毁对象。

### 最小例子
```cpp
FTimerHandle CooldownTimerHandle;

GetWorld()->GetTimerManager().SetTimer(
    CooldownTimerHandle,
    this,
    &AMyCharacter::OnSkillCooldownFinished,
    CooldownSeconds,
    false
);
```

### 常见误区
- 误区：所有持续逻辑都写 Tick。
  - 更准确：能事件驱动或 Timer 的，不要每帧轮询。
- 误区：Timer 不需要管理。
  - 更准确：对象结束时要考虑 ClearTimer。

### 面试表达
- 我会优先避免无意义 Tick。只有每帧依赖 DeltaTime 的逻辑才 Tick；技能冷却、延迟触发、周期检测可以用 Timer 或时间戳。这样能减少每帧负担，也让逻辑触发点更清晰。

### 练习题
- 技能冷却用 Tick、Timer、时间戳三种方式分别有什么优缺点？
- 为什么 `EndPlay` 里可能需要清理 Timer？

## 机制 10：DataAsset、DataTable、数据驱动

### 为什么重要
- 你的 Demo 推荐做技能/敌人数据驱动，这是 UE C++ 客户端非常好讲的点。
- 数据驱动能让你从“写死数值”升级到“可配置系统”。

### 源码入口
- 本地源码：
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Classes\Engine\DataAsset.h`
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Classes\Engine\DataTable.h`
- 官方文档：
  - https://dev.epicgames.com/documentation/unreal-engine/data-driven-gameplay-elements-in-unreal-engine

### 核心讲解
- `UDataAsset` 适合配置对象化数据，例如一个技能配置资产。
- `UPrimaryDataAsset` 可接入 Asset Manager。
- `UDataTable` 适合表格化数据，例如敌人属性表、技能表。
- 数据驱动的核心是：代码负责规则，数据负责参数。

### 最小例子
```cpp
UCLASS(BlueprintType)
class UMySkillDataAsset : public UDataAsset
{
    GENERATED_BODY()

public:
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    FName SkillID;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    float Cooldown = 3.0f;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    float Damage = 20.0f;
};
```

### 常见误区
- 误区：DataAsset 只是把变量搬到资源里。
  - 更准确：它让策划配置、扩展新内容、减少代码改动成为可能。
- 误区：所有数据都适合 DataTable。
  - 更准确：复杂嵌套或对象引用多的数据，用 DataAsset 可能更清晰。

### 面试表达
- 我会把技能、敌人、道具等可配置参数放到 DataAsset 或 DataTable 中，代码只负责执行规则。这样新增技能可以通过新建配置资产完成，减少硬编码，也方便策划调整。

### 练习题
- 技能配置用 DataAsset 还是 DataTable？你会按什么标准选择？
- 一个敌人配置表至少应该包含哪些字段？

## 机制 11：软引用和异步加载

### 为什么重要
- 资源引用会影响加载链和内存。
- 你答卷里软引用/异步加载是短板，后续 Demo 的技能图标、特效、怪物资源都会遇到。

### 源码入口
- 本地源码：
  - `D:\soft\UE_5.7\Engine\Source\Runtime\CoreUObject\Public\UObject\SoftObjectPtr.h`
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Classes\Engine\AssetManager.h`
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Classes\Engine\StreamableManager.h`

### 核心讲解
- 硬引用会让被引用资源跟随加载。
- 软引用保存资源路径，不会立即加载资源。
- 异步加载可以避免运行时卡顿。
- Asset Manager 可以管理 Primary Asset 和异步加载流程。

### 最小例子
```cpp
UPROPERTY(EditDefaultsOnly)
TSoftObjectPtr<UTexture2D> SkillIcon;
```

这个属性不会像硬引用那样立即加载贴图，适合大量可选资源。

### 常见误区
- 误区：软引用就是弱引用。
  - 更准确：软引用面向资产路径和延迟加载；弱引用面向已存在对象的非拥有引用。
- 误区：异步加载只和大项目有关。
  - 更准确：任何运行时加载大资源都可能卡顿。

### 面试表达
- 硬引用会扩大加载链，软引用只保存资产路径，需要时再加载。对于技能图标、特效、怪物资源这类不一定马上用到的资源，我会考虑软引用和异步加载，减少启动或关卡加载压力。

### 练习题
- 软引用和弱引用有什么区别？
- 为什么一个技能 DataAsset 里大量硬引用特效资源可能导致加载变慢？

## 机制 12：Subsystem

### 为什么重要
- 你答卷里不清楚 Subsystem。
- Subsystem 是 UE 中做全局/上下文服务的常用方式，比手写单例更符合引擎生命周期。

### 源码入口
- 本地源码：
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Public\Subsystems\GameInstanceSubsystem.h`
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Public\Subsystems\WorldSubsystem.h`
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Public\Subsystems\LocalPlayerSubsystem.h`

### 核心讲解
- Subsystem 是 UE 自动管理生命周期的系统对象。
- `UGameInstanceSubsystem`：跟随 GameInstance，跨关卡。
- `UWorldSubsystem`：跟随 World，关卡/世界切换会变化。
- `ULocalPlayerSubsystem`：跟随本地玩家。
- 适合放管理类逻辑，如全局配置、匹配状态、本地 UI 状态、运行时服务。

### 最小例子
```cpp
UCLASS()
class UMyCombatLogSubsystem : public UGameInstanceSubsystem
{
    GENERATED_BODY()

public:
    virtual void Initialize(FSubsystemCollectionBase& Collection) override;
    virtual void Deinitialize() override;
};
```

### 常见误区
- 误区：Subsystem 就是全局单例，随便放任何逻辑。
  - 更准确：要根据生命周期选择类型，避免把所有逻辑塞进去。
- 误区：Gameplay Actor 间通信都走 Subsystem。
  - 更准确：局部交互优先组件/接口/委托，Subsystem 适合上下文服务。

### 面试表达
- Subsystem 是 UE 提供的生命周期托管对象，能替代很多手写单例。我会根据生命周期选 GameInstance、World 或 LocalPlayer Subsystem，例如跨关卡数据用 GameInstanceSubsystem，和当前 World 强相关的管理逻辑用 WorldSubsystem。

### 练习题
- 技能系统管理器适合放在 Subsystem 里吗？什么情况下适合，什么情况下不适合？
- `GameInstanceSubsystem` 和 `WorldSubsystem` 的生命周期差异是什么？

## 机制 13：Gameplay Framework 职责

### 为什么重要
- 你答卷里能说出一部分，但面试通常会深入问服务器/客户端职责。
- Demo 后续联机亮点会依赖这些基础。

### 源码入口
- 本地源码：
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Classes\GameFramework\GameModeBase.h`
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Classes\GameFramework\GameStateBase.h`
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Classes\GameFramework\PlayerController.h`
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Classes\GameFramework\PlayerState.h`
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Classes\GameFramework\Pawn.h`
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Classes\GameFramework\Character.h`
- 官方文档：
  - https://dev.epicgames.com/documentation/unreal-engine/gameplay-framework-in-unreal-engine

### 核心讲解
- `GameInstance`：游戏进程级对象，跨关卡存在。
- `GameMode`：服务器权威规则，只在服务器存在。
- `GameState`：同步给客户端的全局游戏状态。
- `PlayerController`：玩家输入和控制，客户端有自己的，本机控制重要。
- `PlayerState`：玩家公共状态，可复制给其他客户端。
- `Pawn`：可被 Controller 控制的实体。
- `Character`：带角色移动组件的人形 Pawn。

### 最小例子
```cpp
// 战斗 Demo 中的一个职责划分
// Character：移动、技能输入、表现触发
// HealthComponent：生命值和伤害
// PlayerState：需要同步给其他人的玩家分数/击杀数
// GameState：当前战斗波次或全局倒计时
// GameMode：服务器上决定战斗开始/结束规则
```

### 常见误区
- 误区：GameMode 可以在客户端读。
  - 更准确：GameMode 只在服务器存在。
- 误区：PlayerController 适合存所有玩家都要看到的数据。
  - 更准确：公共玩家状态更适合 PlayerState。

### 面试表达
- 我会按网络职责划分 Gameplay Framework：GameMode 管服务器规则，GameState 放需要同步的全局状态，PlayerState 放玩家公共状态，PlayerController 处理本地玩家控制，Pawn/Character 是被控制的实体。

### 练习题
- 击杀数应该放 PlayerController 还是 PlayerState？为什么？
- 战斗波次倒计时应该放 GameMode 还是 GameState？为什么？

## 机制 14：网络复制基础：Authority、Ownership、RPC、RepNotify

### 为什么重要
- 你有多人 Demo 经验，但 Ownership、`stat net`、复制条件还需要补。
- ARPG Demo 第二阶段要做联机亮点，必须提前整理。

### 源码入口
- 本地源码：
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Classes\GameFramework\Actor.h`
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Public\Net\UnrealNetwork.h`
- 官方文档：
  - https://dev.epicgames.com/documentation/unreal-engine/networking-overview-for-unreal-engine
  - https://dev.epicgames.com/documentation/unreal-engine/replicate-actor-properties-in-unreal-engine
  - https://dev.epicgames.com/documentation/unreal-engine/remote-procedure-calls-in-unreal-engine

### 核心讲解
- Authority：谁对状态有权威，通常服务器对 replicated Actor 有 Authority。
- Ownership：影响 Client/Server RPC 的调用权限，通常和 PlayerController/连接有关。
- RPC：
  - Server RPC：客户端请求服务器执行。
  - Client RPC：服务器通知 owning client。
  - Multicast RPC：服务器通知所有相关客户端。
- RepNotify：属性复制到客户端后触发响应函数，适合更新表现。
- 权威状态修改应在服务器，客户端可做预测和表现。

### 最小例子
```cpp
UPROPERTY(ReplicatedUsing=OnRep_Health)
float Health;

UFUNCTION()
void OnRep_Health();

UFUNCTION(Server, Reliable)
void ServerCastSkill(int32 SkillIndex);
```

### 常见误区
- 误区：客户端改 replicated 变量会自动同步给服务器。
  - 更准确：复制通常是服务器到客户端，客户端要通过 Server RPC 请求。
- 误区：Multicast 可以从客户端直接调用让所有人看到。
  - 更准确：Multicast 应由服务器调用才会广播给客户端。
- 误区：RPC 不触发一定是网络坏了。
  - 更常见：Actor 不复制、没有 Owner、调用端不对、函数声明不对。

### 面试表达
- UE 网络是服务器权威模型。重要状态由服务器修改并复制到客户端。客户端输入通常通过 Server RPC 请求服务器，服务器验证后修改状态，再通过属性复制或 Multicast 同步表现。Ownership 会影响 RPC 调用权限，`RepNotify` 常用于客户端响应 replicated 属性变化。

### 练习题
- 客户端释放技能，想让所有人看到效果，流程应该怎么走？
- RPC 不触发时，你会检查哪 5 件事？

## 机制 15：UMG 与 Gameplay 通信

### 为什么重要
- 你对 UI 兴趣低，但战斗 Demo 至少需要血条、技能冷却、敌人血量。
- UI 不应强耦合 Gameplay，面试也会看通信方式。

### 源码入口
- 本地源码：
  - `D:\soft\UE_5.7\Engine\Source\Runtime\UMG\Public\Blueprint\UserWidget.h`
  - `D:\soft\UE_5.7\Engine\Source\Runtime\UMG\Public\Components`

### 核心讲解
- Widget 负责显示和输入，不应承担核心战斗逻辑。
- Gameplay 组件维护数据。
- 数据变化通过委托/Event 通知 UI。
- UI 初始化时绑定，销毁时解绑。
- 对频繁变化数据，避免无意义每帧绑定刷新。

### 最小例子
```cpp
// HealthComponent 广播
OnHealthChanged.Broadcast(CurrentHealth, MaxHealth);

// Widget 监听后更新血条
void UMyHUDWidget::HandleHealthChanged(float Current, float Max)
{
    HealthBar->SetPercent(Current / Max);
}
```

### 常见误区
- 误区：Widget 每帧主动查角色血量就行。
  - 更准确：简单项目能跑，但事件驱动更清晰，也更利于性能。
- 误区：UI 可以直接修改战斗核心数据。
  - 更准确：UI 应发出意图，Gameplay 系统决定是否执行。

### 面试表达
- 我会让 Gameplay 组件维护权威数据，UI 只负责展示。比如 HealthComponent 在血量变化时广播委托，Widget 订阅后更新血条。这样 UI 不需要知道伤害计算细节，也避免强耦合。

### 练习题
- 技能冷却 UI 应该 Tick 计算，还是由技能系统通知？可以怎么折中？
- Widget 销毁时为什么要解绑事件？

## 机制 16：性能入门：Tick、Stat、Insights

### 为什么重要
- 你性能工具经验偏弱，但社招会问“怎么定位卡顿”。
- 不需要本阶段深挖渲染，但要有基本定位流程。

### 源码入口
- 本地源码：
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Classes\GameFramework\Actor.h`
  - `D:\soft\UE_5.7\Engine\Source\Runtime\Engine\Public\TimerManager.h`
- 官方文档：
  - https://dev.epicgames.com/documentation/unreal-engine/unreal-insights-in-unreal-engine

### 核心讲解
- `stat unit`：看 Game/Draw/GPU 粗粒度耗时。
- `stat game`：看 GameThread 相关统计。
- `stat slate`：看 UI。
- `stat net`：看网络。
- Unreal Insights：记录和分析更细粒度性能数据。
- 常见优化方向：
  - 减少无意义 Tick。
  - Timer/事件驱动替代轮询。
  - 避免频繁创建销毁对象。
  - 控制 UI 刷新频率。

### 最小例子
```cpp
// 不推荐：每帧检查冷却是否结束
void Tick(float DeltaSeconds);

// 更清晰：冷却结束时 Timer 回调
GetWorld()->GetTimerManager().SetTimer(CooldownHandle, this, &AMyCharacter::OnCooldownEnd, Cooldown, false);
```

### 常见误区
- 误区：性能优化靠感觉。
  - 更准确：先测量，再定位，再修改，再对比。
- 误区：Tick 一定不好。
  - 更准确：必要 Tick 可以用，但要知道为什么每帧都需要。

### 面试表达
- 我定位性能问题会先用 `stat unit` 看瓶颈在 Game、Draw 还是 GPU，再用 `stat game`、`stat slate`、`stat net` 或 Unreal Insights 细看。优化时先改高频和可测量的问题，比如无意义 Tick、频繁分配、UI 过度刷新。

### 练习题
- 如果战斗 Demo 掉帧，你第一步用什么命令看？
- 技能冷却为什么不一定需要 Tick？

## 总索引：后续要继续补的机制
- [ ] Enhanced Input：`InputAction`、`InputMappingContext`、Trigger、Modifier。
- [ ] GAS 基础：Ability、Attribute、Effect、Tag。
- [ ] AI：AIController、Behavior Tree、Perception、StateTree。
- [ ] 碰撞和命中检测：Channel、Trace、Sweep、Overlap。
- [ ] 动画：AnimBP、Montage、Notify、Root Motion。
- [ ] SaveGame。
- [ ] Asset Manager。
- [ ] Module/Build.cs 深入。
- [ ] Packaging 和运行时/编辑器模块边界。
