# 我的UE基础知识库

1.它是什么？（基本定义）

2.它解决什么问题？（存在意义）

3.它是如何工作的？（基本原理）

4.如何使用它？（实际应用）

5.有什么注意事项？（坑和经验）

## UObject

### 1. 它是什么？（基本定义）

UObject是UE独有且是所有UE类中的基类，几乎所有UE中的类都直接或间接的继承自UObject

### 2. 它解决什么问题？（存在意义）

1. 垃圾回收：UObject不像普通C++的类每次创建都需要进行new/delete进行内存管理，防止内存泄漏，UObject它可以自动垃圾回收，可以自动清理不再引用的对象
2. 序列化:可以将类保存到磁盘中,并在需要时重新拿出来,比如我放置一个Actor到地图的一个位置并保存,此时Actor的（Location,Rotation,Scale...)登信息被序列化存储在磁盘中
3. 编辑器集成:这个就十分明显，创建的C++类可以在虚幻引擎中对其暴露的数据进行修改和蓝图操作
4. 网络复制：对于多人游戏开发来说，我们想要实现网络复制更加简单，一般需要对想要上传的数据和函数进行标记"复制"，就可以实现数据在客户端和服务端之间的传递,比如RPC可以实现客户端和服务端数据传输
5. 反射系统:反射系统可以使C++与蓝图紧密相连，且可以使用字符串来调用类中的函数，使用字符串来生成相应类的对象，可以遍历类中属性(蓝图中修改C++属性就和这个有关)

### 3. 它是如何工作的？（基本原理）

1. 垃圾回收：UE中有一个自己维护的Object实例对象图，垃圾回收器会周期性的进行垃圾回收，每次会从一组根节点进行遍历进行访问，没有被访问的Object会被标记(被孤立了和其他的没有联系)，并在合适的时机进行回收
2. 序列化:保存时Object的信息会被序列化并保存起来，当打开时电脑会读取这些文件并进行反序列化来获取这些Object的信息
3. 网络复制:网络复制有属性复制(Replicated)一般用于服务端传递信息到客户端是单向的。还有一直是RPC可以在服务端和客户端双向传输。
4. 反射系统:每个UClass都是一个UObject，其中存储着多种C++元数据,这些元数据包括类名，父类，属性(UPROPERTY)和它含有的函数(UFUNCTION)以及多种信息，通过UClass我们就可以动态的创建对象，获取和设置属性值，调用函数而不需要知道具体的类.

### 4. 如何使用它？（实际应用）

类的创建，在引擎中可以创建C++类，选择对应的父类即可创建子类，不要使用中文
引擎会帮你生成对应的.h和.cpp

```c++
#pragma once

#include "CoreMinimal.h"
#include "MyFirstObject.generated.h" // 必须包含生成的头文件，注意命名与头文件对应

UCLASS(Blueprintable,BlueprintType)// 这个类可以在蓝图中使用，并可以作为蓝图变量类型
class MYTESTPROJECT_API UMyFirstObject : public UObject
{
	GENERATED_BODY()// 必须包含，通常放在类定义的最开始
   	
    public:
    //可以在这写你需要的属性和函数
    
    //属性
    UPROPERTY()//可以在其中写不同的东西来增加属性和方法的功能
    float Health;
	
    //方法
    UFUNCTION()
    void Heal();
};
```

当需要创建Object对应的实例时

```c++
MyFirstObject* Object = NewObject<MyFirstObject>();
```

### 5. 有什么注意事项？（坑和经验）

1. **不要使用`new`和`delete`**：UObject必须由引擎的垃圾回收系统管理。

2. **使用UPROPERTY()保持引用**：如果你有一个指向其他UObject的指针，并且你希望它不被垃圾回收，必须用UPROPERTY()标记。否则，垃圾回收器会认为这个对象没有被引用，从而将其销毁。

   ```c++
   //正确
   UPROPERTY()
   MyFirstObject* Object;
   
   //错误
   //这个指针指向的对象可能会在你使用时被回收
   MyFirstObject* Object；
   ```

3. 使用`NewObject`来创建UObject实例。对于Actor，使用`SpawnActor`。

