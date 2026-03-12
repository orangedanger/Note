# UE5.1 多人枪战游戏

 

## bug:

```c++
//死亡后第二武器不掉落(已解决)//必须当过主武器的才会掉落  139.

//切换武器时换弹动作应该取消(已解决)  Montaget_Stop函数更改CombatState状态

//按住瞄准切换武器后为狙击枪松开右键会触发动画(已解决)	137.SwapWeapon函数中

//死亡后复活子弹显示未更新(已解决)		Pollinit函数中Controller更新则进行子弹的更新即可 

//捡起第二武器时主武器子弹数目改变(已修复)		SetHUDAmmo中Ammo来自于Character->GetEquippedWeapon()->GetAmmo();

//客户端在初始化时并未更新手榴弹数目(已解决)

//以切换武器扔下的枪未销毁

//换子弹时切换武器而无法射击(已修复)       bLocalReload = false;

//LineTraceSingleByChannel的检测不到相应的碰撞箱BUG(已修复)				增加的通道为TraceChannel暂时不知道什么原因而修复的

//高延迟情况下客户端设置客户端会产生2个子弹

//切换武器bug快速多次来回切换会卡(已解决)

//服务器换弹前+1 客户端换弹只+1(已解决) 没问题延迟补偿原因
```



## 38.变量复制

```c++
 //目的：让服务端和客户端分别显示和消失 PickWidget
#include "Net/UnrealNetwork.h"

DOREPLIFETIME_CONDITION;//宏设置复制的
virtual void GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps)const override;//在函数内部调用复制宏



//在次写则客户端 会显示
OnRep_OverlappingWeapon();//只在客户端触发的函数

//在另一个函数写
if(IsLocallyControlled()）//判断是否是服务端控制进入来显示

```

## 39.装备武器

```c++
//目的：使角色装备武器


Replicated;//通过这个变量来使客户端和服务端进行传递信号
SetIsReplicated();

friend class 类名;//用于将自己所以的变量函数分享给其他类

virtual void PostInitializeComponents()override;//进行初始化
//这里Actor对象依然是原始信息未被用户设置过，是常用来编写初始化逻辑的地方

//装备武器
//WeaponState的状态需要改变.且骨骼的插槽位置获取将武器装备到骨骼插槽上面
GetSocketByName();//通过名字获取插槽
```

## 40.让客户端(Client)装备武器

```c++
//目的：让客户端装备武器


UFUNCTION(Server,Reliabled)
void ServerEquipButtonPressed();//RPC方便在服务端调用而在客户端实现的函数
void ServerEquipButtonPressed_Implementation();//这是RPC的实施函数 
//这里我们是为了能够拾取武器

//隐藏Widget
//用进行客户端的隐藏
#include "Net/UnrealNetwork.h"
virtual void GetLifetimeReplicatedProps(TArray<FLifetimeproperty>& OurLifetimeProps)const override;
DOREPLIFETIME(类名,需要复制的参数);
UPROPERTY(ReplicatedUsing=OnRep_参数名)
Type 需要复制的参数;
UFUNCTION()
void OnRep_参数名()

    

```

## 41.装备动画

```c++
//目的：当装备武器时更换动画


```

## 42.下蹲

```c++
//目的：增加下蹲
```

## 43.瞄准

```c++

```

## 44.设置装备时动画

## 45.完善装备时动画

```c++
//视角朝向相对于初始的旋转
FRotator AimRotator = FirstCharacter->GetBaseAimRotation();
//角色移动方向相对于初始的旋转
FRotator MovementRotator = UKismetMathLibrary::MakeRotFromX(FirstCharacter->GetVelocity());

bOrientRotationToMovement;//当为true时人物会朝着加速度方向旋转

bUseControllerRotationYaw;//当为true时人物会永远跟着镜头
//计算公式 			Current+(Target-Current)*Deltatime*InterpSpeed
Math::FInterpTo(Current,Target,Deltatime,InterpSpeed);
//计算公式 			Current+(Target-Current)*InterpSpeed
Math::Lerp(Current,Target,InterpSpeed);
//InterpSpeed 增加速度(0,1)			最后结果随着Current增大无限接近Target

//左右移动动画僵硬问题的解决方案
	if (abs(AimRotator.Yaw - MovementRotator.Yaw + 90.f) < 1.f || abs(AimRotator.Yaw - MovementRotator.Yaw - 90.f) <  1.f)
	{
		FRotator DeltaRot = UKismetMathLibrary::NormalizedDeltaRotator(MovementRotator, AimRotator);
		DeltaRotation.Yaw = FMath::FInterpTo(DeltaRotation.Yaw, DeltaRot.Yaw, DeltaTime, 15.f);
		YawOffset = DeltaRotation.Yaw;
	}
	else
	{
		YawOffset = UKismetMathLibrary::NormalizedDeltaRotator(MovementRotator, AimRotator).Yaw;
	}
```

## 46.静止和跳跃

## 47.蹲着走

单纯的动画蓝图

## 48.行走瞄准



## 49.瞄准偏移

## 50.应用瞄准偏移

## 51.FABRIK

```c++
//改变动画中的动作，手部相对武器的位置

//在动画蓝图中
FABRIK 

//获取武器插槽信息
LeftHandTransform = EquippedWeapon->GetWeaponMesh()->GetSocketTransform(FName("LeftHandSocket"), ERelativeTransformSpace::RTS_World);

//得到武器插槽信息相对于已知正确位置信息
FirstCharacter->GetMesh()->TransformToBoneSpace(FName("Hand_R"), LeftHandTransform.GetLocation(), FRotator::ZeroRotator, OutPosition, OutRotation);

//设置要输出信息
LeftHandTransform.SetLocation(OutPosition);
LeftHandTransform.SetRotation(FQuat(OutRotation));


```



## 52.原地转向

```c++
//新建一个类只包含转向类型

//在角色类中写清转向条件和改变类型来在动画蓝图中实现转向
```



## 53.完善原地转向

## 54.服务端的更新频率

```c++
//在Config文件里DefaultEngine.ini中添加
[/Script/OnlineSubsystem.ipNetDriver]
NetServerMaxTickRate=60
    
//C++文件中和蓝图中
//Fps类游戏一般设置为
NetUpdateFrequency = 66.f;
MinNetUpdateFrequency = 33.f;
```



## 55.不装备的蹲走

## 56.修复前左和后右动画

```c++
//动画蓝图旋转Root 和 spine_01
```



## 57.脚步和跳声音设置

## 58.子弹类的创建

```c++
//子弹类建立碰撞体积，并写清碰撞物品
//设置是否可以碰撞
CollisionBox->SetCollisionEnabled(ECollisionEnabled::QueryAndPhysics);
//设置碰撞类型   
//WorldStatic应用于不移动的任意Actor类型
//WorldDynamic用于受到动画或代码的影响而移动的Actor类型
CollisionBox->SetCollisionObjectType(ECollisionChannel::ECC_WorldDynamic);
//先对所有东西关闭碰撞之后再逐个开启
CollisionBox->SetCollisionResponseToAllChannels(ECollisionResponse::ECR_Ignore);
CollisionBox->SetCollisionResponseToChannel(ECollisionChannel::ECC_Visibility, ECollisionResponse::ECR_Block); 
CollisionBox->SetCollisionResponseToChannel(ECollisionChannel::ECC_WorldStatic, ECollisionResponse::ECR_Block);
```



## 58.开火动画和动画蒙太奇的运用

```c++

```

## 59.完善动画在多人游戏中的显示

## 60.十字显示

```c++
//先获取屏幕中心位置
	FVector2D ViewportSize;
	if (GEngine && GEngine->GameViewport)
	{
		GEngine->GameViewport->GetViewportSize(ViewportSize);
	}
	FVector2D CrosshairLocation(ViewportSize.X / 2, ViewportSize.Y / 2);
//通过屏幕中心位置获取对应世界地图位置
	FVector CrosshairWorldPosition;
	FVector CrosshairWorldDirection;
	bool bScreenToWorld = UGameplayStatics::DeprojectScreenToWorld(
		UGameplayStatics::GetPlayerController(this, 0),
		CrosshairLocation,
		CrosshairWorldPosition,
		CrosshairWorldDirection
	);

//显示一个DebugSphere来反应正确

	if (bScreenToWorld)
	{
		FVector Start = CrosshairWorldPosition;
		FVector End = Start + CrosshairWorldDirection * TRACE_LENGTH;
        //检测Start到End之间有没有可Visibility的东西结果再HitResult中

		GetWorld()->LineTraceSingleByChannel(
			HitResult,
			Start,
			End,
			ECollisionChannel::ECC_Visibility
		);

		if (!HitResult.bBlockingHit)
		{
			HitResult.ImpactPoint = End;
		}
		else
		{
			DrawDebugSphere(
				GetWorld(),
				HitResult.ImpactPoint,
				12.f,
				12,
				FColor::Red
			);
		}
	}
```

## 61.生成子弹

```c++
//获取插槽信息
const USkeletalMeshSocket* MeshScoket= GetWeaponMesh()->GetSocketByName(FName("MuzzleFlash"));
//生成Actor
	APawn* InstigatorPawn = Cast<APawn>(GetOwner());

	if (MeshScoket)
	{
		FTransform WeaponTransform = MeshScoket->GetSocketTransform(GetWeaponMesh());

		//设置子弹的主人
		FActorSpawnParameters SpawnParameters;
		SpawnParameters.Owner = GetOwner();
		SpawnParameters.Instigator = InstigatorPawn;
		//
		FVector ToTarget = HitTarget - WeaponTransform.GetLocation();
		FRotator TargetRotation = ToTarget.Rotation();

		UWorld* World = GetWorld();a
		if (World)
		{
			World->SpawnActor<AProjectlie>(
				ProjectileClass,
				WeaponTransform.GetLocation(),
				TargetRotation,
				SpawnParameters
			);
		}
	}
```

## 62.子弹运动

```c++
#include "GameFramework/ProjectileMovementComponent.h"

//随速度进行旋转
bRotationFollowsVelocity=true;
```

## 63.子弹特效

```c++
//附加特效
if (Tracer)
{
	TracerComponent = UGameplayStatics::SpawnEmitterAttached(
		Tracer,//被附加粒子效果
		CollisionBox,//载体
		FName(),
		GetActorLocation(),
		GetActorRotation(),
		EAttachLocation::KeepWorldPosition
	);
}
```

## 64.复制HitTarget

```c++
//通过Fvector_NetQuantize来在服务器和客户端之前传输数据HitTarget

//FVector_NetQuantize 相对于Fvector 可以节省宽带通过减去小数部分
```

## 65.子弹碰撞事件

```c++
//碰撞摧毁子弹和发出粒子效果和声音
UFUNCTION()//一定要带切记
virtual void OnHit(UPrimitiveComponent* HitComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, FVector NormalImpulse, const FHitResult& Hit);

//粒子效果
UGameplayStatics::SpawnEmitterAtLocation(GetWorld(), ImpactParticle, GetActorTransform());
//播放声音
UGameplayStatics::PlaySoundAtLocation(this, ImpactSound, GetActorLocation());

```

## 66.子弹壳

```c++
//启用重力
CasingMesh->SetEnableGravity(true);
//启用模拟物理
CasingMesh->SetSimulatePhysics(true);
//启用碰撞事件
CasingMesh->SetNotifyRigidBodyCollision(true);
```

## 67.子弹壳的掉落

```c++
//延迟
//TEXT()内为回调函数名称
//
//重点回调函数声明要加UFUNCTION()
//
FLatentActionInfo LantentInfo(0, FMath::Rand(), TEXT("DelayFinish"), this);
UKismetSystemLibrary::Delay(this, 5.0f, LantentInfo);

```

## 68.准心设置

```c++
//准星文件导入后先全选后 资产操作->通过属性矩阵批量操作->Compression->Compression Setting改为UserInterface2D

//将Weapon中的Crosshair传入HUD中,通过Combat传
void UCombatComponent::SetHUDPackage(float Deltatime)
{
	if (Character == nullptr || Character->Controller == nullptr)return;
	//Character->Controller
	Controller = Controller == nullptr ? Cast<AFirstPlayerController>(Character->Controller) : Controller;

	if (Controller)
	{
        //Controller->GetHUD()
		HUD = HUD == nullptr ? Cast<AFirstHUD>(Controller->GetHUD()) : HUD;
		if (HUD)
		{
			FHUDPackage HUDPackage;
			if (EquippedWeapon)
			{
				HUDPackage.CrosshairsCenter	= EquippedWeapon->CrosshairsCenter;
				HUDPackage.CrosshairsRight	= EquippedWeapon->CrosshairsRight;
				HUDPackage.CrosshairsLeft	= EquippedWeapon->CrosshairsLeft;
				HUDPackage.CrosshairsTop	= EquippedWeapon->CrosshairsTop;
				HUDPackage.CrosshairsBottom	= EquippedWeapon->CrosshairsBottom;
			}
			else
			{
				HUDPackage.CrosshairsCenter = nullptr;
				HUDPackage.CrosshairsRight = nullptr;
				HUDPackage.CrosshairsLeft = nullptr;
				HUDPackage.CrosshairsTop = nullptr;
				HUDPackage.CrosshairsBottom = nullptr;
			}
			HUD->SetHUDPackage(HUDPackage);
		}
	}
}
```

## 69.绘画准心

``` c++
	DrawTexture(
		Texture,//材质
		TextureDrawPonint.X,//材质应该画的点
		TextureDrawPonint.Y,//材质应该画的点
		TextureWidth,//材质的宽
		TextureHeight,//材质的高
		0.f,
		0.f,
		1.f,
		1.f,
		FLinearColor::White
	);
```



## 70.准心扩散

```c++
//传入扩散值到HUD中

//HUDPackage.CrosshairSpread [0,1]   Velocity [0,600]
FVector2D WalkSpeedRange(0.f, Character->GetCharacterMovement()->MaxWalkSpeed);
FVector2D VelocityMultiplierRange(0.f, 1.f);

FVector Velocity = Character->GetVelocity();
Velocity.Z = 0.f;

CrosshairVelocityFactor =FMath::GetMappedRangeValueClamped(WalkSpeedRange, VelocityMultiplierRange, Velocity.Size());

if (Character->GetCharacterMovement()->IsFalling())
{
	CrosshairAirFactor = FMath::FInterpTo(CrosshairAirFactor,2.25f,DelTatime,2.25f);
}
else
{
	CrosshairAirFactor = FMath::FInterpTo(CrosshairAirFactor, 0.f, DelTatime, 30.f);
}
if (Character->IsAiming())
{
	if (Character->GetVelocity().Size() == 0.f)
	{
		CrosshairAimFactor = FMath::FInterpTo(CrosshairAimFactor, -0.10f, DelTatime, 10.f);
	}
	else
	{
		CrosshairAimFactor = FMath::FInterpTo(CrosshairAimFactor, -0.5f, DelTatime, 10.f);
	}
}
else
{
	CrosshairAimFactor = FMath::FInterpTo(CrosshairAimFactor, 0.f, DelTatime, 30.f);
}
HUDPackage.CrosshairSpread = CrosshairAirFactor + CrosshairVelocityFactor + CrosshairAimFactor;

HUD->SetHUDPackage(HUDPackage);
```

## 71.修复武器枪口指示位置

```c++
if (FirstCharacter && FirstCharacter->IsLocallyControlled())
{
	bIsLocallyController = true;
	FTransform RightHandTransform = EquippedWeapon->GetWeaponMesh()->GetSocketTransform(FName("Hand_R"), ERelativeTransformSpace::RTS_World);
	RightHandRotation = UKismetMathLibrary::FindLookAtRotation(RightHandTransform.GetLocation(), RightHandTransform.GetLocation() + (RightHandTransform.GetLocation() - FirstCharacter->GetHitTarget()));
	RightHandRotation += FRotator(3.f, -7.5f, 0.f);
}

//初始点->目标点需要的旋转
FindLookAtRotation()
```

