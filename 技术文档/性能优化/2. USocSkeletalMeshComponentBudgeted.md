# USocSkeletalMeshComponentBudgeted

## 注意事项
**Gameplay中数量不可控的SkeletalMesh都应该使用此类**

> 正向案例
> 1. 玩家的SkeletalMesh
>
> 2. 怪物的SkeletalMesh
>
> 3. 武器的SkeletalMesh
>
> 4. 坐骑的SkeletalMesh

> 反向案例
> 1. Sequence中使用的SkeletalMesh
>
> 2. Showcase中使用的SkeletalMesh

- [ ] 建筑
- [ ] 宝箱

## 功能描述
根据项目的情况设置了一些优化性的参数

## 构造初始化
```c++
USocSkeletalMeshComponentBudgeted::USocSkeletalMeshComponentBudgeted(const FObjectInitializer& ObjectInitializer)
	: Super(ObjectInitializer)
{
	KinematicBonesUpdateType = EKinematicBonesUpdateToPhysics::SkipAllBones;
	VisibilityBasedAnimTickOption = EVisibilityBasedAnimTickOption::OnlyTickMontagesWhenNotRendered;
	SetTickGroup(TG_PrePhysics);
	bSkipBoundsUpdateWhenInterpolating = true;
	bComponentUseFixedSkelBounds = true;

}
```

## 设置子Mesh
```c++
void USocSkeletalMeshComponentBudgeted::SetupAsChildMesh()
{
    // 获取父级组件并判断是否是 SkeletalMeshComponent 
    USceneComponent* ParentComponent = GetAttachParent();
    if (ParentComponent && ParentComponent->IsA(USkeletalMeshComponent::StaticClass()))
    {
       UE_LOG(LogSocDebug, Log, TEXT("ParentComponent Component is %s, IsA = %s"), *ParentComponent->GetName(), (ParentComponent->IsA(ThisClass::StaticClass()) ? TEXT("TRUE") :TEXT("FALSE")));
       // 设置子Mesh的TickGroup
       SetTickGroup(TG_DuringPhysics);
       // 设置子Mesh的Bounds
       bUseBoundsFromLeaderPoseComponent = true;
       bIgnoreLeaderPoseComponentLOD = false;
       bUseAttachParentBound = true;
    }
}
```

## 子Mesh参数更新时机

``` c++
	virtual void OnRegister() override;
	virtual void PostEditChangeProperty(struct FPropertyChangedEvent& PropertyChangedEvent) override;
	virtual void PostEditChangeChainProperty(struct FPropertyChangedChainEvent& PropertyChangedEvent) override;
	virtual void PostEditComponentMove(bool bFinished) override;
```

## 参数

###  KinematicBonesUpdateType 
> EKinematicBonesUpdateToPhysics::SkipAllBones 
> Movement更新时跳过骨骼的物理更新

### VisibilityBasedAnimTickOption

> 	/**
> 		When rendered Tick Pose and Refresh Bone Transforms,
> 		otherwise, just update montages and skip everything else.
> 		(AnimBP graph will not be updated).
> 	*/

### TickGroup
1. 默认的Mesh(主Mesh) 使用 **TG_PrePhysics** Tick组
2. 子Mesh使用 **TG_DuringPhysics** Tick组

### bSkipBoundsUpdateWhenInterpolating
> 		/** Whether to skip bounds update when interpolating. Bounds are updated to the target interpolation pose only on ticks when they are evaluated. */

### bComponentUseFixedSkelBounds
> 		/** When true, skip using the physics asset etc. and always use the fixed bounds defined in the SkeletalMesh. */

### bUseBoundsFromLeaderPoseComponent
>	/** 
>	 * When true, we will just using the bounds from our LeaderPoseComponent.  This is useful for when we have a Mesh Parented
>	 * to the main SkelMesh (e.g. outline mesh or a full body overdraw effect that is toggled) that is always going to be the same
>	 * bounds as parent.  We want to do no calculations in that case.
>	 */

### bIgnoreLeaderPoseComponentLOD
>	/** Flag that when set will ensure UpdateLODStatus will not take the > LeaderPoseComponent's current LOD in consideration when determining the correct LOD level (this requires LeaderPoseComponent's LOD to always be >= determined LOD otherwise bone transforms could be missing */


### bUseAttachParentBound
>	/** If true, this component uses its parents bounds when attached.
>	 *  This can be a significant optimization with many components attached together.
>	 */