4. 必须包含生成的头文件（例如`#include "MyObject.generated.h"`），并且必须放在头文件最后包含的头文件之前（通常放在最下面，但要在其他头文件之后）。

5. 如果你的UObject类在一个模块中，而另一个模块要使用它，需要在模块的Build.cs文件中添加依赖。

   ```c#
   PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "EnhancedInput" });
   ```

6. 必须正确使用宏：`UCLASS()`、`GENERATED_BODY()`、`UPROPERTY()`、`UFUNCTION()`等。注意，`GENERATED_BODY()`必须放在类定义的最开始。

7. 按照UE的约定，继承自UObject的类以`U`为前缀，继承自Actor的类以`A`为前缀，结构体一般用大写`F`，继承自Component的类以`U`为前缀

8. 如果对象需要在网络上复制，必须正确设置复制属性和函数。在UCLASS中设置`Replicated`，并在类中重写`GetLifetimeReplicatedProps`函数。

   ```c++
   UPROPERTY(Replicated)
   float Health;
   
   virtual void GetLifetimeReplicatedProps( TArray< class FLifetimeProperty > & OutLifetimeProps ) const override;
   ```

9. 垃圾回收是非确定性的，你无法知道一个对象具体何时被销毁。当一个UObject被销毁时，它的析构函数会被调用。但通常你不需要自己写析构函数，除非你手动分配了非UObject的内存（这时需要手动释放）。

## Actor

### 1. 它是什么？（基本定义）

Actor是Object的子类,所有能放进关卡(Level)中的Object都是Actor，Actor相比Object多了Transform(Location,Rotation,Scale),还有一些函数如：BeginPlay(),Tick(),Destroy(),组件系统

### 2. 它解决什么问题？（存在意义）

1. 变化(Transform) 记录了物体的位置、旋转、缩放信息，对于物体的位置摆放更加方便

2. 增加了生命周期 BeginPlay(),Tick(),Destroy()来进行管理Actor让Actor活起来了

3. 组件(Component) 引入组件系统，可以使各个组件各自执行自己的功能，实现功能的复用，避免继承的复杂

   ```c++
   // 传统继承方式导致复杂的类层次
   class ACharacter : public AActor {};
   class AMyCharacter : public ACharacter {};
   class AMyCharacterWithWeapon : public AMyCharacter {};
   class AMyCharacterWithWeaponAndMagic : public AMyCharacterWithWeapon {};
   class AMyCharacterWithWeaponAndMagicAndPets : public AMyCharacterWithWeaponAndMagic {};
   // ... 类爆炸！
   
   
   // 通过添加不同的组件来组合功能
   MovementComponent = CreateDefaultSubobject<UFloatingPawnMovement>(TEXT("MovementComp"));
   HealthComponent = CreateDefaultSubobject<UHealthComponent>(TEXT("HealthComp"));
   ManaComponent = CreateDefaultSubobject<UManaComponent>(TEXT("ManaComp"));
   InventoryComponent = CreateDefaultSubobject<UInventoryComponent>(TEXT("InventoryComp"));
           
   // 设置根组件
   RootComponent = CreateDefaultSubobject<USceneComponent>(TEXT("RootComp"));
   ```

### 3. 它是如何工作的？（基本原理）

1. Transform 每个Actor都有RootComponent，其中Transform信息就存储在RootComponent中Actor的其他Component变换是相对RootComponent的
2. 生命周期：Spawn -> BeginPlay -> Tick -> 销毁流程 -> Destroy
3. Component: Actor本身就是容器，所有的功能都通过对应的Component进行实现

### 4. 如何使用它？（实际应用）

类的创建和Object创建一样

在Actor中需要实现对应的组件，需要在.h文件中声明，并在构造函数中去创建实例,CreateDefaultSubobject创造的实例是和Actor相同生命周期的

```c++
private:
	UPROPERTY(EditAnywhere) //可以在任何地方去修改
	UStaticMeshComponent* StaticMeshComponent;
```

关于根节点的使用一般使用什么功能都不负责的只要负责Transform的最好方便管理，所有使用USceneComponent作为根节点