## 72.瞄准时放大

```c++
//调整摄像机的SetFieldOfView()


void UCombatComponent::SetCarmerFOV(float DeltaTime)
{
	if (EquippedWeapon == nullptr)return;

	if (bAiming)
	{
		CurrentFOV = FMath::FInterpTo(CurrentFOV, EquippedWeapon->GetZoomFOV(), DeltaTime, EquippedWeapon->GetZoomInterpTime());
	}
	else
	{
		CurrentFOV = FMath::FInterpTo(CurrentFOV, DefaultFOV, DeltaTime, ZoomInterpSpeed);
	}
	if (Character && Character->GetFollowCamera())
	{
		Character->GetFollowCamera()->SetFieldOfView(CurrentFOV);
	}
}
```



## 73.增加瞄准和射击时准心扩散

```c++
//增加2个Factor
CrosshairAimFactor
CrosshairShootFactor
```



## 74.长按连射

```c++
//UE5 定时器

//权柄的声明
FTimerHandle FireHandle;
//回调函数
void Shoot();
//启用定时器
GetWorld()->GetTimerManager().SetTimer(
    									FireHandle,  				//InOutHandle：定时器绑定的句柄，如果该句柄已指向其他定时器，则取消这个其他定时器；
     									this, 						//InObj：调用执行函数的对象；
      									&UCombatComponent::Shoot,	//InTimerMethod：定时器所执行的函数；
      									EquippedWeapon->GetRPM(), 	//InRate：函数执行的时间间隔，如果<=0，则清除现存定时器，即 InOutHandle 所绑定的定时器；
     									true, 						//InbLoop：该定时器是否循环，若不循环则只执行一次；
      									0.f							//InFirstDelay：从设置定时器到执行定时器的时间间隔，若<0，则使用 InRate 代替。
										);						

//关闭定时器

GetWorld()->GetTimerManager().ClearTimer(FireHandle);



```

## 75.准心颜色的变化

```c++
//Interface 本次只用检测类是否有借口相当于打上一个标签

OriginalObject->Implements<UReactToTriggerInterface>(); // 如果OriginalObject实现了UReactToTrigger，bIsImplemented将为true。
	⬇									⬇
HitResult.GetActor()->Implements<UInterfaceWithCrosshairInterface>()

//RInerpTo和FInterpTo相同用法
```

## 76.修复距离太近相机和准心的问题

```c++
//相机贴墙太近隐藏角色和武器  SetOwnerNoSee  SetVisibility
 void AFirstCharacter::HideCameraIfCharacterClose()
{
	if (!IsLocallyControlled())return;
	if ((FollowCamera->GetComponentLocation()-GetActorLocation()).Size()< CharacterCloseDistance)
	{
		GetMesh()->SetVisibility(false);
		if (Combat&& Combat->EquippedWeapon &&Combat->EquippedWeapon->GetWeaponMesh())
		{
			Combat->EquippedWeapon->GetWeaponMesh()->SetOwnerNoSee(true);
		}
	}
	else
	{
		GetMesh()->SetVisibility(true); 
		if (Combat && Combat->EquippedWeapon && Combat->EquippedWeapon->GetWeaponMesh())
		{
			Combat->EquippedWeapon->GetWeaponMesh()->SetOwnerNoSee(false);
		}
	}
}

//将Tacer的Start往前调整避免检测到人物后方的角色  DistanceToCharacter

float DistanceToCharacter = (Character->GetActorLocation() - Start).Size();
Start += CrosshairWorldDirection * (DistanceToCharacter + 100.f);
```

## 77.子弹命中角色

```c++
//修改CollisionType
//在项目设置搜Object找到Collision添加新的CollisonChannel
//ReactHitMontage
//多个动画的组合

//在子弹OnHit中去得到OtherActor是Character来播放动画
```

## 78.在其他代理时候进行平滑旋转

```c++
//动画蓝图调用不是Tick()故传输到服务端时有些时候没有数据而导致抖动



//每当运动发生改变则会调用可以有效的减少无效的Tick
OnRep_ReplicateMovement();

//关于遇到左转或右转
 

```

## 79.生命值



## 80.更新生命在HUD上



## 81.伤害

```c++
//子弹类中继承的OnHit检测到后
UGameplayStatics::ApplyDamage(
    OtherActor, //伤害对象
    Damage, 	//伤害大小
    OwnerController, 	//伤害主人的控制
    this, 				//子弹本身，伤害物品
    UDamageType::StaticClass()	//类型
);
//子弹类完成转到受伤害的地方
//Character 
//伤害监听器受到任何伤害时调用绑定于自己的函数
OnTakeAnyDamage.AddDynamic(this, &AFirstCharacter::ReciveDamage);
//在自己函数中计算生命，且进行更新和播放受击动画

//且需与客户端同步 OnRep_Health 更新HealthHUD和动画
```

## 82.GameMode创建制定游戏规则

```c++
//在新创建的GameMode中
virtual void PlayerEliminated(ElimmedCharacter,//受伤者
                              VictimController,//受伤者的控制器
                              AttackerController//攻击者的控制器
                             );
//在Character中调用PlayerEliminated
GetAuthGameMode<>();
//声明 
void Elim();

```

## 83.淘汰(Elim)动画

```c++
//用蒙太极(Montage)动画

//实现服务器同步使用
UFUNCTION(NetMulticast,Reliabled)
```

## 84.重生

```c++
//设定定时器

//写重生函数

virtual void Respawning(ACharacter* ElimmedCharacter,AController* ElimmedController);

//
if (ElimmedCharacter)
{
	ElimmedCharacter->Reset();
	ElimmedCharacter->Destroy();
}
if (ElimmedController)
{
	TArray<AActor*> PlayerActor;
    //获取所有重生点
	UGameplayStatics::GetAllActorsOfClass(this, APlayerStart::StaticClass(), PlayerActor);
    //随机重生点
	int Selection=FMath::RandRange(0, PlayerActor.Num() - 1);
	//重生
	RestartPlayerAtPlayerStart(ElimmedController, PlayerActor[Selection]);
}
```

## 85.制作溶解材质

## 86.溶解材质运用

```c++
//死亡溶解

UPROPERTY(VisibleAnywhere)
UTimelineComponent* DissolveTimeline;

FOnTimelineFloat DissolveTrack;
//曲线
UPROPERTY(EditAnywhere)
UCurveFloat* DissolveCurve;

UFUNCTION()
void UpdateDissolveMaterial(float DissolveValue);
void StartDissolve();

//运行时更改的实例
UPROPERTY(VisibleAnywhere,Category=Elim)
UMaterialInstanceDynamic* DissolveMaterialInstanceDynamic;
//在蓝图更改的实例
UPROPERTY(EditAnywhere, Category = Elim)
UMaterialInstance* DissolveMaterialInstance;

//初始化
DissolveMaterialInstanceDynamic = UMaterialInstanceDynamic::Create(DissolveMaterialInstance, this);
GetMesh()->SetMaterial(0, DissolveMaterialInstanceDynamic);
DissolveMaterialInstanceDynamic->SetScalarParameterValue(TEXT("Dissolve"), 0.55f);
DissolveMaterialInstanceDynamic->SetScalarParameterValue(TEXT("Glow"), 200.f);


void AFirstCharacter::UpdateDissolveMaterial(float DissolveValue)
{
	if (DissolveMaterialInstanceDynamic)
	{
		DissolveMaterialInstanceDynamic->SetScalarParameterValue(TEXT("Dissolve"), DissolveValue);
	}
}

void AFirstCharacter::StartDissolve()
{
	//绑定
	DissolveTrack.BindDynamic(this, &AFirstCharacter::UpdateDissolveMaterial);
	if (DissolveCurve&& DissolveTimeline)
	{
		//增加轨道
		DissolveTimeline->AddInterpFloat(DissolveCurve, DissolveTrack);
		//开始
		DissolveTimeline->Play();
	}
}
```

## 87.死亡后人物处理

```c++
//人物碰撞，移动，枪械的掉落
	//取消控制
	//WASD
	GetCharacterMovement()->DisableMovement();
	//鼠标
	GetCharacterMovement()->StopMovementImmediately();
	//input
	if (FirstPlayerController)
	{
		DisableInput(FirstPlayerController);
	}
	//取消碰撞
	GetCapsuleComponent()->SetCollisionEnabled(ECollisionEnabled::NoCollision);
	GetMesh()->SetCollisionEnabled(ECollisionEnabled::NoCollision);

//枪械掉落
		if (HasAuthority())
		{
			AreaSphere->SetCollisionEnabled(ECollisionEnabled::QueryOnly);
		}
		WeaponMesh->SetSimulatePhysics(true);
		WeaponMesh->SetEnableGravity(true);
		WeaponMesh->SetCollisionEnabled(ECollisionEnabled::QueryAndPhysics);
```

## 88.死亡时特效

```c++
//ElimRobot
//特效
UPROPERTY(EditAnywhere)
UParticleSystem* ElimBotEffect;
UPROPERTY(VisibleAnywhere)
UParticleSystemComponent* ElimBotEffectComponent;

//声音
UPROPERTY(EditAnywhere)
USoundBase* ElimBotSound;
```

## 89.死亡后生命不刷新

```c++
//在Controller文件夹中加入
virtual void OnPossess(APawn* InPawn)override;
//再次调用SetHealth
void AFirstPlayerController::OnPossess(APawn* InPawn)
{
	Super::OnPossess(InPawn);
	AFirstCharacter* FirstCharacter = Cast<AFirstCharacter>(InPawn);
	if (FirstCharacter)
	{
		SetHealth(FirstCharacter->GetMaxHealth(), FirstCharacter->GetHealth());
	}
}
```

## 90.显示得分(Score)(Player State)

```c++
//得分一般在PlayerState中故新建个PlayerState文件
//在PlayerState中Score是被复制的变量
//服务端更新
void AddToScore(float ScoreAmout);//Controller->SetScore(Score);
//客户端更新
virtual void OnRep_Score()override;

//FIrstPlayerController的
void SetScore(float Score);
//对HUD中 Text的信息修改 SetText();


//Score的初始化
//在FirstCharacter文件中，Tick运行AddScore(0.f);
void Pollinit();
```

## 91.显示死亡

```c++
//和显示分数基本一致
```

## 92.子弹数目

```c++
//在Weapon中设置变量
int32 Ammo;
//因为是整个服务器共享的需要复制这个变量
OnRep_Ammo();
//在服务端消耗Ammo
void SpendRound();


//在FIrstPlayerController
void SetAmmo(float Ammo);

//枪械丢出后重新拿到武器显示其子弹数量
//由于其Owner改变
void OnRep_Owner();


//替换武器时进行Dropped();


//失去武器Ammo归零
//MulticastElim_Implementation
Controller->SetAmmo(0);
```

## 93.没子弹时禁止开火

```c++
//更改了自动开火逻辑

//按下时触发Fire()在FIre中出发SetTimer
//调用FireTimerFinish->FIre来完成自动开火
//故在Fire中添加一个判断前提CanFire()
bool CanFire()
{
return !EquippedWeapon->IsEmpty()&&bCanFire;//IsEmpty()判断子弹数量，bCanFIre调控Fire和FireTimerFinish
}
```

## 94.人物储备子弹数目

```c++
//CarriedAmmoAmout


int32 CarriedAmmo;
//需要复制这个变量 CND_OwnerOnly

//每种武器有不同的子弹
//新建enum
EWeaponType

TMap<EWeaponType,int32>CarriedAmmoMap;


void initializeCarriedAmmo();

//Map的设置 
CarriedAmmoMap.Emplace(EWeaponType变量,int32变量)；
    
//检查是否含有相应变量返回bool类型
CarriedAmmoMap.Contain(EWeaponType变量)；

```

## 95.显示储备子弹数目

```c++
//和显示分数基本一致
//获取武器时SetCarriedAmmo()
```

## 96.换子弹

```c++
//Reload R键
//键位绑定
BindAction();

//动画设置
//想改变中间的位置前后不变，则选定骨骼在开始和结尾设置Key在想改变位置改变后加Key就行

Reload();
ServerReload_Impementation();
```

## 97.在客户端实现动画

```c++
//新建enum
ECombatState//控制其播放动画
ServerReload//只在服务器播放故需要传递到客户端->OnRep_CombatState();    
//在下面函数再一次播放动画即可
OnRep_CombatState();
//由于CombatState只改变一次多次R不起作用
UFUNCTION(BlueprintCallable)//在蓝图调用
void ReloadFinish();
//改回原类型
CombatState=ECombatState::ECS_Unoccupied;
//在动画Montage中添加通知->新建通知->ReloadFinish
//在动画蓝图中调用通知的ReloadFinish(通知名)
//在蓝图中Character->Combat->ReloadFinish(),Combat需要改成BlueprintReadOnly,且私有
meta=(AllowPrivateAccess="true");
//手应为FABRIK导致动画手部位置不对
//在FirstAnimInstance中加入
bool bFABRIK;
//判断条件
bFABRIK!=ECombatState::ECS_Reloading;
//最后完成动画蓝图编辑即可只需要取消换弹的手Transform即可
```

## 98.CanFire限制

```c++
//CombatState在Reload时不可Fire()
```

## 99.更新子弹和备弹数量

```c++
//计算需要多少子弹
int32 UCombatComponent::AmoutToReload()
{
	if (EquippedWeapon == nullptr) return 0;
	int32 RoomInMag = EquippedWeapon->GetMagCapactity() - EquippedWeapon->GetAmmo();
	if (CarriedAmmoMap.Contains(EquippedWeapon->GetWeaponType()))
	{
		int32 CarriedAmout = CarriedAmmoMap[EquippedWeapon->GetWeaponType()];
		int32 Least = FMath::Min(RoomInMag, CarriedAmout);
	/*	return FMath::Clamp(RoomInMag, 0, Least);*/
		return FMath::Clamp(RoomInMag, 0, CarriedAmout);
	}
	return 0;
}

void UCombatComponent::UpdateAmmoValue()
{
	if (Character == nullptr || EquippedWeapon == nullptr)return;
	int32 ReloadAmmo = AmoutToReload();
	if (CarriedAmmoMap.Contains(EquippedWeapon->GetWeaponType()))
	{
		CarriedAmmoMap[EquippedWeapon->GetWeaponType()] -= ReloadAmmo;
		CarriedAmmo = CarriedAmmoMap[EquippedWeapon->GetWeaponType()];
	}
	Controller = Controller == nullptr ? Cast<AFirstPlayerController>(Character->Controller) : Controller;
	if (Controller)
	{
		Controller->SetCarriedAmmo(CarriedAmmo);
	}
	EquippedWeapon->AddAmmo(-ReloadAmmo);
}

```

## 100.声音和手部位置

```c++
//装备武器和换弹声音
//换弹在Montage中
//武器装备
GameplayStatics::PlaySoundAtLocation();
//手部位置
//AimOffset有关
bRightTransform=FirstCharacter->GetCombatState() != ECombatState::ECS_Reloading;
bAimOffset = FirstCharacter->GetCombatState() != ECombatState::ECS_Reloading;
```

## 101.自动换弹

```c++
//Fire完成和EquipWeapon
//判断子弹是否为空Reload()
```

## 102.游戏倒计时

```c++
//设置Text

//游戏启动时间
GetWorld()->GetTimeSeconds();

void AFirstPlayerController::SetHUDTime()
{
	uint32 SecondsLeft = FMath::CeilToInt(GameTime - GetServerTime());
	if (CountDownTime!= SecondsLeft)
	{
		SetGameTimeText(GameTime - GetServerTime());
	}

	CountDownTime = SecondsLeft;
}
```

