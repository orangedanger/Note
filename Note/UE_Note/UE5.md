# **UE5**

## 一、UObject子类的创建

[UE4 无法打开源文件 Actor.generated.h](https://blog.csdn.net/qq_33727884/article/details/91966617)

## 二、其蓝图类与基础宏

Blueprint 蓝图

其中在宏的括号后

```c++
UPROPERTY()//对变量的写在变量上
UFUNCTION()//对函数的写在函数上

BlueprintReadWrite// 对参数进行读写  
BlueprintCallable//对函数进行访问

VisibleAnywhere//哪里都能看
EditAnywhere///哪里都能改
    
EditInstanceOnly//只能在实例中进行编辑
VisibleInstanceOnly//只能在实例中进行看
EditDefaultsOnly//只能在蓝图模板中进行编辑
VisibleDefaultsOnly//只能在蓝图模板中进行看

BlueprintImplementableEvent //在C++可以声明函数（不能定义，蓝图重写），在C++里调用该函数，蓝图重写实现该函数
//C++不能实现功能不能重写，在蓝图实现函数的功能(Event),且在蓝图中调用

BlueprintNativeEvent 
//C++不能直接调用,但可以实现功能进行重写需要加_Implementation,c++中重写内容相当于蓝图中的Event，蓝图中没有重写功能只能调用后实现C++中功能
    
//BlueprintNativeEvent在特殊情况可以在c++中调用使用类名::Execute_函数名()可以调用

```

```c++
 meta=(ClampMin = -5.0f, ClampMax = 5.0f, UIMin = -5.0f, UIMax = 5.0f)
     
 //例子
 UPROPERTY(EditAnywhere, Category = "My Actor Properties | Vector", meta=(ClampMin = -5.0f, ClampMax = 5.0f, UIMin = -5.0f, UIMax = 5.0f))
 //Clamp限定所填的数字
 //UI限定的是滑动的数字
 
```



## 三、C++类的删除

先删除其子类和蓝图

后关闭vs和ue

打开工程文件夹找到Souce内部要删除的类

后回退到工程文件夹删除Binaries文件夹

右键ue启动程序点Generate Vs project

后启动

## 四、前缀

模板类的前缀是T。
object类的派生类前缀是u。o AActor类的派生类前缀是A。oswidget类的派生类前缀是s。
抽象接口类前缀是I。
枚举类型前缀是E。
布尔类型变量的变量名必须加上前缀b(例如: bPendingDestruction或者bHasFadedIn ) 。

## 五、RPC和OnRep

### Replicated修饰变量

OnRep是指 当一个被标记为`Replicated`的变量在`服务器端`被修改时，引擎会自动触发对应的OnRep_函数在客户端进行执行

故需要一个参数在客户端被改变执行OnRep：

一、对参数进行复制

二、写Server函数让参数传递到服务端被改变

三、写OnRep函数

### RPC类型

如下表所示，RPC有四种不同的类型：

| **元数据说明符** | **说明**                                                     |
| ---------------- | ------------------------------------------------------------ |
| `Client`         | 在此Actor的所属客户端连接上执行RPC。                         |
| `Server`         | 在服务器上执行RPC。必须从拥有此Actor的客户端调用。           |
| `Remote`         | 在连接的远程端执行RPC。连接的远程端可以是服务器也可以是客户端，但必须在客户端拥有的Actor上调用RPC。该RPC的行为既像 `Client` 又像 `Server` RPC，但绝不会在连接的本地端执行，仅在远程端执行。 |
| `NetMulticast`   | 在服务器上以及与Actor相关的所有当前连接的客户端上执行RPC。`NetMulticast` RPC设计用于从服务器调用，但也可从客户端调用。从客户端调用的 `NetMulticast` RPC仅在本地执行。 |



## 六、增加自定义碰撞通道

一、打开引擎项目设置中Object Channels增加通道

二、打开Vs 项目名.h 文件 加入 

```c++
#define ECC_(名字) ECollisionChannel::ECC_GameTraceChannel(数字)
```

## 七、委托

委托的Broadcast就是调用委托

```c++
DECLARE_MULTICAST_DELEGATE_OneParam(FAssertTagDelegate, const FGameplayTagContainer&)
DECLARE_MULTICAST_DELEGATE(FDataInfoDelegate)
```

这是2个委托一个是有一个变量的委托FAssertTagDelegate

一个是不含变量的FDataInfoDelegate



委托需要先绑定函数

委托绑定的函数需要在声明委托名称时写出变量

```c++

//代理委托就是函数指针（类成员函数指针），函数指针指向函数地址，然后调用该函数指针，实现所需效果。

//UE 委托进行Broadcast时会调用之前绑定的函数
```

### 三大类型

UE的委托分为三大类，具体的枚举分类可以参见：DelegateCombinations.h文件

- **1.单播委托:** TDelegate模板类，通常用`DECLARE_DELEGATE`及`DECLARE_DELEGATE_XXXXParams`宏进行声明，后面的宏表示带XXXX个参数*

- **2.多播委托:** TMulticastDelegate模板类，通常用`DECLARE_MULTICAST_DELEGATE`及`DECLARE_MULTICAST_DELEGATE_<Num>Params`宏进行声明，后面的宏表示带XXXX个参数

  可以绑定多个函数
  
- **3.动态委托:** 继承于TBaseDynamicDelegate，TBaseDynamicMulticastDelegate模板类，通常用`DECLARE_DYNAMIC_DELEGATE`及`DECLARE_DYNAMIC_MULTICAST_DELEGATE`宏进行声明，分别表示单播与多播，另外还有包含参数个数的宏如`DECLARE_DYNAMIC_DELEGATE_OneParam`，其本质集成了UObject的反射系统，让其可以注册蓝图实现的函数及支持序列化存储到本机，UFunction只需给函数名称及蓝图支持是其最大的特性，概念上并未逃出前两类，性能和功能弱于前两类。

- **2.1 Event:** 其继承多播，不单独讨论了。

### 核心区别总结

| **特性**     | **`DECLARE_` 系列**    | **`DECLARE_DYNAMIC_` 系列**                     |
| :----------- | :--------------------- | :---------------------------------------------- |
| **蓝图支持** | ❌ 不支持               | ✅ 支持（通过`BlueprintAssignable`等）           |
| **序列化**   | ❌ 不支持               | ✅ 可序列化（保存/加载）                         |
| **性能**     | ⚡️ 高效（直接函数指针） | ⚠️ 较高开销（名称查找）                          |
| **绑定限制** | 任意C++函数            | 仅限`UFUNCTION`标记函数                         |
| **参数类型** | 任意类型               | UE支持的类型（如`float/FString`，不能是裸指针） |
| **典型用途** | 纯C++系统              | 蓝图-C++交互                                    |

总结区别在于是否能和蓝图交互

## 八、IK绑定和重定向器

教程：[【UE5】从零开始创建IK绑定与重定向器，虚幻争霸角色动画重定向给MetaHuman](https://www.bilibili.com/video/BV1SP4y127KG/?spm_id_from=333.337.search-card.all.click)

1.先创建一个Rig文件夹创建IK绑定器命名规则IK_**01

2.右键root骨骼创建重定向链，右键pelvis设置为重定向根

3.分别将spine(脊椎全部），clavicle,(upperarm->hand),几个手指的骨骼(index,middle,pinky,ring,thumb),(neck->head),(thigh,calf,foot,ball),设置骨骼重定向链

4.相同方法再建一个你要传递动画原始地方的骨骼的IK绑定器IK_**02

5.同样操作进行骨骼创建重定向链

6.创建IK重定向器，谁是原动作的持有者用谁的骨骼进行创建命名规则RTG_**02

7.目标IKRig资产选择第一个IK绑定器IK_**01,和对应的网格体

8.检查有无映射错误(若有错误下拉菜单手动选择)，(IK绑定时目标链和源链名字一样就不会出错)

9.随机选个动画进行查看动作是否正常，进行适当修改(编辑姿势)

## 九、关于UE5的一些问题

### 1.关于粒子组件消失在摄像机中后粒子消失的问题

1.进入粒子的组件内搜索Calculate Bounds Mode 改为Fixd

2.修改Fixed Bounds下的Min和Max的范围即可

### 2.UE中文乱码解决方案

[VS2022设置编码方式为utf-8的三种方式_vs utf8-CSDN博客](https://blog.csdn.net/hfy1237/article/details/129858976)