```c++
AMyFirstActor::AMyFirstActor()
{
    //是否开启Tick功能,一般如果不使用Tick()功能可以改为false
	PrimaryActorTick.bCanEverTick = true;

    //创建根节点
	RootComponent = CreateDefaultSubobject<USceneComponent>("RootComponent");
    //创建静态网格组件
	StaticMeshComponent = CreateDefaultSubobject<UStaticMeshComponent>("StaticMeshComponent");
	//将静态网格组件附加到根节点上
	StaticMeshComponent->SetupAttachment(RootComponent);
}
```



我写了一个可以将Actor变为可以闪烁旋转且颜色变化的Actor

```c++
void AMyFirstActor::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	//物体旋转
	RootComponent->AddWorldRotation(DeltaTime*RotationSpeed);

	//光亮闪烁
	Timer += DeltaTime * OscillationSpeed;
	if (bIntensityChange)
	{
		PointLightComponent->SetIntensity(FMath::Sin(Timer)*0.5f*(MaxValue - MinValue) + (MaxValue + MinValue)*0.5f);
	}
	
	if (bColorChange)
	{
		ColorTimer = FMath::Fmod(Timer * ColorChangeSpeed,1.f);
		//颜色变化
		PointLightComponent->SetLightColor(CurveLinearColor->GetLinearColorValue(ColorTimer));
	}
}
```

### 5.有什么注意事项？（坑和经验）

1. **【性能】谨慎使用 Tick**：

   - Tick 每帧都会调用，Actor的Tick功能十分吃性能建议在不使用时关闭
   - 解决方案：禁用不需要的 Tick（`PrimaryActorTick.bCanEverTick = false`），使用定时器或事件驱动

2. **组件初始化顺序**：

   - 在构造函数中创建组件(`CreateDefaultSubobject<>()`)
   - 不能在构造函数中访问其他Actor，因为此时其他Actor可能未被创建,可以在`BeginPlay()` 中访问其他 Actor，因为BeginPlay是当所有Actor都创建后执行

3. **Actor 生命周期理解**：

   - `Destroy()` 不是立即销毁，而是在当前帧结束时标记为待销毁，
   - Destroy只是发起摧毁请求，Destroyed是摧毁的前一瞬间，在这些过程中Actor还都是有效的,一般我们在Destroyed中释放手动开辟的内存，放置内存泄漏

   - 销毁流程: Destroy() → 标记待销毁 → 当前帧结束 → Destroyed() → 垃圾回收 → 内存释放
   - 使用 `IsValid()` 检查 Actor 是否有效，而不是直接判断 `!= nullptr`，因为指针判断 `!= nullptr`只是判断它有没有指向地址，只要地址存在就不为空，但地址存在不代表地址原本的Actor有效.意思就是Actor只剩一个空壳了,你指针指向的是这个空壳但是它内在不见了

4. **网络复制注意事项**：

   - 只有服务器可以生成复制的 Actor,意思是在客户端生成的Actor无法自动上传到服务端,如果是在服务端生成的Actor且复制了服务端会自动复制到客户端
   - 需要被复制的Actor要在其构造函数中将复制打开 bReplicates = true;
   - 复制的属性需要在 `GetLifetimeReplicatedProps` 中注册
   - RPC（远程过程调用）有严格的规则

5. **内存管理**：

   - Actor 被销毁后，其组件也会自动销毁
   - 所有`UObject`指针成员必须用UPROPERTY()

6. **变换同步问题**：

   - 移动 Actor 时，应该移动其 Root Component
   - 如果要网络复制移动，需要设置 `SetReplicateMovement(true)`,Actor的移动复制需要单独开启如果不启用服务端移动不会自动复制给客户端和 bReplicates是一样的

7. **Actor的生成**

   - Actor生成可以使用SpawnActor，但是如果在C++中直接使用 

   - `SpawnActor<AMyFirstActor>` 生成的是**C++原生类**，不会调用你在蓝图中做的设置。

   - 需要添加一个指针来添加蓝图类 `TSubclassOf<AMyFirstActor> ActorBlueprintClass;`

     ```c++
     AMyFirstActor* MyFirstActor = GetWorld()->SpawnActor<AMyFirstActor>(
                 ActorBlueprintClass,  // 使用蓝图类，不是C++类
                 SpawnLocation,
                 GetActorRotation(),
                 SpawnParams
             );
     ```

     

   