## 103.游戏倒计时同步

```c++
//由于获取的是每个电脑的进入游戏开始的时间故要同步

//获取服务器的时间
//传输到服务器和传输回服务器的时间
//传回客户端时客户端的时间
//运算后得到



	//函数在角色和控制器链接时调用:   客户端调用开始传输
	virtual void ReceivedPlayer()override;
	//获取服务器当前时间
	UFUNCTION(Server,Reliable)
	void ServerRequestServerTime(float TimeOfClientRequest);
	//客户端收到服务端传回的时间，和客户端当前的时间
	UFUNCTION(Client, Reliable)
	void ClientReportServerTime(float TimeOfClientRequest, float TimeServerReceivedClientRequest);

	float GetServerTime();

	UPROPERTY(EditAnywhere)
	int32 GameTime = 120.f;
	uint32 CountDownTime=0;
	float ClientSeverDelta = 0.f;
	UPROPERTY(EditAnywhere,Category=Time)
	float TimeSyncFrequency = 5.f;
	float TimeSyncRunning = 0.f;





void AFirstPlayerController::ReceivedPlayer()
{
	Super::ReceivedPlayer();
	if (IsLocalController())
	{
	ServerRequestServerTime(GetWorld()->GetTimeSeconds());
	}
}

void AFirstPlayerController::ServerRequestServerTime_Implementation(float TimeOfClientRequest)
{
	float ServerTime=GetWorld()->GetTimeSeconds();
	ClientReportServerTime(TimeOfClientRequest, ServerTime);
}

void AFirstPlayerController::ClientReportServerTime_Implementation(float TimeOfClientRequest, float TimeServerReceivedClientRequest)
{
	float RoundTripTime = GetWorld()->GetTimeSeconds() - TimeOfClientRequest;
	float CurrentServerTime = TimeServerReceivedClientRequest + (RoundTripTime * 0.5);
	ClientSeverDelta = CurrentServerTime - GetWorld()->GetTimeSeconds();
}

float AFirstPlayerController::GetServerTime()
{
	if (HasAuthority())
	{
		return GetWorld()->GetTimeSeconds();
	}
	else
	{
		return GetWorld()->GetTimeSeconds() + ClientSeverDelta;
	}
}


```

## 104.GameState准备阶段

```c++
//在GameMode中
//构造函数中
bDElayedStart=true;


//Tick函数中倒计时结束执行
StartMatch();
//将MatchState从WaitingToStart->InProgress



//由于5.1使用EnhancedInput而StartMatch会导致EnhancedInput失效
//重写StartMatch
//在函数中重新绑定输入
	// 获取玩家控制器并重新绑定输入
APlayerController* PlayerController = UGameplayStatics::GetPlayerController(GetWorld(), 0);
if (PlayerController) 
{
	AFirstCharacter* YourCharacter = Cast<AFirstCharacter>(PlayerController->GetPawn());
	if (YourCharacter && YourCharacter->GetLocalRole() == ROLE_Authority) 
    {
		YourCharacter->BindEnhancedInput();
	}
}
//BindEnhancedInput()
void AFirstCharacter::BindEnhancedInput()
{
	//UE5 增加映射
	if (APlayerController* PlayerController = Cast<APlayerController>(Controller))
	{
		UEnhancedInputLocalPlayerSubsystem* Subsystem = ULocalPlayer::GetSubsystem<UEnhancedInputLocalPlayerSubsystem>(PlayerController->GetLocalPlayer());
		Subsystem->AddMappingContext(IMC_FirstAction, 0);
	}
}
```

## 105.准备阶段的HUD显示问题

```c++
//替换HUD

//先将FirstHUD中的BeginPlay()  CharacterOverlap替换成准备阶段的Overlap
//替换MatchState时会执行 OnMatchState()
void AFirstGameMode::OnMatchStateSet()
{
	Super::OnMatchStateSet();
	//所有的控制器的MatchState进行更改
	for (FConstPlayerControllerIterator It = GetWorld()->GetPlayerControllerIterator(); It; ++It)
	{
		AFirstPlayerController* PlayerController = Cast<AFirstPlayerController>(*It);
		if (PlayerController)
		{
			PlayerController->OnMatchState(MatchState);
		}
	}
}
//PlayerController->OnMatchState(MatchState);
void AFirstPlayerController::OnMatchState(FName State)
{
	MatchState = State;
	if (MatchState == MatchState::InProgress)
	{
		HandleMatchHasStart();
	}
	else if (MatchState == MatchState::CoolDown)
	{
		HandleCoolDown();
	}
}


//替换MatchState需要客户端同时替换HUD   故有OnRep_MatchState()


//HandleMatchHasStart();
void AFirstPlayerController::HandleMatchHasStart()
{
	FirstHUD = FirstHUD == nullptr ? Cast<AFirstHUD>(GetHUD()) : FirstHUD;
	if (FirstHUD && FirstHUD->CharacterOverlap==nullptr)
	{
        //显示HUD的替换
		FirstHUD->AddCharacterOverlap();
		if (FirstHUD->GameStartOverlap)
		{
             //准备阶段进行隐藏结束时可以重复利用
			FirstHUD->GameStartOverlap->SetVisibility(ESlateVisibility::Hidden);
		}
	}
}
```

## 106.准备时间

## 107.准备时间的更新

```c++
//参考游戏时间倒计时
CountdownTime = WarmupTime - GetWorld()->GetTimeSeconds() + LevelStartingTime;
if (CountdownTime <= 0.f)
{
	StartMatch();
}

//设置更新时间 Tick()
void AFirstPlayerController::SetHUDTime()
{
	float LeftTime = 0.f;
	if (MatchState == MatchState::WaitingToStart) LeftTime = WarmupTime - GetServerTime() + LevelStartingTime;
	else if (MatchState == MatchState::InProgress) LeftTime = WarmupTime + GameTime - GetServerTime() + LevelStartingTime;
	else if(MatchState==MatchState::CoolDown )LeftTime = CoolDownTime + WarmupTime + GameTime - GetServerTime() + LevelStartingTime;

	uint32 SecondsLeft = FMath::CeilToInt(LeftTime);

	if (HasAuthority())
	{
		FirstGameMode = FirstGameMode == nullptr ? Cast<AFirstGameMode>(UGameplayStatics::GetGameMode(this)) : FirstGameMode;
		if (FirstGameMode)
		{
			SecondsLeft = FirstGameMode->GetCountdownTime();
		}
	}
	
	if (CountDownTime!= SecondsLeft)
	{
		if (MatchState == MatchState::WaitingToStart||MatchState==MatchState::CoolDown)
		{
			SetGameStartTimeText(LeftTime);
		}
		if (MatchState == MatchState::InProgress)
		{
			SetGameTimeText(LeftTime);
		}
		
	}

	CountDownTime = SecondsLeft;
}
```



## 108.结束时间

```c++
//类似
CountdownTime =CoolDownTime+ WarmupTime + GameTime - GetWorld()->GetTimeSeconds() + LevelStartingTime;
if (CountdownTime <= 0.f)
{
	RestartGame();
}
```

## 109.结束时间内容

```c++
//重新显示
//重新设置
//设置获胜者

//判断Score当有分数增加时与曾经最高分进行比较如果高且不是同一位玩家时进行替换，如果有一位相同则同时写进
//无获胜者
//1位获胜者
//多位获胜者
	if (FirstGameState)
	{
		TArray<AFirstPlayerState*> TopPlayers = FirstGameState->TopScorePlayer;
		FString TipsString;
		if (TopPlayers.Num() == 0)
		{
			TipsString = FString("There is no Winner.");
		}
		else if (TopPlayers.Num() == 1&&TopPlayers[0]== FirstPlayerState)
		{
			TipsString = FString("You Are Winner.");
		}
		else if (TopPlayers.Num() == 1)
		{
			TipsString = FString::Printf(TEXT("Winner:\n%s"),*TopPlayers[0]->GetPlayerName());
		}
		else if (TopPlayers.Num() > 1)
		{
			TipsString = FString("Player Tied for the win:\n");
			for (auto TiedPlayer : TopPlayers)
			{
				TipsString.Append(FString::Printf(TEXT("%s\n"), *TiedPlayer->GetPlayerName()));
			}
		}
		FirstHUD->GameStartOverlap->Tips->SetText(FText::FromString(TipsString));
	}
		
```

## 110.Rocket子弹

```c++
//碰撞检测即可，范围的应用伤害衰减

void AProjectlieRocket::OnHit(UPrimitiveComponent* HitComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, FVector NormalImpulse, const FHitResult& Hit)
{
	if (OtherActor == GetOwner())
	{
		//UE_LOG(LogTemp, Warning, TEXT("Hit Self"));
		return;
	}
	APawn* FiringPawn = GetInstigator();
	if (FiringPawn&&HasAuthority())
	{
		AController* FiringController=FiringPawn->GetController();
		if (FiringController)
		{
            //范围应用伤害衰减
			UGameplayStatics::ApplyRadialDamageWithFalloff(
				this,
				Damage,
				10.f,
				GetActorLocation(),
				200.f,
				500.f,
				1.f,
				UDamageType::StaticClass(),
				TArray<AActor*>(),
				this,
				FiringController
			);
		}
	}

	GetWorldTimerManager().SetTimer(
		DestoryTimer,
		this,
		&AProjectlieRocket::DestoryTimeFinish,
		DestoryTime
	);

	if (ImpactParticle)
	{
		UGameplayStatics::SpawnEmitterAtLocation(GetWorld(), ImpactParticle, GetActorTransform());
	}
	if (ImpactSound)
	{
		UGameplayStatics::PlaySoundAtLocation(this, ImpactSound, GetActorLocation());
	}
	if (TrailSystemComponent&& TrailSystemComponent->GetSystemInstance())//关
	{
		TrailSystemComponent->GetSystemInstance()->Deactivate();
	}
	if (ProjectileStatic)
	{
		ProjectileStatic->SetVisibility(false);
	}
	if (CollisionBox)
	{
		CollisionBox->SetCollisionEnabled(ECollisionEnabled::NoCollision);
	}
	if (ProjectileLoopComponent&&ProjectileLoopComponent->IsPlaying())
	{
		ProjectileLoopComponent->Stop();
	}
	//Super::OnHit(HitComponent, OtherActor, OtherComp, NormalImpulse, Hit);
}

```

## 111.Rocket轨迹显示

```c++
//特效过早消失问题

//Destroy进行延迟就行
//网格体和碰撞箱进行隐藏
```

## 112.Rocket与自身碰撞的修复

```c++
//对自身碰撞的取消，检测到碰撞为自己则跳过
//跳过后则想办法回复其本身运动
//增加Movement

#include "RocketMovementComponent.h"
virtual EHandleBlockingHitResult HandleBlockingHit(const FHitResult& Hit, float TimeTick, const FVector& MoveDelta, float& SubTickTimeRemaining) override;
virtual void HandleImpact(const FHitResult& Hit, float TimeSlice = 0.f, const FVector& MoveDelta = FVector::ZeroVector) override;
/**
*
*/
#include "RocketMovementComponent.cpp"
URocketMovementComponent::EHandleBlockingHitResult URocketMovementComponent::HandleBlockingHit(const FHitResult& Hit, float TimeTick, const FVector& MoveDelta, float& SubTickTimeRemaining)
{
	Super::HandleBlockingHit(Hit,TimeTick,MoveDelta,SubTickTimeRemaining);
	return EHandleBlockingHitResult::AdvanceNextSubstep;
}

void URocketMovementComponent::HandleImpact(const FHitResult& Hit, float TimeSlice, const FVector& MoveDelta)
{
	//当碰撞箱碰到发射者不应该停止移动
}

```

## 113.HitScanWeapon

```c++
//不用子弹的射击武器(激光武器)
//利用碰撞直线检测，检测是否有玩家，然后进行伤害
	//获取碰撞点   从枪口到指向位置中间
	World->LineTraceSingleByChannel(
		FireResult,
		Start,
		End,
		ECollisionChannel::ECC_Visibility
	);
	FVector BeamEnd = End;
	//应用伤害
	if (FireResult.bBlockingHit)
	{			
		//获取碰撞点   从枪口到指向位置中间
		BeamEnd = FireResult.ImpactPoint;
		AFirstCharacter* DamagedCharacter = Cast<AFirstCharacter>(FireResult.GetActor());
		if (DamagedCharacter&& HasAuthority() &&InstigatorController)
		{
			UGameplayStatics::ApplyDamage(
				DamagedCharacter,
				Damaged,
				InstigatorController,
				this,
				UDamageType::StaticClass()
			);
		}
	}

```



## 114.BeamSmoke射击轨道

```c++
//特效
if (SmokeBeamParticle)
{
    //设置特效开始点
	UParticleSystemComponent* Beam = UGameplayStatics::SpawnEmitterAtLocation(
		World,
		SmokeBeamParticle,
		SocketTransform
	);
	if (Beam)
	{
        //设置特效终点
		Beam->SetVectorParameter(FName("Target"), BeamEnd);
	}
}
```



## 115.Submachine Gun(冲锋枪)

```c++
//HitScan的自动射击模式
```



## 116.冲锋枪的模拟物理

```c++
//改变其物理模型
//将需要重力的地方设置模拟物理即可
```

## 117.StotGun(散弹枪)

```c++
//多个子弹
//随机坐标
//在指定范围内
//开始坐标和生成的随机坐标连线延长即可
FVector AHitScanWeapon::TraceEndSctter(const FVector& TraceStart, const FVector& HitTarget)
{
	FVector ToTargetNormalize = (HitTarget - TraceStart).GetSafeNormal();
	FVector SphereCenter = TraceStart + ToTargetNormalize * DistanceToSphere;
	//随机生成坐标
	FVector RandVec = UKismetMathLibrary::RandomUnitVector() * FMath::FRandRange(0.f, SphereRadius);
	FVector EndLoc = SphereCenter + RandVec;
	FVector ToEndLoc = EndLoc - TraceStart;
	//指定范围
	DrawDebugSphere(GetWorld(),SphereCenter,SphereRadius,12,FColor::Red,true);
    //随机生成的坐标点
	DrawDebugSphere(GetWorld(), EndLoc, 4.f, 12, FColor::Orange, true);
    //连线
	DrawDebugLine(
		GetWorld(),
		TraceStart,
		TraceStart + ToEndLoc * TRACE_LENGTH / ToEndLoc.Size(),
		FColor::Cyan,
		true
	);

	return FVector(TraceStart + ToEndLoc * TRACE_LENGTH / ToEndLoc.Size());
}

```

## 118.武器的散射伤害判定

```c++
//获取碰撞结果
void AHitScanWeapon::TraceWithSctter(const FVector& TraceStart, const FVector& HitTarget, FHitResult& OutResult)
{
	FVector End = bUseScatter?TraceEndSctter(TraceStart,HitTarget): TraceStart + (HitTarget - TraceStart) * 1.25f;

	UWorld* World = GetWorld();
	if (World)
	{
		//获取碰撞点   从枪口到指向位置中间
		World->LineTraceSingleByChannel(
			OutResult,
			TraceStart,
			End,
			ECollisionChannel::ECC_Visibility
		);
		FVector BeamEnd = End;
		
		if (OutResult.bBlockingHit)
		{
			//获取碰撞点   从枪口到指向位置中间
			BeamEnd = OutResult.ImpactPoint;
		}
	}
}



//TMap 存储打到多个角色的伤害计算
TMap<AFirstCharacter*, int32> Hits;

for (uint32 i = 0; i < NumOfScatter; ++i)
{
	FVector End = TraceEndSctter(Start, HitTarget);

	FHitResult FireResult;
	TraceWithSctter(Start, HitTarget, FireResult);


	if (FireResult.bBlockingHit)
	{
		//获取碰撞点   从枪口到指向位置中间
		AFirstCharacter* DamagedCharacter = Cast<AFirstCharacter>(FireResult.GetActor());
		if (DamagedCharacter && HasAuthority() && InstigatorController)
		{
            //计算打到的子弹数目
			if (Hits.Contains(DamagedCharacter))
			{
				Hits[DamagedCharacter]++;
			}
			else
			{
				Hits.Emplace(DamagedCharacter, 1);
			}
		}
	}
}

//进行伤害结算   角色的遍历
for (auto Pair : Hits)
{
	if (Pair.Key && HasAuthority() && InstigatorController)
	{
		UGameplayStatics::ApplyDamage(
			Pair.Key,
			Damaged * Pair.Value,
			InstigatorController,
			this,
			UDamageType::StaticClass()
		);
	}
}





```