## Casting

### 1.它是什么？（基本定义）

UE内置的一个检查对象是否属于某个特定类型，如果检查成功返回目标类型指针，则类型转换成功并将其作为这个特定类型看待

### 2.它解决什么问题？（存在意义）

1. **接口隔离**：只在需要时才访问特定类型的接口，保持代码的清晰性
2. **事件处理**：在通用事件（如碰撞）中区分不同类型的参与者
3. **避免过度设计**：不需要为所有可能的交互创建复杂的接口或继承关系

### 3.它是如何工作的？（基本原理）

### 4.如何使用它？（实际应用）

比如你有一个碰撞检测的类型检测到对应类型即造成伤害

```c++
//检测触发碰撞的是否是APlayer类型或者其子类
if(APlayer* Player = Cast<APlayer>(OverlapActor))
{
    //检测成功即执行对应功能
    Player->CasuedDamaged();
}
```

### 5.有什么注意事项？（坑和经验）

1. **【性能】避免每帧转换**：
   - 类型转换有开销，不要在 Tick 中频繁转换同一对象
   - 解决方案：缓存转换结果或使用其他标识方法
2. **继承关系理解**：
   - 只能向上或向下转换在同一个继承链中的类型
   - `Csat`不能转化兄弟类比如(`ClassA`,`ClassB`都是继承与`ClassBase`)
   - `Cast<ClassA>(ClassB)` 的返回值为`nullptr`
   - 不能在不相关的类型之间转换
3. **空指针检查**：
   - 转换失败返回 `nullptr`，一定要检查

## Component

### 1.它是什么？（基本定义）

Component是一个能重用的可以添加到Actor上的，附加特定功能或行为的模块

### 2.它解决什么问题？（存在意义）

1. **打破复杂的继承链**：减少了复杂的继承
2. **功能复用**：同一组件可以在不同类型的 Actor 间共享
3. **动态组合**：可以在运行时随用随开，随禁随停
4. **职责分离**：每个组件负责自己的功能，让结构看的更加清晰,不同程序员可以独立开发不同组件

### 3.它是如何工作的？（基本原理）

1. **组件层次结构**：附加在Actor上使用
2. **生命周期管理**：在Actor构造函数中创建，随Actor销毁而销毁,Component具备自己的`BeginPlay(),TickComponent(),EndPlay()`
3. **变换传播**：一般来说将`SceneComponent`作为为根节点其他组件为`SceneComponent`的子节点,移动 RootComponent 会移动所有附加的子组件
4. **组件查找**：

- Actor 维护组件列表
- 可以通过类型或标签查找组件

### 4.如何使用它？（实际应用）



在这之前我写了一个可以将Actor变为可以闪烁旋转且颜色变化的Actor用到了PointLightComponent,并在Actor的Tick文件中写了一大堆关于PointLightComponent的使用，使得Actor的Tick函数承担了太多功能，所有我决定将PointLightComponent的功能分离出去独立封装

```c++
#include "MyPointLightComponent.h"

#include "Curves/CurveLinearColor.h"

UMyPointLightComponent::UMyPointLightComponent()
{
	PrimaryComponentTick.bCanEverTick = true;
}


void UMyPointLightComponent::BeginPlay()
{
	Super::BeginPlay();
	StartIntensity = Intensity;
}

void UMyPointLightComponent::UpdateLightIntensity()
{
	const float SinValue = FMath::Sin(Timer * OscillationSpeed);
	const float NewIntensity =SinValue*0.5f*(MaxValue - MinValue) + (MaxValue + MinValue)*0.5f;
	SetIntensity(NewIntensity);
}

void UMyPointLightComponent::UpdateLightColor()
{
	if (CurveLinearColor)
	{
		const float ModValue = FMath::Fmod(Timer * ColorChangeSpeed,1.f);
		const FLinearColor NewLinearColor = CurveLinearColor->GetLinearColorValue(ModValue);
		SetLightColor(NewLinearColor);
	}
}

void UMyPointLightComponent::TickComponent(float DeltaTime, ELevelTick TickType,
                                           FActorComponentTickFunction* ThisTickFunction)
{
	Super::TickComponent(DeltaTime, TickType, ThisTickFunction);
	Timer += DeltaTime;
	//光亮闪烁
	if (bIntensityChange)
	{
		UpdateLightIntensity();
	}	
	//颜色变化
	if (bColorChange)
	{
		UpdateLightColor();
	}
}
```

而在Actor中只需要创建UMyPointLightComponent即可

### 5.有什么注意事项？（坑和经验）

1. **【重要】组件创建时机**：

   - 组件**必须**在 Actor 的**构造函数**中创建
   - 使用 `CreateDefaultSubobject<>()`，不能在 `BeginPlay()` 或其他地方创建默认组件

2. **组件查找性能**：

   - 避免在 Tick 中频繁使用 `FindComponentByClass()` 或 `GetComponentByClass()`

     ```c++
     //通过GetOwner()寻找其他组件
     GetOwner()->FindComponentByClass<UDamageFeedbackComponent>();
     ```

   - 解决方案：在 `BeginPlay()` 中缓存组件指针

3. **SceneComponent 的层级关系**：

   - 设置父子关系使用 `SetupAttachment()`（构造函数中）或 `AttachToComponent()`（运行时）
   - 错误的层级设置会导致变换问题

4. **网络复制**：

   - 组件属性需要单独设置复制
   - 在组件中重写 `GetLifetimeReplicatedProps`
   - 组件 RPC 需要正确的网络角色检查

5. **组件依赖关系**：

   - 如果组件A对组件B有依赖，即在组件A中使用了组件B，通过下面函数即可访问组件B在这种情况下需要先`声明`且在`构造函数`中先创建`组件B`

     ```c++
     GetOwner()->FindComponentByClass<UDamageFeedbackComponent>();
     ```

   - 在 `BeginPlay()` 中验证依赖关系：不能在A的构造函数中访问组件B由Actor的功能可知构造函数中组件可能还不存在，因为BeginPlay是当所有Component都创建后执行

6. **组件的手动创建**:

   - 组件的手动创建需要调用`RegisterComponent()`,如果不调用这个组件不会参与游戏逻辑，不会调用BeginPlay，不会Tick

     ```c++
     UMyComponent* Component = NewObject<UMyComponent>(this);
     Component->RegisterComponent();
     ```

7. **内存管理**：

   - 组件随 Actor 自动销毁，一般不需要手动管理

   - 动态添加的组件要小心生命周期,需要手动销毁

     ```c++
     void DemonstrateDestruction()
     {
         // 创建动态组件
         UMyComponent* DynamicComp = NewObject<UMyComponent>();
         DynamicComp->RegisterComponent();
         
         // 方法1：手动立即销毁
         DynamicComp->DestroyComponent();
         // 组件立即被标记为待销毁
         
         // 方法2：延迟销毁
         DynamicComp->ConditionalBeginDestroy();
         
         // 方法3：通过Owner销毁（当Actor销毁时）
         // 如果组件有正确的Outer，会在Actor销毁时自动清理
         //NewObject<UMyComponent>(this);中的this就是Outer，这个就会自动销毁
     }
     ```

     

8. **与蓝图的交互**：

   - 使用 `BlueprintSpawnableComponent` 使组件在蓝图中可添加

     ```c++
     UCLASS(ClassGroup=(Custom), meta=(BlueprintSpawnableComponent))
     ```

## Pawn

### 1.它是什么？（基本定义）

他是Actor的子类，是`可控制实体`的基类，玩家和AI在世界的“化身”

### 2.它解决什么问题？（存在意义）