## 119.狙击枪及其瞄准镜

```c++

```

## 120.霰弹枪的换弹

```c++
UFUNCTION(BlueprintCallabl
void ShotgunShellRelod();
void UpdateShotgunAmmoValue();
          
 //蓝图调用 Shell 每次播放声音时换次弹
void UCombatComponent::ShotgunShellRelod()
{
	if (Character && Character->HasAuthority())
	{
		UpdateShotgunAmmoValue();
	}
}

void UCombatComponent::UpdateShotgunAmmoValue()
{
    //和UpdateAmmoValue差不多
	if (Character == nullptr || EquippedWeapon == nullptr)return;
	if (CarriedAmmoMap.Contains(EquippedWeapon->GetWeaponType()))
	{
        //备弹数目-1并更新
		CarriedAmmoMap[EquippedWeapon->GetWeaponType()] -= 1;
		CarriedAmmo = CarriedAmmoMap[EquippedWeapon->GetWeaponType()];
	}
	Controller = Controller == nullptr ? Cast<AFirstPlayerController>(Character->Controller) : Controller;
	if (Controller)
	{
		Controller->SetCarriedAmmo(CarriedAmmo);
	}
    //子弹数目-(-1)
	EquippedWeapon->AddAmmo(-1);
    //满足条件跳转动画至结尾
	if (EquippedWeapon->IsFull()||CarriedAmmo==0)
	{
		JumpToShellOfEnd();
	}
}

void UCombatComponent::JumpToShellOfEnd()
{
	//Jump To ShellOfEnd
		UAnimInstance* AnimInstance = Character->GetMesh()->GetAnimInstance();
		if (AnimInstance&&Character->GetReloadMontage())
		{
			AnimInstance->Montage_JumpToSection(FName("ShellOfEnd"));
		}
}
          
//为了使得换弹时开火在CanFire()中增加
          //使得CanFire()返回值为true
if (!EquippedWeapon->IsEmpty() && 
    bCanFire && 
    CombatState == ECombatState::ECS_Reloading && 
    EquippedWeapon->GetWeaponType() == EWeaponType::EWT_Shot)
          return true;          
          
//和MulticastFire_Implementation()增加
if (Character && CombatState == ECombatState::ECS_Reloading && EquippedWeapon->GetWeaponType() == EWeaponType::EWT_Shot)
{
    //改变CombatState
	CombatState = ECombatState::ECS_Unoccupied;
	Character->PlayFireMontage(bAiming);
	EquippedWeapon->Fire(TracerHitTarget);
	return;
}
          
```

## 121.武器的边缘发光(custom depth)

```c++
//放置后期盒(PostProcessVolume)
//开启 Infinite Extent(Unbound)  无限范围按钮
//Post Process Materials增加 发光材质
//后期处理材质(WeightedBlendables) 
//项目设置中(custom depth-Stencil Pass)Enabled with Stencil
//添加材质


//c++
//设置边框颜色
	WeaponMesh->SetCustomDepthStencilValue(CUSTOM_DEPTH_BLUE);
	WeaponMesh->MarkRenderStateDirty();
//开启渲染
	WeaponMesh->SetRenderCustomDepth(true);
```

## 121.手榴弹的动画蒙太奇Montage

```c++
//蓝图Blueprint部分
//增加输入动作
//增加Montage动画

//c++
//动作绑定， Montage动画的播放

//播放动画
void AFirstCharacter::PlayThrowingGrenadeMontage()
{
	UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();
	if (AnimInstance&&ThrowingGrenadeMontage)
	{
		AnimInstance->Montage_Play(ThrowingGrenadeMontage);
	}
}
//动作绑定
void AFirstCharacter::ThrowingGrenadeButtonPressed()
{
	if (Combat)
	{
		Combat->ThrowGrenade();
	}
}
//Combat

//服务器的ThrowGrenade
void UCombatComponent::ThrowGrenade()
{
	if (CombatState != ECombatState::ECS_Unoccupied) return;
	CombatState = ECombatState::ECS_ThrowingGrenade;
	if (Character)
	{
		Character->PlayThrowingGrenadeMontage();
	}
	if (!Character->HasAuthority())
	{
		ServerThrowGrenade();
	}
}

//客户端的ThrowGrenade
void UCombatComponent::ServerThrowGrenade_Implementation()
{
	CombatState = ECombatState::ECS_ThrowingGrenade;
	if (Character)
	{
		Character->PlayThrowingGrenadeMontage();
	}
}
//在Montage中结尾进行改变CombatState
void UCombatComponent::ThrowGrenadeFinish()
{
	CombatState = ECombatState::ECS_Unoccupied;
}
```





## 121.扔手榴弹时左右手动作

```c++

void UCombatComponent::AttachActorToRightHandSocket(AActor* ActorToAttach)
{
	if (Character==nullptr || Character->GetMesh() == nullptr || ActorToAttach == nullptr)return;
    //获取右手插槽
	const USkeletalMeshSocket* HandSocket = Character->GetMesh()->GetSocketByName(FName("RightHandSocket"));
	if (HandSocket)
	{
        //插槽和武器绑定
		HandSocket->AttachActor(ActorToAttach, Character->GetMesh());
	}
}

void UCombatComponent::AttachActorToLeftHandSocket(AActor* ActorToAttach)
{
	if (Character == nullptr || Character->GetMesh() == nullptr || ActorToAttach == nullptr||EquippedWeapon==nullptr)return;
	//手枪拿法不一样
	bool bUsePistolScoket = EquippedWeapon->GetWeaponType() == EWeaponType::EWT_Pistol ||
							EquippedWeapon->GetWeaponType() == EWeaponType::EWT_Submachine;
	FName ScoketName = bUsePistolScoket ? FName("PistolSocket") : FName("LeftHandSocket");
    //获取插槽
	const USkeletalMeshSocket* HandSocket = Character->GetMesh()->GetSocketByName(ScoketName);
	if (HandSocket)
	{
        //插槽和武器绑定
		HandSocket->AttachActor(ActorToAttach, Character->GetMesh());
	}
}
```



## 122.手榴弹资源

```c++
//
```



## 123.手雷的显示

```c++
//手上的手雷显示
//动画中新建通知
//在动画蓝图中调用函数




void UCombatComponent::ShowAttachGrenade(bool bShowGrenade)
{
	if (Character == nullptr || Character->GetGrenadeLauncher() == nullptr)return;

	Character->GetGrenadeLauncher()->SetVisibility(bShowGrenade);
}

```



## 124.扔出手雷

```c++

//和手雷显示连接

//在动画运行一般调用这个函数

//展示手上的手雷
void UCombatComponent::ShowGrenadeFinish()
{
	ShowAttachGrenade(false);
	if(Character&&Character->IsLocallyControlled())
	ServerShowGrenadeFinish(HitTarget);
	
}
//展示手上的手雷结束并生成在世界中的手雷
void UCombatComponent::ServerShowGrenadeFinish_Implementation(const FVector_NetQuantize& Target)
{
	if (Character && GrenadeClass && Character->GetGrenadeLauncher())
	{
        //开始位置和终点
		FVector StartLocation = Character->GetGrenadeLauncher()->GetComponentLocation();
		FVector ToTarget = Target - StartLocation;
		//设置参数  手雷的主人(Owner)和始作俑者(Instigator)
		FActorSpawnParameters SpawnParameters;
		SpawnParameters.Owner = Character;
		SpawnParameters.Instigator = Character;

		UWorld* World = GetWorld();
		if (World)
		{
            //生成手雷
			World->SpawnActor<AProjectlie>(
				GrenadeClass,
				StartLocation,
				ToTarget.Rotation(),
				SpawnParameters
			);
		}
	}
}
```



## 附一、找到的bug:死亡时被爆炸炸死会反复死亡(解决)，

```c++
//在径向伤害处将死亡角色进行排除
TArray<AActor*>ElimActor;
ElimActor = TArray<AActor*>();
for (FConstPawnIterator It = GetWorld()->GetPawnIterator(); It; ++It)
{
	AFirstCharacter* PlayerCharacter = Cast<AFirstCharacter>(*It);
			
	if (PlayerCharacter->GetIsElim())
	{
		ElimActor.AddUnique(PlayerCharacter);
	}
}


//简单方法在ReciveDamage处
//提前判断是否死亡即可
if(bElime)return;
```



## 附二、在ECombatState状态为非ECS_Unoccupied时受到伤害状态无法改变(解决)

```c++
//原因是在其他状态受伤会播放受伤蒙太奇
//添加内容即可
if (Combat->CombatState != ECombatState::ECS_Unoccupied)return;
```



## 125.手榴弹的HUDs

```c++
//显示手雷数目
```



## 126.子弹拾取类

```c++
//记得绑定  自己的碰撞开始重叠函数

//增加了旋转
PickupStaticMesh->AddWorldRotation(FRotator(0.f, BaseRotationSpeed*DeltaTime,0.f).Quaternion());


if (HasAuthority())
{
	OverlapComponent->OnComponentBeginOverlap.AddDynamic(this, &APickup::OnSphereOverlap);
}
```

## 127.完善子弹拾取

```c++

void AAmmoPickup::OnSphereOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	Super::OnSphereOverlap(OverlappedComponent, OtherActor, OtherComp, OtherBodyIndex, bFromSweep, SweepResult);
	AFirstCharacter* Character = Cast<AFirstCharacter>(OtherActor);
	if (Character)
	{
		UCombatComponent* Combat = Cast<UCombatComponent>(Character->GetCombat());
		if (Combat)
		{
            //主要碰撞发生时调用函数
			Combat->PickupAmmo(WeaponType, AmmoAmout);
		}
	}
	Destroy();
}


void UCombatComponent::PickupAmmo(EWeaponType WeaponType, int32 AmmoAmout)
{
    //判断是否包含
	if (CarriedAmmoMap.Contains(WeaponType))
	{
        //加相应数目
		CarriedAmmoMap[WeaponType] += AmmoAmout;
		//即使更新
		UpdateCarriedAmmo();
	}
    //如果当时空的就即使换弹
	if (EquippedWeapon && EquippedWeapon->IsEmpty() && EquippedWeapon->GetWeaponType() == WeaponType)
	{
		Reload();
	}
}

```



## 128.BuffComponent类的创建

```c++
//和CombatComponent类类似

//在UBuffComponent.h中构造函数下
friend class AFirstCharacter;

//在FirstCharacter中也要声明创建BuffComponent和相应的Buff接口函数方便访问

```



## 129.生命拾取类

```c++
//类似于子弹


void AHealthPickup::OnSphereOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
	
	Super::OnSphereOverlap(OverlappedComponent, OtherActor, OtherComp, OtherBodyIndex, bFromSweep, SweepResult);
	AFirstCharacter* Character = Cast<AFirstCharacter>(OtherActor);
	if (Character)
	{	
		UBuffComponent* Buff = Cast<UBuffComponent>(Character->GetBuff());
		if (Buff)
		{
			Buff->Heal(HealAmount, HealingTime);
		}
	}
	Destroyed();
}


void UBuffComponent::Heal(float HealAmout, float HealingTime)
{
    //设置是慢慢增加即需要开关
	bHealing = true;
	//速度和数目
	HealingRate = (HealAmout + AmountToHeal) / HealingTime;
	AmountToHeal += HealAmout;

}


void UBuffComponent::UpdateHealing(float DeltaTime)
{
    //开关条件
	if (!bHealing || Character == nullptr || Character->GetIsElim())return;
	//每帧增加数目
	const float HealingFrameValue = HealingRate * DeltaTime;
    //Clamp防止超过最大值
	Character->SetHealth(FMath::Clamp(Character->GetHealth() + HealingFrameValue, 0, Character->GetMaxHealth()));
    //随即更新
	Character->UpdateHUDHealth();
    //减少所加过的值
	AmountToHeal -= HealingFrameValue;
	//关闭条件
	if (AmountToHeal <= 0 || Character->GetHealth() >= Character->GetMaxHealth())
	{
		AmountToHeal = 0.f;
		HealingRate = 0.f;
		bHealing = false;
	}
}

```

## 130.速度拾取类

```c++
	

Buff->SetInitialBaseSpeeds(GetCharacterMovement()->MaxWalkSpeed, GetCharacterMovement()->MaxWalkSpeedCrouched);
//注意构造函数里写bReplicate=true 否则客户端不消除


//碰撞同理
void ASpeedPickup::OnSphereOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{

	Super::OnSphereOverlap(OverlappedComponent, OtherActor, OtherComp, OtherBodyIndex, bFromSweep, SweepResult);
	AFirstCharacter* Character = Cast<AFirstCharacter>(OtherActor);
	if (Character)
	{
		UBuffComponent* Buff = Cast<UBuffComponent>(Character->GetBuff());
		if (Buff)
		{
			Buff->SpeedUp(WalkSpeedIncreased, CrouchSpeedIncreased, SpeedIncreasedTime);
		}
	}
	Destroyed();
}


//Buff中

//在Character中获取初始速度
void UBuffComponent::SetInitialBaseSpeeds(float WalkSpeed, float CrouchSpeed)
{
	BaseWalkSpeed = WalkSpeed;
	BaseCrouchSpeed = CrouchSpeed;
}


void UBuffComponent::SpeedUp(float WalkSpeedIncreased, float CrouchSpeedIncreased, float SpeedIncreasedTime)
{
	if (Character == nullptr)return;
	//时间到启动ResetSpeed函数速度回到初始值
	GetWorld()->GetTimerManager().SetTimer(
		SpeedBuffTimerHandle,
		this,
		&UBuffComponent::ResetSpeed,
		SpeedIncreasedTime
	);
	if (Character->GetCharacterMovement())
	{
        //服务端速度数值改变
		Character->GetCharacterMovement()->MaxWalkSpeed = WalkSpeedIncreased;
		Character->GetCharacterMovement()->MaxWalkSpeedCrouched = CrouchSpeedIncreased;
	}
    //由于速度改变只发生在服务端故RPC来相应各个客户端
	MulticastSpeedBuff(WalkSpeedIncreased, CrouchSpeedIncreased);
}

void UBuffComponent::ResetSpeed()
{
	if (Character == nullptr || Character->GetCharacterMovement() == nullptr)return;
 	//服务端速度数值改变
	Character->GetCharacterMovement()->MaxWalkSpeed = BaseWalkSpeed;
	Character->GetCharacterMovement()->MaxWalkSpeedCrouched = BaseCrouchSpeed;
	//由于速度改变只发生在服务端故RPC来相应各个客户端
	MulticastSpeedBuff(BaseWalkSpeed, BaseCrouchSpeed);
}

//由于速度改变只发生在服务端故RPC来相应各个客户端
void UBuffComponent::MulticastSpeedBuff_Implementation(float WalkSpeed, float CrouchSpeed)
{
	Character->GetCharacterMovement()->MaxWalkSpeed = WalkSpeed;
	Character->GetCharacterMovement()->MaxWalkSpeedCrouched = CrouchSpeed;
}

```