1. **控制器分离**： 使控制器(Controller)和被控制的对象(Pawn)分离，就如同Actor和Component的关系，让Pawn专注于能力，Controller专注于决策
2. **输入抽象**：提供统一的输入处理框架，支持玩家输入和 AI 指令（玩家和AI只需要调用Pawn相同指令即可)
3. **角色蓝图**：作为复杂角色（Character）的简化版本，适用于非人形实体

### 3.它是如何工作的？（基本原理）

1. **控制器系统**：通过Controller调用Possess(Pawn)来获取控制权`UnPossess()`来解除控制

2. **输入流程**：键盘输入的按键->Controller->`InputComponent/UEnhancedInputComponent`(绑定`UInputAction`（在蓝图中绑定按键）,和触发函数)->输入组件绑定函数触发Pawn中的函数

3. **网络复制模型**：

   - 服务器拥有权威的 Pawn 状态：服务端拥有对Pawn最终状态的决定权

   - 客户端通过网络复制同步 Pawn 的状态(被复制的的变量会自动复制到客户端)

   - 控制权在服务器和客户端间正确传递

     ```c++
     ROLE_Authority - 服务器，可以做任何事
     ROLE_AutonomousProxy - 本地控制客户端，可以发送输入
     ROLE_SimulatedProxy - 其他客户端，只能观看复制
     ```

4. **移动组件集成**：

   - 默认包含基本的移动能力
   - 可以通过 `PawnMovementComponent` 增强移动逻辑

5. **摄像机管理**：

   - 处理玩家视角的摄像机
   - 支持各种摄像机模式（第一人称、第三人称等）

### 4.如何使用它？（实际应用）

### 5.有什么注意事项？（坑和经验）

1. **设置根组件**：在Actor中我们通常使用USceneComponent作为根组件，但是在Pawn中我们一般使用Collision作为根节点。
2. **组件的创建:**组件的创建一定要在构造函数中创建，否则不会生效(亲测)
3. **模拟物理：**启动模拟物理后，需要设置碰撞检测，默认为不考虑物理碰撞的，需要设置为检测物理碰撞
4. **控制器关系管理**：
   - Pawn 被销毁时，控制器会自动调用 `UnPossess()`
   - 手动调用 `Possess()` 时要确保之前的控制关系已清理
   - 使用 `GetController()` 安全地访问控制器
5. **【重要】输入处理时机**：
   - 输入绑定只能在 `SetupPlayerInputComponent` 中进行
   - 只有被 `PlayerController` 控制的 Pawn 才能接收输入
   - 使用 `EnableInput()` 和 `DisableInput()` 控制输入开关
   - 在设置按键绑定`InputAction`时，不止需要在`InputMappingContext`中设置`InputAction`，同样需要在`InputAction`中填入IA_行为(在蓝图中完成)
6. **网络复制挑战**：
   - 移动同步需要正确设置网络角色(由于使用的是PawnMovementComponent需要进行手动同步，所以客户端的更新不会上传服务端)
   - 客户端预测移动可能带来同步问题
   - 复杂的状态需要在服务器和客户端间正确复制
7. **移动组件选择**：
   - 基础 Pawn 移动比较简单，适合简单场景
   - 复杂移动需求考虑使用 `CharacterMovementComponent`
   - 自定义移动逻辑要注意网络同步

## Character

### 1.它是什么？（基本定义）

Character是UE专门为人形角色设计的基类
内置了`CharacterMovementComponent`提供了自动网络复制

包含**CapsuleComponent**作为碰撞体，**SkeletalMeshComponent**作为网格体

### 2.它解决什么问题？（存在意义）

Character解决了人形角色在3D世界的移动和交互

1. **动画集成**：为骨骼动画提供完整的移动基础
2. **网络同步**：为多人游戏提供可靠的角色移动同步

### 3.它是如何工作的？（基本原理）

```
ACharacter
├── UCapsuleComponent (RootComponent) - 碰撞胶囊体(对碰撞的检测)
├── USkeletalMeshComponent (Mesh) - 骨骼网格体(动画的承载体)
└── UCharacterMovementComponent - 移动逻辑核心
```

### 4.如何使用它？（实际应用）

为了实现一个Character的正常控制功能我需要
PlayerController(实现按键的输入和绑定)