## 131.跳跃拾取类

```c++
//与速度完全一样

```

## 132.护盾

```c++
//与生命一样
```

## 133.护盾的更新

```c++
//在OnPossess


void AFirstPlayerController::OnPossess(APawn* InPawn)
{
	Super::OnPossess(InPawn);
	AFirstCharacter* FirstCharacter = Cast<AFirstCharacter>(InPawn);
	if (FirstCharacter)
	{
		SetHealth(FirstCharacter->GetMaxHealth(), FirstCharacter->GetHealth());
        
        //死亡后及时更新护盾
		SetShield(FirstCharacter->GetMaxShield(), FirstCharacter->GetShield());
		////炸弹数目更新
		//if (FirstCharacter->GetCombat())
		//{
		//	SetGrenadeAmoutText(FirstCharacter->GetCombat()->GetGrenadesAmmo());
		//}
	}
}
```

## 134.护盾拾取类

```c++
//和生命拾取类一样
```

## 135.拾取生成类

```c++
//一个数组存储各种pickup

//需要倒计时生成

//什么条件进行生成，绑定函数OnDestroyed当Actor被摧毁时开始计时生成

#include "PickupSpawnPoint.cpp"

void APickupSpawnPoint::BeginPlay()
{
	Super::BeginPlay();
	//初始调用以生成第一个
	StartPickupSpwanTimer((AActor*)nullptr);
}

void APickupSpawnPoint::SpawnPickup()
{
	//统计数组总数方便随机
	const int32 NumOfPickupArray = PickupArray.Num();
	if (NumOfPickupArray > 0)
	{
		//进行随机选择一个进行生成
		const int32 Selection = FMath::FRandRange(0, NumOfPickupArray);
		//生成前将其存储，以绑定OnDestroyed
		Pickup = GetWorld()->SpawnActor<APickup>(PickupArray[Selection], GetActorTransform());

		if (HasAuthority() && Pickup)
		{
			//绑定后在Pickup摧毁后会调用StartPickupSpwanTimer函数完成循环
			Pickup->OnDestroyed.AddDynamic(this, &APickupSpawnPoint::StartPickupSpwanTimer);
            //绑定的函数需要是UFUNCTION()
		}
	}
}

void APickupSpawnPoint::StartPickupSpwanTimer(AActor* DestroyedActor)
{
	//生成时间的随机
	const float SpawnTime = FMath::FRandRange(PickupSpawnTimeMin, PickupSpawnTimeMax);
	//时间到执行PickupSpwanTimerFinish函数
	GetWorldTimerManager().SetTimer(
		PickupSpawnTimer,
		this,
		&APickupSpawnPoint::PickupSpwanTimerFinish,
		SpawnTime
	);
}

void APickupSpawnPoint::PickupSpwanTimerFinish()
{
	if (HasAuthority())
	{
		//执行SpawnPickup完成循环
		SpawnPickup();
	}
}
//站在Pickup上等待其生成会导致其不再刷新原因：
//Pickup在生成时绑定了OverlapComponent->OnComponentBeginOverlap.AddDynamic(this, &APickup::OnSphereOverlap);
//导致还未完全生成就被Destroyed()而无法再生成且不会触发回调函数而完全被摧毁无法生成
//解决方案进行一个倒计时给他时间去生成来触发回调函数，即绑定时间晚一点点
#include "Pickup.cpp"
void APickup::BeginPlay()
{
	Super::BeginPlay();
	if (HasAuthority())
	{
		GetWorldTimerManager().SetTimer(
			OverlapTimer,
			this,
			&APickup::OverlapTimerFinish,
			OverlapTime
		);
	}
}

void APickup::OverlapTimerFinish()
{
	OverlapComponent->OnComponentBeginOverlap.AddDynamic(this, &APickup::OnSphereOverlap);
}

//找到为什么不生成的原因
//Pickup.cpp中Destroyed()
//Super::Destroyed()写成了Super::Destroy()
//导致无法触发AActor::Destroyed()函数使计时器无法开始
```

## 136.第二武器

```c++
//准备插槽方便武器的放置
//判断什么时候该是第二武器

void UCombatComponent::EquipWeapon(AWeapon* Weapon)
{
	if (Character == nullptr || Weapon == nullptr) return;
	if (CombatState != ECombatState::ECS_Unoccupied)return;

	if (EquippedWeapon != nullptr && SecondaryWeapon == nullptr)
	{
		EquipSecondaryWeapon(Weapon);
	}
	else
	{
		EquipPrimaryWeapon(Weapon);
	}
}
//装备第二武器
void UCombatComponent::EquipSecondaryWeapon(AWeapon* Weapon)
{
	SecondaryWeapon = Weapon;
	SecondaryWeapon->SetWeaponState(EWeaponState::EWS_EquippedSecondary);
	AttachActorToBackSocket(Weapon);
	SecondaryWeapon->SetOwner(Character);
	EquippedWeaponSound();
}
//将武器附在背包插槽上
void UCombatComponent::AttachActorToBackSocket(AActor* ActorToAttach)
{
	if (ActorToAttach == nullptr || Character==nullptr)return;
	const USkeletalMeshSocket* BackSocket = Character->GetMesh()->GetSocketByName(FName("BackSocket"));

	if (BackSocket)
	{
		BackSocket->AttachActor(ActorToAttach, Character->GetMesh());
	}
}

```

## 137.交换武器

```c++
//服务端和客户端的统一
//按钮函数时进行

void AFirstCharacter::SwapWeaponPressed(const FInputActionValue& Value)
{
	if (bDisableGameplay)return;
	if (Combat)
	{
		ServerSwapWeapon();
	}
}

void AFirstCharacter::ServerSwapWeapon_Implementation()
{
	Combat->SwapWeapon();
}



void UCombatComponent::SwapWeapon()
{
	if (SecondaryWeapon == nullptr || EquippedWeapon == nullptr) return;
	//换弹时和换武器的冲突
	if (CombatState == ECombatState::ECS_Reloading)
	{
		CombatState = ECombatState::ECS_Unoccupied;
		UAnimInstance* AnimInstance =Character->GetMesh()->GetAnimInstance();
		//BlendOutTime: 这是指定动画蒙太奇停止时要进行淡出的时间长度。在这段时间内，动画会平滑地淡出到停止状态。
		AnimInstance->Montage_Stop(0.3f);
	}
    //瞄准时换枪
	if (bAiming)
	{
		SetAiming(false);
	}
    //武器的交换
	AWeapon* TeampWeapon = SecondaryWeapon;
	SecondaryWeapon = EquippedWeapon;
	EquippedWeapon = TeampWeapon;
	//新主武器的更改设定
	EquippedWeapon->SetWeaponState(EWeaponState::EWS_Equipped);
	AttachActorToRightHandSocket(EquippedWeapon);
	EquippedWeapon->SetHUDWeaponAmmo();
	UpdateCarriedAmmo();
	ReloadEquippedWeapon();

    //新副武器的更改设定
	SecondaryWeapon->SetWeaponState(EWeaponState::EWS_EquippedSecondary);
	AttachActorToBackSocket(SecondaryWeapon);
	EquippedWeaponSound();
}
```

## MarkRenderStateDirty作用

```c++
/*
UE 中的 MarkRenderStateDirty() 函数是用于标记渲染状态为脏的方法。在游戏引擎中，渲染状态指的是描述如何在屏幕上绘制对象的一系列参数和设置。当渲染状态被标记为脏时，意味着需要重新计算或更新这些状态，以确保正确的渲染结果。

通常情况下，当对象的某些属性或状态发生变化时，需要调用 MarkRenderStateDirty() 函数来通知引擎需要重新计算该对象的渲染状态。这样可以确保在下一帧或渲染过程中正确地呈现对象的变化。

例如，当对象的位置、大小、颜色或材质等属性发生变化时，需要标记其渲染状态为脏，以便引擎在下一次渲染时更新相应的渲染状态，确保对象被正确绘制到屏幕上。
*/

//更新游戏状态以方便显示正确
```

## 138.初始武器

```c++


void AFirstCharacter::SpawnDefaultWeapon()
{
	//GameMode 只有在FIrstGameMode才能进行装备武器，防止在准备大厅装备武器
	AFirstGameMode* GameMode = Cast<AFirstGameMode>(UGameplayStatics::GetGameMode(this));

	UWorld* World = GetWorld();
	if (GameMode && World && DefaultWeaponClass && !bElim && Combat)
	{
		AWeapon* StartingWeapon = World->SpawnActor<AWeapon>(DefaultWeaponClass);
		StartingWeapon->bDestroyedElim = true;
		Combat->EquipWeapon(StartingWeapon);
	}
}
```

## 139.第二武器的掉落

```c++
//处理玩家死后的武器是摧毁还是丢弃
void AFirstCharacter::WeaponDropOrDestroyByElim()
{
	if (Combat)
	{
		if (Combat->EquippedWeapon)
		{
			DestroyOrDropWeapon(Combat->EquippedWeapon);
		}
		if (Combat->SecondaryWeapon)
		{
			DestroyOrDropWeapon(Combat->SecondaryWeapon);
		}
	}
}
//具体实现
void AFirstCharacter::DestroyOrDropWeapon(AWeapon* Weapon)
{
	if (Weapon->bDestroyedElim)//每个武器是否应该摧毁的属性
	{
		Weapon->Destroy();
	}
	else
	{
		Weapon->Dropped();
	}
}

```

## 140.高延迟警告

```c++
//图片导入
//设置导入格式
//图片加入HUD
//为图片制作闪烁动画，渲染不透明度




void AFirstPlayerController::ShowHighPingImage()
{
	FirstHUD = FirstHUD == nullptr ? Cast<AFirstHUD>(GetHUD()) : FirstHUD;
	bool IsValue = FirstHUD &&
		FirstHUD->CharacterOverlap &&
		FirstHUD->CharacterOverlap->HighPingImage&&
		FirstHUD->CharacterOverlap->HighPingAnim;
	if (IsValue)
	{
		FirstHUD->CharacterOverlap->HighPingImage->SetRenderOpacity(1.f);
		FirstHUD->CharacterOverlap->PlayAnimation(FirstHUD->CharacterOverlap->HighPingAnim, 0.f, 5);
	}
}

void AFirstPlayerController::StopHighPingImage()
{
	FirstHUD = FirstHUD == nullptr ? Cast<AFirstHUD>(GetHUD()) : FirstHUD;
	bool IsValue = FirstHUD &&
		FirstHUD->CharacterOverlap &&
		FirstHUD->CharacterOverlap->HighPingImage &&
		FirstHUD->CharacterOverlap->HighPingAnim;
	if (IsValue)
	{
		FirstHUD->CharacterOverlap->HighPingImage->SetRenderOpacity(0.f);
		if (FirstHUD->CharacterOverlap->IsAnimationPlaying(FirstHUD->CharacterOverlap->HighPingAnim))
		{
			FirstHUD->CharacterOverlap->StopAnimation(FirstHUD->CharacterOverlap->HighPingAnim);
		}
	}
}


//检测高网络延迟
void AFirstPlayerController::CheakHighPing(float DeltaTime)
{
	StartHighPingFrequencyTime += DeltaTime;
	if (StartHighPingFrequencyTime > HighPingFrequency)
	{
		PlayerState = PlayerState == nullptr ? GetPlayerState<APlayerState>() : PlayerState;
		if (PlayerState)
		{
			if (PlayerState->GetPing() * 4 > HighPingStandard)//这里GetPing()是Ping/4
			{
				ShowHighPingImage();
			}
		}
		StartHighPingFrequencyTime = 0.f;
	}
	bool IsPlayAnim =
		FirstHUD &&
		FirstHUD->CharacterOverlap &&
		FirstHUD->CharacterOverlap->HighPingAnim &&
		FirstHUD->CharacterOverlap->IsAnimationPlaying(FirstHUD->CharacterOverlap->HighPingAnim);
	if (IsPlayAnim)
	{
		StartHighPingDurationTime += DeltaTime;
		if (StartHighPingDurationTime > HIghPingDuration)
		{
			StartHighPingDurationTime = 0.f;
			StopHighPingImage();
		}
	}
}
```

## 141.高延迟情况下调整射击

```c++

void UCombatComponent::FireButtonPressed(bool bPressed)
{
	bFireButtonPressed = bPressed;
	if (bFireButtonPressed)
	{
		Fire();
	}
}
//Fire And FIreTimerFinish之间反复通过SetTimer调用
void UCombatComponent::Fire()
{
	if (CanFire())
	{
		bCanFire = false;
		ServerFire(HitTarget);
		LocalFire(HitTarget);
		if (EquippedWeapon)
		{
			CrosshairShootFactor = 0.75f;
		}
		StartFireHandle();
	}
}

void UCombatComponent::StartFireHandle()
{
	if (EquippedWeapon == nullptr || Character == nullptr) return;
	Character->GetWorldTimerManager().SetTimer(
		FireHandle,
		this,
		&UCombatComponent::FireTimerFinished,
		EquippedWeapon->GetRPM()
	);
}

void UCombatComponent::FireTimerFinished()
{
	if (EquippedWeapon == nullptr) return;
	bCanFire = true;
	if (bFireButtonPressed && EquippedWeapon->GetbAutoFiring())
	{
		Fire();
	}
	ReloadEquippedWeapon();
}

void UCombatComponent::ServerFire_Implementation(const FVector_NetQuantize& TracerHitTarget)
{
	MulticastFire(TracerHitTarget);
}

void UCombatComponent::MulticastFire_Implementation(const FVector_NetQuantize& TracerHitTarget)
{
	if (Character && Character->IsLocallyControlled()&&!Character->HasAuthority())return;//客户端执行时需要避免
	if (Character && Character->IsLocallyControlled() && Character->HasAuthority())return;//跳过服务端
	LocalFire(TracerHitTarget);//当客户端开枪客户端自己不执行服务端执行完成客户端服务端的统一客户端在FIre()函数中执行完成
								//服务端开枪时已经执行
}

void UCombatComponent::LocalFire(const FVector_NetQuantize& Target)
{
	if (EquippedWeapon == nullptr)return;
	if (Character && CombatState == ECombatState::ECS_Reloading && EquippedWeapon->GetWeaponType() == EWeaponType::EWT_Shot)
	{
		CombatState = ECombatState::ECS_Unoccupied;
		Character->PlayFireMontage(bAiming);
		EquippedWeapon->Fire(Target);
		return;
	}
	if (Character && CombatState == ECombatState::ECS_Unoccupied)
	{
		Character->PlayFireMontage(bAiming);
		EquippedWeapon->Fire(Target);
	}
}
```

## 142.延迟状态下散射偏离

```c++
//解决散射  因为子弹射击没有问题
//则进行分类 UENUM
//终点的计算在客户端进行即可


void UCombatComponent::HitScanFire()
{
    //客户端执行获取HitTarget
	if (EquippedWeapon)
	{
		HitTarget = EquippedWeapon->bUseScatter ? EquippedWeapon->TraceEndSctter(HitTarget) : HitTarget;
	}

	LocalFire(HitTarget);
	ServerFire(HitTarget);
}

```

## 143.散弹枪的射击改动

```c++
//为了使延迟高玩家开枪同步



void UCombatComponent::ShotGunFire()
{
	if (EquippedWeapon == nullptr)return;
	
	TArray<FVector_NetQuantize> HitTargets;
	EquippedWeapon->ShotgunTraceEndScatter(HitTarget, HitTargets);

	LocalShotgunFire(HitTargets);
	SeverShotgunFire(HitTargets);
}
//获得随机的HitTargets
void AWeapon::ShotgunTraceEndScatter(const FVector& HitTarget, TArray<FVector_NetQuantize>& HitTargets)
{
	const USkeletalMeshSocket* MuzzleFlashSocket = GetWeaponMesh()->GetSocketByName(FName("MuzzleFlash"));

	if (MuzzleFlashSocket == nullptr)return ;
	const FTransform SocketTransform = MuzzleFlashSocket->GetSocketTransform(GetWeaponMesh());
	const FVector TraceStart = SocketTransform.GetLocation();

	const FVector ToTargetNormalize = (HitTarget - TraceStart).GetSafeNormal();
	const FVector SphereCenter = TraceStart + ToTargetNormalize * DistanceToSphere;
	for (int32 i = 0; i < ShotgunScatterNums; ++i)
	{
		const FVector RandVec = UKismetMathLibrary::RandomUnitVector() * FMath::FRandRange(0.f, SphereRadius);
		const FVector EndLoc = SphereCenter + RandVec;
		const FVector ToEndLoc = EndLoc - TraceStart;
		const FVector Target = TraceStart + ToEndLoc * TRACE_LENGTH / ToEndLoc.Size();

		//DrawDebugSphere(GetWorld(), EndLoc, 16.0f, 16, FColor::Red, true);
		HitTargets.Add(Target);
	}
}
//本地射击
void UCombatComponent::LocalShotgunFire(const TArray<FVector_NetQuantize>& TracerHitTargets)
{
	AShotGun* ShotGun = Cast<AShotGun>(EquippedWeapon);
	if (EquippedWeapon == nullptr || Character == nullptr|| ShotGun==nullptr)return;

	if (CombatState == ECombatState::ECS_Unoccupied|| CombatState == ECombatState::ECS_Reloading)
	{
		Character->PlayFireMontage(bAiming);
		ShotGun->ShotgunFire(TracerHitTargets);
		CombatState = ECombatState::ECS_Unoccupied;
	}
}
//重写了shotgun的射击 为保证数组参数的统一
void AShotGun::ShotgunFire(const TArray<FVector_NetQuantize>& HitTargets)
{
	AWeapon::Fire(FVector());

	APawn* OwnerPawn = Cast<APawn>(GetOwner());
	if (OwnerPawn == nullptr)return;
	AController* InstigatorController = Cast<AController>(OwnerPawn->GetController());

	const USkeletalMeshSocket* MuzzleFlashSocket = GetWeaponMesh()->GetSocketByName(FName("MuzzleFlash"));

	if (MuzzleFlashSocket == nullptr)return;
	const FTransform SocketTransform = MuzzleFlashSocket->GetSocketTransform(GetWeaponMesh());
	const FVector Start = SocketTransform.GetLocation();
	TMap<AFirstCharacter*, int32> Hits;
	//主要改动
	for (FVector_NetQuantize HitTarget : HitTargets)
	{
		FHitResult HitResult;
		TraceWithSctter(Start, HitTarget, HitResult);
		if (HitResult.bBlockingHit)
		{
			if (ImpactParticle)
			{
				UGameplayStatics::SpawnEmitterAtLocation(
					GetWorld(),
					ImpactParticle,
					HitResult.ImpactPoint,
					HitResult.ImpactNormal.Rotation()
				);
			}

			if (HitSound)
			{
				UGameplayStatics::SpawnSoundAtLocation(
					this,
					HitSound,
					HitResult.ImpactPoint
				);
			}

			AFirstCharacter* DamagedCharacter = Cast<AFirstCharacter>(HitResult.GetActor());
			if (DamagedCharacter&&InstigatorController&&HasAuthority())
			{
				if (!Hits.Contains(DamagedCharacter))
				{
					Hits.Emplace(DamagedCharacter, 1);
				}
				else
				{
					Hits[DamagedCharacter]++;
				}
			}
		}
	}
	for (auto Pair : Hits)
	{
		if (Pair.Key && HasAuthority() && InstigatorController)
		{
			UGameplayStatics::ApplyDamage(
				Pair.Key,
				Damaged * Pair.Value,
				InstigatorController,
				this,
				UDamageType::StaticClass()
			);
		}
	}
}
//常规
void UCombatComponent::SeverShotgunFire_Implementation(const TArray<FVector_NetQuantize>& TracerHitTargets)
{
	MulticastShotgunFire(TracerHitTargets);
}
//常规
void UCombatComponent::MulticastShotgunFire_Implementation(const TArray<FVector_NetQuantize>& TracerHitTargets)
{
	if (Character && Character->IsLocallyControlled() && !Character->HasAuthority())return;//客户端执行时需要避免
	if (Character && Character->IsLocallyControlled() && Character->HasAuthority())return;//跳过服务端
	LocalShotgunFire(TracerHitTargets);
}
```

## 144.客户端高延迟下的射击子弹数量

```c++
//Ammo的复制取消
//客户端记录下执行次数
//在服务端传回消息时进行修正



void AWeapon::AddAmmo(int32 AmmoToAdd)
{
	Ammo = FMath::Clamp(Ammo + AmmoToAdd, 0, MagCapactity);
	SetHUDWeaponAmmo();
	ClientUpdateAmmo(AmmoToAdd);
}

void AWeapon::ClientAddAmmo_Implementation(int32 AmmoToAdd)
{
	if (HasAuthority())return;
	FirstOwnerCharacter = FirstOwnerCharacter == nullptr ? Cast<AFirstCharacter>(GetOwner()) : FirstOwnerCharacter;
	if (FirstOwnerCharacter && FirstOwnerCharacter->GetCombat() && IsFull())
	{
		FirstOwnerCharacter->GetCombat()->JumpToShellOfEnd();
	}
	SetHUDWeaponAmmo();
}

void AWeapon::SpendRound()
{
	Ammo = FMath::Clamp(Ammo - 1, 0, MagCapactity);
	SetHUDWeaponAmmo();
	if (HasAuthority())
	{
		ClientUpdateAmmo(Ammo);
	}
	else
	{
		++Sequence;
	}
}
//客户端和服务端之间延迟中执行次数来让客户端和服务端统一
void AWeapon::ClientUpdateAmmo_Implementation(int32 ServerAmmo)
{
	if (HasAuthority())return;
 	Ammo = ServerAmmo;
	FirstOwnerCharacter = FirstOwnerCharacter == nullptr ? Cast<AFirstCharacter>(GetOwner()) : FirstOwnerCharacter;
	if (FirstOwnerCharacter && FirstOwnerCharacter->GetCombat() && FirstOwnerCharacter->GetCombat()->GetCombatState() == ECombatState::ECS_Unoccupied)
	{
		--Sequence;
		Ammo -= Sequence;
	}

	SetHUDWeaponAmmo();
}
```

## 145.客户端高延迟下的换弹

```c++
//多加个bLocalReload控制bFABRIK来控制手部动作
```

## 146.延迟补偿

```c++
//设置结构体

//存储盒体信息
USTRUCT(BlueprintType)
struct FBoxInformation
{
	GENERATED_BODY()

	UPROPERTY()
	FVector BoxLocation;
	UPROPERTY()
	FRotator BoxRoatation;
	UPROPERTY()
	FVector BoxExtent;


};

//人物盒体碰撞体积集合和当前时间
USTRUCT(BlueprintType)
struct FFramePackage
{
	GENERATED_BODY()

	UPROPERTY()
	float Time;

	UPROPERTY()
	TMap<FName, FBoxInformation> BoxInfo;
};

#include "LogCompensationComponent.cpp"
void ULogCompensationComponent::SaveFramePackage(FFramePackage& FramePackage)
{

	Character = Character == nullptr ? Cast<AFirstCharacter>(GetOwner()) : Character;
	if (Character)
	{
        //存储位置的时间
		FramePackage.Time = GetWorld()->GetTimeSeconds();
		//遍历所有盒体方便逐个存入
		for (TPair<FName, UBoxComponent*>& BoxInfo : Character->BoxCollision)
		{
			FBoxInformation BoxInformation;
			BoxInformation.BoxLocation = BoxInfo.Value->GetComponentLocation();
			BoxInformation.BoxRoatation = BoxInfo.Value->GetComponentRotation();
			BoxInformation.BoxExtent = BoxInfo.Value->GetScaledBoxExtent();
            //盒体信息的导入
			FramePackage.BoxInfo.Add(BoxInfo.Key, BoxInformation);
		}
	}
}

//将每一帧的盒体和时间进行打包处理
//存储在双链表中方便删除和添加
void ULogCompensationComponent::FramePackageToHistory()
{
	if (BoxHistory.Num() <= 1)
	{
		FFramePackage ThisPackage;
		SaveFramePackage(ThisPackage);
		BoxHistory.AddHead(ThisPackage);

	}
	else
	{
		float BoxHistoryLength = BoxHistory.GetHead()->GetValue().Time - BoxHistory.GetHead()->GetValue().Time;
		while (BoxHistoryLength > MaxHistoryTime)
		{
			BoxHistory.RemoveNode(BoxHistory.GetTail());
			BoxHistoryLength = BoxHistory.GetHead()->GetValue().Time - BoxHistory.GetHead()->GetValue().Time;
		}
		FFramePackage ThisPackage;
		SaveFramePackage(ThisPackage);
		BoxHistory.AddHead(ThisPackage);
		//ShowFramePackage(ThisPackage, FColor::Red);
	}
}


```

## 147.倒带找到被击中时刻的FramePackage

```c++

//Yonger和Older指针 反复寻找找到被击中时刻两边后进行插值
//记得考虑特殊情况 ：


//寻找击中的时间
void ULogCompensationComponent::RewindingTime(AFirstCharacter* HitCharacter,float HitTime)
{
	bool bReturn =
		HitCharacter == nullptr ||
		HitCharacter->GetLogCompensation() == nullptr ||
		HitCharacter->GetLogCompensation()->BoxHistory.GetHead() == nullptr ||
		HitCharacter->GetLogCompensation()->BoxHistory.GetTail() == nullptr;
	if (bReturn)return;
	//是否需要插值
	bool bShouldInterpolate = true;
	const TDoubleLinkedList<FFramePackage>& History = HitCharacter->GetLogCompensation()->BoxHistory;
	const float OldestTime = History.GetTail()->GetValue().Time;
	const float YoungestTime = History.GetHead()->GetValue().Time;
	if (OldestTime > HitTime)
	{
		//超过最大检测时间
		return;
	}
	TDoubleLinkedList<FFramePackage>::TDoubleLinkedListNode* HitPackage;

	//两种特殊情况单独考虑
	if (YoungestTime <= HitTime)
	{
		//击中时间比最新记录的时间还小
		HitPackage = History.GetHead();
		bShouldInterpolate = false;
	}
	if (OldestTime == HitTime)
	{
		HitPackage = History.GetTail();
		bShouldInterpolate = false;
	}


	if(bShouldInterpolate)
	{
		//在击中时刻之前
		TDoubleLinkedList<FFramePackage>::TDoubleLinkedListNode* Younger = History.GetHead();
		//在击中时刻之后
		TDoubleLinkedList<FFramePackage>::TDoubleLinkedListNode* Older = Younger;

		while (Older->GetValue().Time > HitTime)
		{
			if (Older->GetNextNode() == nullptr)return;
			Older = Older->GetNextNode();
			if (Older->GetValue().Time > HitTime)
			{
				Younger = Older;
			}
		}

		if (Older->GetValue().Time == HitTime)
		{
			HitPackage = Older;
			bShouldInterpolate = false;
		}
	}
	
	if (bShouldInterpolate)
	{
		//进行在Yonger和Older之间进行插值来找到


	}
	
}
```

## 148.对Time进行插值(Interp)

```c++


//进行在Yonger和Older之间进行插值来创建HitFrame
FFramePackage ULogCompensationComponent::GetInterpHiFrame(const FFramePackage& OlderFrame, const FFramePackage& YoungerFrame, float HitTime)
{
	//两者的时间距离
	const float Distance = OlderFrame.Time - YoungerFrame.Time;
	//占比
	const float InterpFraction = FMath::Clamp((OlderFrame.Time - HitTime) / Distance,0.f,1.f);

	FFramePackage HitFrame;
	HitFrame.Time = HitTime;

	for (auto& BoxInfo : YoungerFrame.BoxInfo)
	{
		const FName& InterpName = BoxInfo.Key;
		FBoxInformation BoxInformation;

		const FBoxInformation& OlderInfo = OlderFrame.BoxInfo[InterpName];
		const FBoxInformation& YoungerInfo = YoungerFrame.BoxInfo[InterpName];
		//完善输出量
		BoxInformation.BoxLocation = FMath::VInterpTo(OlderInfo.BoxLocation, YoungerInfo.BoxLocation, 1.f, InterpFraction);
		BoxInformation.BoxRoatation = FMath::RInterpTo(OlderInfo.BoxRoatation, YoungerInfo.BoxRoatation, 1.f, InterpFraction);
		BoxInformation.BoxExtent = YoungerInfo.BoxExtent;


		HitFrame.BoxInfo.Add(InterpName, BoxInformation);
	}
	return HitFrame;

}
```

## 149.确认射击击中

```c++
//检查是否击中目标
FServerSideRewingResult ULogCompensationComponent::HitConfirmed(AFirstCharacter* HitCharacter, const FVector_NetQuantize& TraceStart, const FVector_NetQuantize& HitLocation, FFramePackage& HitPackage)
{
	if(HitCharacter==nullptr) return FServerSideRewingResult();

	FFramePackage CurrentPackage;
    //缓存并获取Box
	CacheBoxCollison(HitCharacter, CurrentPackage);
	MoveBoxCollision(HitCharacter, HitPackage);

	//是否击中头部
    //对头部开启Channel
	UBoxComponent* HeadBox = HitCharacter->BoxCollision[FName("head")];
	HeadBox->SetCollisionEnabled(ECollisionEnabled::QueryAndPhysics);
	HeadBox->SetCollisionResponseToChannel(ECollisionChannel::ECC_EngineTraceChannel1, ECollisionResponse::ECR_Block);
	FHitResult HitResult;
	const FVector EndLoc = TraceStart + (HitLocation - TraceStart) * 1.25;

	UWorld* World = GetWorld();
	if (World)
	{	
		World->LineTraceSingleByChannel(
			HitResult,
			TraceStart,
			EndLoc,
			ECollisionChannel::ECC_EngineTraceChannel1
		);

		if (HitResult.bBlockingHit)
		{
			//击打中头部
			ResetBoxCollison(HitCharacter, CurrentPackage);
			return FServerSideRewingResult{ true ,true };
		}
		else
		{
			//检查其他部位
			//开启全部通道方便检测是否命中
			for (auto& HitBoxPair : HitCharacter->BoxCollision)
			{
				HitBoxPair.Value->SetCollisionEnabled(ECollisionEnabled::QueryAndPhysics);
				HitBoxPair.Value->SetCollisionResponseToChannel(ECollisionChannel::ECC_EngineTraceChannel1, ECollisionResponse::ECR_Block);
			}

			if (World)
			{
				World->LineTraceSingleByChannel(
					HitResult,
					TraceStart,
					EndLoc,
					ECollisionChannel::ECC_EngineTraceChannel1
				);
				if (HitResult.bBlockingHit)
				{
					//击打中其他部位
					ResetBoxCollison(HitCharacter, CurrentPackage);
					return FServerSideRewingResult{ true ,false };
				}

			}
		}
		//没击打中任何部位
		ResetBoxCollison(HitCharacter, CurrentPackage);
		return FServerSideRewingResult{ false ,false };
	}
}

//进行缓存
void ULogCompensationComponent::CacheBoxCollison(AFirstCharacter* HitCharacter, FFramePackage& OutPackage)
{
	if (HitCharacter == nullptr)return;

	for (auto& HitBoxPair : HitCharacter->BoxCollision)
	{
		if (HitBoxPair.Value)
		{
			FBoxInformation HitBoxInfo;
			HitBoxInfo.BoxLocation = HitBoxPair.Value->GetComponentLocation();
			HitBoxInfo.BoxRoatation = HitBoxPair.Value->GetComponentRotation();
			HitBoxInfo.BoxExtent = HitBoxPair.Value->GetScaledBoxExtent();
			OutPackage.BoxInfo.Add(HitBoxPair.Key, HitBoxInfo);
		}
	}
}
//移动Box到HitBox位置
void ULogCompensationComponent::MoveBoxCollision(AFirstCharacter* HitCharacter, FFramePackage& Package)
{
	if (HitCharacter == nullptr)return;

	for (auto& HitBoxPair : HitCharacter->BoxCollision)
	{
		if (HitBoxPair.Value)
		{
			HitBoxPair.Value->SetWorldLocation(Package.BoxInfo[HitBoxPair.Key].BoxLocation);
			HitBoxPair.Value->SetWorldRotation(Package.BoxInfo[HitBoxPair.Key].BoxRoatation);
			HitBoxPair.Value->SetWorldScale3D(Package.BoxInfo[HitBoxPair.Key].BoxExtent);
		}

	}
}
//改变了Collision现在进行复原
void ULogCompensationComponent::ResetBoxCollison(AFirstCharacter* HitCharacter, FFramePackage& Package)
{
	if (HitCharacter == nullptr)return;

	for (auto& HitBoxPair : HitCharacter->BoxCollision)
	{
		if (HitBoxPair.Value)
		{
			HitBoxPair.Value->SetWorldLocation(Package.BoxInfo[HitBoxPair.Key].BoxLocation);
			HitBoxPair.Value->SetWorldRotation(Package.BoxInfo[HitBoxPair.Key].BoxRoatation);
			HitBoxPair.Value->SetWorldScale3D(Package.BoxInfo[HitBoxPair.Key].BoxExtent);
			HitBoxPair.Value->SetCollisionEnabled(ECollisionEnabled::NoCollision);
		}

	}
}

```