CharacterMovementComponent(实现具体移动等功能的实现)

Character(中转站，调控Controller和MovementComponent的连接)

### 5.有什么注意事项？（坑和经验）

**基类的替换：**使用了`ObjectInitializer`来替换基类注意如果需要使用相应的类的对象需要进行Cast

```c++
AMyCharacter::AMyCharacter(const FObjectInitializer& ObjectInitializer):Super(ObjectInitializer.SetDefaultSubobjectClass<UMyCharacterMovementComponent>(CharacterMovementComponentName))
{
	PrimaryActorTick.bCanEverTick = true;

	MyMovementComponent = Cast<UMyCharacterMovementComponent>(GetCharacterMovement());
}

```

**人物的旋转：**在设置`InputAction`时一定要改Value Type(在这次中由于没改导致一直只有一个轴生效),其次人物旋转使用`AddControllerYawInput`...，如果使用了其实对于Yaw轴是仍然不生效的需要改变`SpringArm`中的`UsePawnControllerRotation` ,如果不想角色跟着旋转需要关闭角色的`UseControllerRotationYaw`...

**自由坠落：**一般Actor需要启动模拟物理，但在Character中如果启用了Capsule的模拟物理则会导致角色一碰就倒，正确方法是开启`MovementComponent`的Tick更新

**网络同步问题**：

- 移动同步使用客户端预测+服务器校正
- 复杂移动逻辑需要在服务器和客户端都执行
- 使用 `ROLE_Authority` 检查确保只在服务器执行关键逻辑

发现为什么没有碰撞效果，原因是Move函数应该使用AddMovementInput，而不是AddActorWorldLocation,在使用AddMovementInput后变成了滑动(原因是我之前设置行走最大度数为0，则一直判断无法行走)

**人物移动后退：**在人物后退时出现了抽搐，原因`bOrientRotationtoMovement`(人物会朝加速方向旋转)，人物动画的数据最好在事件蓝图中完成

## Controller

### 1.它是什么？（基本定义）

**Controller** 是 Unreal Engine 中负责**控制 Pawn** 的类，是玩家输入或 AI 逻辑与 Pawn 之间的桥梁。

可以独立于Pawn单独存在

### 2.它解决什么问题？（存在意义）

Controller 系统解决了控制逻辑与游戏实体分离的核心架构问题：

1. **控制权与实体的分离**：Pawn 专注于"是什么"，Controller 专注于"谁在控制"
2. **输入处理抽象**：提供统一的输入处理框架，支持多种输入设备
3. **AI 行为管理**：为 AI 提供决策和行为执行的框架
4. **状态持久化**：Controller 可以比 Pawn 存活更久，支持角色重生机制

### 3.它是如何工作的？（基本原理）

Controller可以通过使用Possess和UnPossess来和Pawn建立和断开连接
且Controller是最先接收输入的

### 4.如何使用它？（实际应用）

### 5.有什么注意事项？（坑和经验）

1. **Controller-Pawn 生命周期**：
   - Controller 可以比 Pawn 存活更久
   - Pawn 被销毁时自动调用 `UnPossess()`
   - 手动调用 `Possess()` 时要确保清理之前的控制关系
2. **【重要】网络权限检查**：
   - 关键游戏逻辑必须在服务器执行
   - 使用 `HasAuthority()` 检查服务器权限
   - 客户端预测的操作需要服务器验证
3. **网络同步挑战**：
   - `PlayerController` 只在所属客户端和服务器存在
   - AIController 在服务器和所有客户端复制
   - RPC 调用要注意网络角色和可靠性
4. **HUD 和 UI 管理**：
   - PlayerController 负责创建和管理 HUD
   - UI 输入模式设置要注意游戏状态
   - 多人游戏中每个客户端有自己的 HUD 实例



## PlayerState

### 1.它是什么？（基本定义）

**PlayerState** 是 Unreal Engine 中用于存储和同步**玩家相关数据**的类，是多人游戏中玩家状态管理的核心。

- 每个玩家在游戏中都有一个对应的 PlayerState 实例
- 存储玩家的游戏相关数据：得分、生命值、资源、状态等
- 与 PlayerController 关联，但比 PlayerController 存活更久
- 在**所有客户端**和服务器之间同步