## 150.射击附加伤害

```c++
//

//伤害造成
void ULogCompensationComponent::ServerHitRequest_Implementation(AFirstCharacter* HitCharacter, const FVector_NetQuantize TraceStart, const FVector_NetQuantize& HitLocation, float HitTime, AWeapon* DamagedCasuer)
{
	if (HitCharacter == nullptr|| DamagedCasuer==nullptr)return;
	//获取是否击中
	FServerSideRewingResult HitResult = ServerSideRewing(HitCharacter, TraceStart, HitLocation, HitTime);

	if (HitResult.bHitConfirmed)
	{
		Character = Character == nullptr ? Cast<AFirstCharacter>(GetOwner()) : Character;

		if (!Character) return;
		//应用伤害
		UGameplayStatics::ApplyDamage(
			HitCharacter,
			DamagedCasuer->GetDamaged(),
			Character->Controller,
			DamagedCasuer,
			UDamageType::StaticClass()
		);
	}
}

```

## 151.散弹枪的倒带(Rewing)

```c++
//Shotgun传递是否击中的信息
FShotgunServerSideRewingResult ULogCompensationComponent::ShotgunServerSideRewing(TArray<AFirstCharacter*> HitCharacters, const FVector_NetQuantize& TraceStart, const TArray<FVector_NetQuantize>& HitLocations, float HitTime)
{
	TArray<FFramePackage> HitFramePackages;
	for (AFirstCharacter* HitCharacter : HitCharacters)
	{
		HitFramePackages.Add(GetHitFramePackage(HitCharacter, HitTime));
	}
	return ShotgunHitConfirmed(HitFramePackages, TraceStart, HitLocations);
}

//Shotgun判断是否击中目标的
FShotgunServerSideRewingResult ULogCompensationComponent::ShotgunHitConfirmed(TArray<FFramePackage>& HitPackages, const FVector_NetQuantize& TraceStart, const TArray<FVector_NetQuantize>& HitLocations)
{
	TArray<FFramePackage> CurrentPackages;
	for (FFramePackage HitPackage : HitPackages)
	{
		if (HitPackage.Character == nullptr) return FShotgunServerSideRewingResult();

		FFramePackage CurrentPackage;
		//关键步骤：设置为碰撞时角色方便还原时候调用的一一对应
		CurrentPackage.Character = HitPackage.Character;
		//缓存现在存在的位置
		CacheBoxCollison(HitPackage.Character, CurrentPackage);
		//移动到发生碰撞时间的位置
		MoveBoxCollision(HitPackage.Character, HitPackage);
		CurrentPackages.Add(CurrentPackage);


		//是否击中头部
		UBoxComponent* HeadBox = HitPackage.Character->BoxCollision[FName("head")];
		HeadBox->SetCollisionEnabled(ECollisionEnabled::QueryAndPhysics);
		HeadBox->SetCollisionResponseToChannel(ECollisionChannel::ECC_Shot, ECollisionResponse::ECR_Block);
	}
	FShotgunServerSideRewingResult  ShotgunServerSideRewingResult;
	UWorld* World = GetWorld();
	if (World)
	{
		for (FVector HitLocation : HitLocations)
		{
			FHitResult HitResult;
			FVector End = TraceStart + (HitLocation - TraceStart) * 1.25;
			World->LineTraceSingleByChannel
			(
				HitResult,
				TraceStart,
				End,
				ECollisionChannel::ECC_Shot
			);

			//击中头部
			if (HitResult.bBlockingHit)
			{
				AFirstCharacter* BlockingHitCharacter =Cast<AFirstCharacter>(HitResult.GetActor());
				if (BlockingHitCharacter)
				{
					if (!ShotgunServerSideRewingResult.HeadShots.Contains(BlockingHitCharacter))
					{
						//首次加入
						ShotgunServerSideRewingResult.HeadShots.Emplace(BlockingHitCharacter, 1);
					}
					else
					{
						//已经存在时
						ShotgunServerSideRewingResult.HeadShots[BlockingHitCharacter]++;
					}
				}		
			}
		}

		//其他部位
		for (FFramePackage HitPackage : HitPackages)
		{
			//重新设置
			ResetBoxCollison(HitPackage.Character, HitPackage);
			//对其余部位开启检测是否碰撞
			for (auto PairBox : HitPackage.Character->BoxCollision)
			{
				if (PairBox.Key == FName("head")) continue;
				PairBox.Value->SetCollisionEnabled(ECollisionEnabled::QueryAndPhysics);
				PairBox.Value->SetCollisionResponseToChannel(ECollisionChannel::ECC_Shot, ECollisionResponse::ECR_Block);
			}
		}

		for (FVector HitLocation : HitLocations)
		{
			FHitResult HitResult;
			FVector End = TraceStart + (HitLocation - TraceStart) * 1.25;
			World->LineTraceSingleByChannel
			(
				HitResult,
				TraceStart,
				End,
				ECollisionChannel::ECC_Shot
			);

			//击中其他部位
			if (HitResult.bBlockingHit)
			{
				AFirstCharacter* BlockingHitCharacter = Cast<AFirstCharacter>(HitResult.GetActor());
				if (BlockingHitCharacter)
				{
					if (!ShotgunServerSideRewingResult.BodyShots.Contains(BlockingHitCharacter))
					{
						//首次加入
						ShotgunServerSideRewingResult.BodyShots.Emplace(BlockingHitCharacter, 1);
					}
					else
					{
						//已经存在时
						ShotgunServerSideRewingResult.BodyShots[BlockingHitCharacter]++;
					}
				}		
			}
		}	
		for (FFramePackage CurrentPackage : CurrentPackages)
		{
			/*
				//关键步骤：设置为碰撞时角色方便还原时候调用的一一对应
				CurrentPackage.Character = HitPackage.Character;
			*/
			//重新设置
			ResetBoxCollison(CurrentPackage.Character, CurrentPackage);
		}
	}
	return ShotgunServerSideRewingResult;
}
```

## 152.散弹枪的附加伤害

```c++

//造成伤害
void ULogCompensationComponent::ServerShotgunHitRequest_Implementation(const TArray<AFirstCharacter*>& HitCharacters,const FVector_NetQuantize& TraceStart,const TArray<FVector_NetQuantize>& HitLocations,float HitTime,AWeapon* DamagedCauser)
{
	if (Character == nullptr || Character->Controller == nullptr)return;
	//获取信息
	FShotgunServerSideRewingResult ShotgunServerSideRewingResult = ShotgunServerSideRewing(HitCharacters, TraceStart, HitLocations, HitTime);

	for (AFirstCharacter* HitCharacter : HitCharacters)	
	{
		float TotalDamage = 0.f;
		//伤害叠加未完成
		float HeadDamage = ShotgunServerSideRewingResult.HeadShots[HitCharacter] * DamagedCauser->GetDamaged();
		TotalDamage += HeadDamage;

		float BodyDamage = ShotgunServerSideRewingResult.BodyShots[HitCharacter] * DamagedCauser->GetDamaged();
		TotalDamage += BodyDamage;

		if (HitCharacter == nullptr || DamagedCauser == nullptr )continue;
		//造成伤害
		UGameplayStatics::ApplyDamage
		(
			HitCharacter,
			TotalDamage,
			Character->Controller,
			DamagedCauser,
			UDamageType::StaticClass()
		);
	}
}
```

## 153.预测弹丸轨迹

```c++

FPredictProjectilePathResult PathResult;
FPredictProjectilePathParams PathParams;
//设置预测时间
PathParams.MaxSimTime = 4.f;
PathParams.SimFrequency = 30.f;
PathParams.DrawDebugTime = 5.f;
PathParams.DrawDebugType = EDrawDebugTrace::ForDuration;
//忽视的碰撞Actor
PathParams.ActorsToIgnore.Add(this);
//添加检测通道
PathParams.bTraceWithChannel = true;
PathParams.bTraceWithCollision = true;
PathParams.TraceChannel = ECollisionChannel::ECC_Visibility;
//设置初始化速度和初始位置
PathParams.LaunchVelocity = GetActorForwardVector()*InitialSpeed;
PathParams.StartLocation = GetActorLocation();
//画出来的球半径
PathParams.ProjectileRadius = 5.f;

UGameplayStatics::PredictProjectilePath(this, PathParams, PathResult);
```

## 154.PostEditChangeProperty

```c++
#if WITH_EDITOR
void AProjectlieBullet::PostEditChangeProperty(FPropertyChangedEvent& Event)
{
	Super::PostEditChangeProperty(Event);

	FName PropertyName = Event.Property !=nullptr ? Event.Property->GetFName():NAME_None;

	if (PropertyName == GET_MEMBER_NAME_CHECKED(AProjectlieBullet, InitialSpeed))
	{
		ProjectileMovementComponent->InitialSpeed = InitialSpeed;
		ProjectileMovementComponent->MaxSpeed = InitialSpeed;
	}
}
#endif
```

## 155.不同的生成Projectile

```c++

void AProjectileWeapon::Fire(const FVector& HitTarget)
{
	Super::Fire(HitTarget);


	//获取插槽信息
	const USkeletalMeshSocket* MeshScoket= GetWeaponMesh()->GetSocketByName(FName("MuzzleFlash"));
	APawn* InstigatorPawn = Cast<APawn>(GetOwner());

	if (MeshScoket)
	{
		FTransform WeaponTransform = MeshScoket->GetSocketTransform(GetWeaponMesh());
		//设置子弹的主人
		FActorSpawnParameters SpawnParameters;
		SpawnParameters.Owner = GetOwner();
		SpawnParameters.Instigator = InstigatorPawn;
		FVector ToTarget = HitTarget - WeaponTransform.GetLocation();
		FRotator TargetRotation = ToTarget.Rotation();
		AProjectlie* Projectile = nullptr;

		UWorld* World = GetWorld();
		if (World)
		{
			if (bUseServerSideRewing)
			{
				if (InstigatorPawn->HasAuthority())//Server -Weapon SSR
				{
					if (InstigatorPawn->IsLocallyControlled()) //Server -LocallyController -Replicated -No SSR
					{
						Projectile = World->SpawnActor<AProjectlie>(ProjectileClass,WeaponTransform.GetLocation(),TargetRotation,SpawnParameters);
						Projectile->bUseServerSideRewing = false;
						Projectile->Damage = Damaged;
					}
					else//Server -No LocallyController -No Replicated - No SSR
					{
						Projectile = World->SpawnActor<AProjectlie>(ServerSideRewingProjectileClass, WeaponTransform.GetLocation(), TargetRotation, SpawnParameters);
						Projectile->bUseServerSideRewing = false;
					}
				}
				else//Client -Weapon SSR
				{
					if (InstigatorPawn->IsLocallyControlled())//Client -LocallyController -No Replicated -SSR
					{
						Projectile = World->SpawnActor<AProjectlie>(ServerSideRewingProjectileClass, WeaponTransform.GetLocation(), TargetRotation, SpawnParameters);
						Projectile->TraceStart = WeaponTransform.GetLocation();
						Projectile->InitialVelocity = Projectile->GetActorForwardVector() * Projectile->InitialSpeed;
						Projectile->Damage = Damaged;
						Projectile->bUseServerSideRewing = true;
					}
					else//Client -LocallyController -No Replicated -No SSR
					{
						Projectile = World->SpawnActor<AProjectlie>(ServerSideRewingProjectileClass, WeaponTransform.GetLocation(), TargetRotation, SpawnParameters);
						Projectile->bUseServerSideRewing = false;
					}
				}
			}
			else
			{
				if (InstigatorPawn->HasAuthority())//Weapon No SSR
				{
					Projectile = World->SpawnActor<AProjectlie>(ProjectileClass, WeaponTransform.GetLocation(), TargetRotation, SpawnParameters);
					Projectile->bUseServerSideRewing = false;
					Projectile->Damage = Damaged;
				}
			}		
		}
	}	
}
```

## 156.Projectile的倒带(Rewing)

```c++
//Projectile传递是否击中的信息
FServerSideRewingResult ULogCompensationComponent::ProjectileServerSideRewing(AFirstCharacter* HitCharacter, const FVector_NetQuantize& TraceStart, const FVector_NetQuantize100& InitialSpeed, float HitTime)
{
	const FFramePackage HitPackage = GetHitFramePackage(HitCharacter, HitTime);

	return ProjectileHitConfirmed(HitPackage,HitCharacter,TraceStart,InitialSpeed,HitTime);
}

//Projectile判断是否击中目标的
FServerSideRewingResult ULogCompensationComponent::ProjectileHitConfirmed(const FFramePackage& HitPackage, AFirstCharacter* HitCharacter, const FVector_NetQuantize& TraceStart, const FVector_NetQuantize100& InitialSpeed, float HitTime)
{
	if (HitCharacter == nullptr)return FServerSideRewingResult();
	FFramePackage CurrentPackage;
	//调整碰撞箱位置到碰撞时刻和缓存现在位置
	CacheBoxCollison(HitCharacter, CurrentPackage);
	MoveBoxCollision(HitCharacter, HitPackage);

	//头部通道开启
	UBoxComponent* HeadCollison = HitCharacter->BoxCollision[FName("head")];
	HeadCollison->SetCollisionEnabled(ECollisionEnabled::QueryAndPhysics);
	HeadCollison->SetCollisionResponseToChannel(ECC_Shot, ECollisionResponse::ECR_Block);

	//开始检测碰撞
	UWorld* World = GetWorld();
	if (World)
	{
		FPredictProjectilePathParams PathParams;
		FPredictProjectilePathResult PathResult;

		//设置预测时间
		PathParams.MaxSimTime = MaxHistoryTime;
		PathParams.SimFrequency = 15.f;
		PathParams.DrawDebugTime = 5.f;
		PathParams.DrawDebugType = EDrawDebugTrace::ForDuration;
		//忽视的碰撞Actor
		PathParams.ActorsToIgnore.Add(GetOwner());
		//添加检测通道
		PathParams.bTraceWithChannel = true;
		PathParams.bTraceWithCollision = true;
		PathParams.TraceChannel = ECC_Shot;
		//设置初始化速度和初始位置
		PathParams.LaunchVelocity = InitialSpeed;
		PathParams.StartLocation = TraceStart;
		//画出来的球半径
		PathParams.ProjectileRadius = 5.f;

		UGameplayStatics::PredictProjectilePath(World, PathParams, PathResult);

		if (PathResult.HitResult.bBlockingHit)//击中了头部
		{
			//还原
			ResetBoxCollison(HitCharacter, CurrentPackage);
			return FServerSideRewingResult{ true,true };
		}
		else//没有击中头部
		{
			for (auto& HitBox : HitCharacter->BoxCollision)
			{
				//对其余部位开启碰撞
				HitBox.Value->SetCollisionEnabled(ECollisionEnabled::QueryAndPhysics);
				HitBox.Value->SetCollisionResponseToChannel(ECC_Shot, ECollisionResponse::ECR_Block);
			}

			UGameplayStatics::PredictProjectilePath(World, PathParams, PathResult);

			if (PathResult.HitResult.bBlockingHit)//击中了其余部位
			{
				//还原
				ResetBoxCollison(HitCharacter, CurrentPackage);
				return  FServerSideRewingResult{ false,true };
			}
			else//什么都没击中
			{
				//还原
				ResetBoxCollison(HitCharacter, CurrentPackage);
				return  FServerSideRewingResult{ false,false };
			}
		}
	}
	return FServerSideRewingResult();
}
 
//Projectile 应用伤害
void ULogCompensationComponent::ServerProjectileHitRequest_Implementation(AFirstCharacter* HitCharacter, const FVector_NetQuantize& TraceStart, const FVector_NetQuantize100& InitialSpeed, float HitTime,AWeapon* DamageCauser)
{
	if (HitCharacter == nullptr)return;
	//获取信息
	FServerSideRewingResult HitResult = ProjectileServerSideRewing(HitCharacter, TraceStart, InitialSpeed, HitTime);

	if (HitResult.bHitConfirmed)
	{
		Character = Character == nullptr ? Cast<AFirstCharacter>(GetOwner()) : Character;

		if (!Character) return;
		//应用伤害
		UGameplayStatics::ApplyDamage(
			HitCharacter,
			DamageCauser->GetDamaged(),
			Character->Controller,
			DamageCauser,
			UDamageType::StaticClass()
		);
	}
}
```

## 157.在高延迟情况下关闭倒带

```c++
//利用委托  告诉

void AFirstPlayerController::ServerPingTooHigh_Implementation(bool bPingHigh)
{
	HighPingDelegate.Broadcast(bPingHigh);
}

//在需要的地方进行绑定
	if (FirstOwnerCharacter)
	{
		FirstOwnerController = FirstOwnerController == nullptr ? Cast<AFirstPlayerController>(FirstOwnerCharacter->GetController()) : FirstOwnerController;
		if (FirstOwnerController)
		{
			if (FirstOwnerCharacter->GetEquippedWeapon() && !FirstOwnerController->HighPingDelegate.IsBound())
			{
				FirstOwnerController->HighPingDelegate.AddDynamic(this, &AWeapon::OnPingTooHigh);
			}
		}
	}
}

void AWeapon::OnPingTooHigh(bool bPingHigh)
{
	bUseServerSideRewing = !bPingHigh;
}

```

## 158.交换武器动画

```c++

//服务端进行
void UCombatComponent::SwapWeapon()
{
	if (SecondaryWeapon == nullptr || EquippedWeapon == nullptr) return;

	

	if (CombatState == ECombatState::ECS_Reloading&&Character->HasAuthority())
	{
		CombatState = ECombatState::ECS_Unoccupied;
		bLocalReload = false;
		UAnimInstance* AnimInstance = Character->GetMesh()->GetAnimInstance();
		//BlendOutTime: 这是指定动画蒙太奇停止时要进行淡出的时间长度。在这段时间内，动画会平滑地淡出到停止状态。
		AnimInstance->Montage_Stop(0.3f);
	}


	if (Character && CombatState != ECombatState::ECS_SawpWeapon && Character->HasAuthority()&& Character->bSwapFinished)
	{
		Character->bSwapFinished = false;
		CombatState = ECombatState::ECS_SawpWeapon;
		Character->PlaySwapMontage();
	}
}

void UCombatComponent::SwapFinish()
{
	if (Character && Character->HasAuthority())
	{
		CombatState = ECombatState::ECS_Unoccupied;
	}
	if(Character) Character->bSwapFinished = true;
}

void UCombatComponent::SwapAttachFinish()
{
	if (Character && Character->HasAuthority())
	{
		if (bAiming)
		{
			SetAiming(false);
		}

		AWeapon* TeampWeapon = SecondaryWeapon;
		SecondaryWeapon = EquippedWeapon;
		EquippedWeapon = TeampWeapon;

		EquippedWeapon->SetWeaponState(EWeaponState::EWS_Equipped);
		AttachActorToRightHandSocket(EquippedWeapon);

		EquippedWeapon->SetHUDWeaponAmmo();
		UpdateCarriedAmmo();
		ReloadEquippedWeapon();

		SecondaryWeapon->SetWeaponState(EWeaponState::EWS_EquippedSecondary);
		AttachActorToBackSocket(SecondaryWeapon);
		EquippedWeaponSound();
	}
}

```

## 159.退出游戏按钮

```c++
#include "ReturnToMainMenu.h"

void UReturnToMainMenu::MenuSetup()
{
	UWorld* World = GetWorld();
	if (World)
	{
		PlayerController = PlayerController == nullptr ? World->GetFirstPlayerController() : PlayerController;
		if (PlayerController)
		{
			AddToViewport();
			SetVisibility(ESlateVisibility::Visible);
			bIsFocusable = true;

			FInputModeGameAndUI InputData;
			InputData.SetWidgetToFocus(TakeWidget());
			//InputData.SetLockMouseToViewportBehavior(EMouseLockMode::DoNotLock);
			PlayerController->SetShowMouseCursor(true);
			PlayerController->SetInputMode(InputData);
		}
	}

	if (ReturnButton)
	{
		ReturnButton->OnClicked.AddDynamic(this, &UReturnToMainMenu::ReturnButtonClicked);
	}

	UGameInstance* Instance = GetGameInstance();
	if (Instance)
	{
		SessionSubsystem = Instance->GetSubsystem<UMultuPlayerSessionsSubsystem>();
		if (SessionSubsystem)
		{
			SessionSubsystem->MultuPlayerOnStartSessionComplete.AddDynamic(this, &UReturnToMainMenu::OnDestroySession);
		}	
	}
}

bool UReturnToMainMenu::Initialize()
{
	if (!Super::Initialize())
	{
		return false;
	}
	return true;
}

void UReturnToMainMenu::MenuTearDown()
{
	UWorld* World = GetWorld();
	if (World)
	{
		PlayerController = PlayerController == nullptr ? World->GetFirstPlayerController() : PlayerController;
		if (PlayerController)
		{
			RemoveFromParent();

			FInputModeGameOnly InputData;
			PlayerController->SetShowMouseCursor(false);
			PlayerController->SetInputMode(InputData);
		}
	}
}

void UReturnToMainMenu::ReturnButtonClicked()
{
	ReturnButton->SetIsEnabled(false);

	if (SessionSubsystem)
	{
		SessionSubsystem->DestroyGameSession();
	}
}

void UReturnToMainMenu::OnDestroySession(bool bWasSuccess)
{
	if (!bWasSuccess)
	{
		ReturnButton->SetIsEnabled(true);
		return;
	}

	UWorld* World = GetWorld();
	if (World)
	{	
		AGameModeBase* GameMode = World->GetAuthGameMode<AGameModeBase>();
		if (GameMode)//服务端
		{
			GameMode->ReturnToMainMenuHost();
		}
		else//客户端
		{
			PlayerController = PlayerController == nullptr ? World->GetFirstPlayerController() : PlayerController;
			if (PlayerController)
			{
			}
			PlayerController->ClientReturnToMainMenuWithTextReason(FText());
		}
	}
}

```

## 160.显示退出游戏Widget

```c++
void AFirstPlayerController::SetupInputComponent()
{
	Super::SetupInputComponent();
	if (InputComponent == nullptr)return;

	InputComponent->BindAction("ReturnToMainMenu", EInputEvent::IE_Pressed, this, &AFirstPlayerController::ShowReturnToMainMenu);
	

}

void AFirstPlayerController::ShowReturnToMainMenu()
{
	if (ReturnToMainMenuWidget == nullptr)return;
	if (ReturnToMainMenu == nullptr)
	{
		ReturnToMainMenu = CreateWidget<UReturnToMainMenu>(this, ReturnToMainMenuWidget);
		
	}
	
	if (bShowReturnToMainMenu)
	{
		ReturnToMainMenu->MenuSetup();
	}
	else
	{
		ReturnToMainMenu->MenuTearDown();
	}
	bShowReturnToMainMenu = !bShowReturnToMainMenu;
}
```

## 161.退出游戏的一些处理和优化

```c++
//退出游戏枪械的掉落
//退出游戏分数榜的清除


void AFirstCharacter::ServerLeftGame_Implementation()
{
	
	AFirstGameMode* GameMode = GetWorld()->GetAuthGameMode<AFirstGameMode>();
	FirstPlayerState = FirstPlayerState==nullptr? GetPlayerState<AFirstPlayerState>(): FirstPlayerState;
	if (GameMode && FirstPlayerState)
	{
		//操作在服务器
		GameMode->PlayerLeftGame(FirstPlayerState);
	}
}

void AFirstGameMode::PlayerLeftGame(AFirstPlayerState* PlayerLeaving)
{
	AFirstGameState* FirstGameState = GetGameState<AFirstGameState>();
	if (FirstGameState)
	{
		FirstGameState->TopScorePlayer.Remove(PlayerLeaving);
	}
	AFirstCharacter* LeaveCharacter = Cast<AFirstCharacter>(PlayerLeaving->GetPawn());
	if (LeaveCharacter)
	{
		LeaveCharacter->Elim(true);
	}
}


```

## 162.领先玩家标记

```c++
//在分数最高的玩家头顶生成一个Niagara
```

## 163.击杀显示

```c++

#include "FirstGameMode.h"
//遍历所有Controller来添加
for (FConstPlayerControllerIterator It=GetWorld()->GetPlayerControllerIterator(); It; ++It)
{
	AFirstPlayerController* FirstPlayerController = Cast<AFirstPlayerController>(*It);
	if (FirstPlayerController)
	{
		FirstPlayerController->BroadcastElimedAnnouncement(AttackerController->GetPlayerState<APlayerState>(), VictimController->GetPlayerState<APlayerState>());
	}
}

#include "FirstPlaeyrController"
void AFirstPlayerController::BroadcastElimedAnnouncement(APlayerState* Attacker, APlayerState* Victim)
{
	ClientElimedAnnouncement(Attacker, Victim);
}
//只在所有人的客户端进行添加
void AFirstPlayerController::ClientElimedAnnouncement_Implementation(APlayerState* Attacker, APlayerState* Victim)
{
	APlayerState* Self = GetPlayerState<APlayerState>();
	if (Self && Attacker&& Victim)
	{
		if (FirstHUD)
		{
            //不同的击杀提示
			if (Victim == Attacker && Self == Victim)
			{
				FirstHUD->AddElimedAnnouncment("你", "你自己");
				return;
			}
			if (Victim == Self && Attacker != Self)
			{
				FirstHUD->AddElimedAnnouncment(Attacker->GetPlayerName(), "你");
				return;
			}
			if (Attacker == Self && Victim != Self)
			{
				FirstHUD->AddElimedAnnouncment("你", Victim->GetPlayerName());
				return;
			}

			FirstHUD->AddElimedAnnouncment(Attacker->GetPlayerName(), Victim->GetPlayerName());
			return;
		}	
	}
}

#include "FirstHUD.h"
void AFirstHUD::AddElimedAnnouncment(FString AttackerName, FString VictimName)
{
	APlayerController* PlayerController = GetOwningPlayerController();
	if (PlayerController && ElimedAnnouncedWigetClass)
	{
		ElimedAnnouncedWidget = CreateWidget<UElimedAnnouncedWidget>(PlayerController, ElimedAnnouncedWigetClass);
		ElimedAnnouncedWidget->ElimAnnouncement(AttackerName, VictimName);
		ElimedAnnouncedWidget->AddToViewport();
	}
}
```



## 164.击杀显示的消失于换行

```c++

#include “FIrstHUD"



#include "Components/HorizontalBox.h"
#include "Components/CanvasPanelSlot.h"
#include "Blueprint/WidgetLayoutLibrary.h"
	for (UElimedAnnouncedWidget* AnnouncedTemp: ElimedAnnouncedArray)
	{	
        //遍历所有的框进行移动
		if (AnnouncedTemp&&AnnouncedTemp->AnnouncedHorizontal)
		{
			UCanvasPanelSlot* CanvasPanel = UWidgetLayoutLibrary::SlotAsCanvasSlot(AnnouncedTemp->AnnouncedHorizontal);
			if (CanvasPanel)
			{
				FVector2D Position = CanvasPanel->GetPosition();
				FVector2D NewPosition = FVector2D(Position.X, Position.Y - CanvasPanel->GetSize().Y);
				CanvasPanel->SetPosition(NewPosition);
			}
		}
	}



	FTimerHandle ElimedAnnouncementHandle;
	FTimerDelegate ElimedAnnouncementDelegate;
	//启动时间调度器 进行消除倒计时
	ElimedAnnouncementDelegate.BindUFunction(this, FName("ElimedAnnouncedTimerFinished"), ElimedAnnouncedWidget);
		GetWorld()->GetTimerManager().SetTimer(
			ElimedAnnouncementHandle,
			ElimedAnnouncementDelegate,
			ElimedAnnouncedTime,
			false
			);


```



## 165.爆头伤害

```c++

//击中点的骨骼名称是否等于head
HitResult.BoneName.ToString()==FString("head");
```

## 166.Team(团队)

```c++
//通过switch设置材质

void AFirstCharacter::SetTeamMaterial(ETeam Team)
{
	if (GetMesh() == nullptr )return;
	switch (Team)
	{
	case ETeam::ET_NoTeam:
		if (BlueTeamDissolveMaterialIns&&NoTeamMaterialIns)
		{
			GetMesh()->SetMaterial(0, NoTeamMaterialIns);
			DissolveMaterialInstance = BlueTeamDissolveMaterialIns;
		}
		break;
	case ETeam::ET_BlueTeam:
		if (BlueTeamDissolveMaterialIns && BlueTeamMaterialIns)
		{
			GetMesh()->SetMaterial(0, BlueTeamMaterialIns);
			DissolveMaterialInstance = BlueTeamDissolveMaterialIns;
		}
		break;
	case ETeam::ET_RedTeam:
		if (RedTeamDissolveMaterialIns && RedTeamMaterialIns)
		{
			GetMesh()->SetMaterial(0, RedTeamMaterialIns);
			DissolveMaterialInstance = RedTeamDissolveMaterialIns;
		}
		break;
	case ETeam::ET_MAX:
		break;
	}
}

```

## 167.团队比分

```c++

```

## 168.主界面菜单优化

```c++

```