### 2.它解决什么问题？（存在意义）

PlayerState 解决了多人游戏中玩家数据管理和同步的关键问题：

1. **玩家数据持久化**：在玩家死亡、重生、重新连接时保持数据不变
2. **数据与控制的分离**：将玩家数据从控制逻辑中分离出来
3. **全客户端同步**：所有玩家都能看到其他玩家的状态信息

### 3.它是如何工作的？（基本原理）

1. **生命周期管理**：
   - 玩家加入游戏时创建
   - 玩家离开游戏时销毁
   - 在整个游戏会话期间持续存在
2. **网络复制机制**：
   - 服务器是 `PlayerState` 的权威版本
   - 所有客户端都有所有玩家的 `PlayerState` 副本
   - 通过属性复制同步数据变化
3. **数据聚合**：
   - `GameState` 可以访问所有 `PlayerState` 来聚合游戏数据
   - 为记分板、统计等提供数据基础
4. **与控制器关系**：

   - `PlayerController`拥有`PlayerState`，但是`PlayerState`的存活时间比`PlayerController`更旧(原因是为了再玩家退出游戏，重新进入时读取数据，可以通过`GameState`来获取所有的`PlayerState`来重新复用)

### 4.如何使用它？（实际应用）

### 5.有什么注意事项？（坑和经验）

1. **【重要】网络权限检查**：

   - 数据修改**必须**在服务器进行
   - 使用 `HasAuthority()` 检查服务器权限
   - 客户端修改需要通过 Server RPC

2. **数据一致性**：

   - 确保所有客户端看到相同的数据状态
   - 使用 `ReplicatedUsing` 确保状态同步时的回调
   - 注意网络延迟可能导致的数据暂时不一致

3. **性能优化**：

   - 避免在 PlayerState 中存储大量数据

   - 使用条件复制减少网络流量：

     ```c++
     DOREPLIFETIME_CONDITION(AMyPlayerState, SomeProperty, COND_OwnerOnly);
     ```

4. **生命周期管理**：

   - PlayerState 在玩家离开游戏时自动销毁
   - 不要手动销毁 PlayerState
   - 在玩家重新连接时可能创建新的 PlayerState

5. **与 GameState 的协作**：

   - GameState 应该聚合 PlayerState 数据
   - 避免在 PlayerState 中存储全局游戏状态
   - 使用 GameState 进行跨玩家数据比较

## GameMode



## EnhancedInput

## HUD

## RPC



## Replicated



## GAS

## GamePlayTag

## GamePlayEffect

## GamePlayAbility



在我的GAS中我
我在增强输入中进行了按键和函数的绑定，因为我需要知道按下的按键是什么(正常只会在你按下时输出1)通过对Tag的判断知道了按下的Tag，再通过Tag进行分离，如果是触发Ability的则通过对应的Tag触发相应的Ability，因为再Ability中有相应的InputTag

Ability的Tag有什么用 AbilityTag是查看Character是否具备该能力
Effect是用来修改Attribute的基本所以的属性变化都由Effect执行

## Tick()

Actor/Component的Tick

```c++
USTRUCT()
struct FMyTickFunction : public FTickFunction
{
	GENERATED_USTRUCT_BODY()

public:
	ENGINE_API virtual void ExecuteTick(float DeltaTime, ELevelTick TickType, ENamedThreads::Type CurrentThread, const FGraphEventRef& MyCompletionGraphEvent) PURE_VIRTUAL(,);
};

template<>
struct TStructOpsTypeTraits<FMyTickFunction> : public TStructOpsTypeTraitsBase2<FMyTickFunction>
{
	enum
	{
		WithCopy = false,
		WithPureVirtual = true
	};
};
```

还需要在BeginPlay中激活Tick的功能





Timer的Tick





FTickableGameObject的Tick

```c++
class UMyObject :public UObject,public FTickableGameObject
{
    GENERATED_BODY()
public:
    virtual void Tick(float DeltaTime) override;
}
```

直接继承即可调用

