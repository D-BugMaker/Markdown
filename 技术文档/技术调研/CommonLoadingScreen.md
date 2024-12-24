# Loading调研

[TOC]

---

## 1 概览

- **插件名称:**  `CommonLoadingScreen`
- **模块**
  - `CommonLoadingScreen`
  - `CommonStartupLoadingScreen`
- **功能描述:**
> 加载屏幕管理器处理项目指定的加载屏幕UI的创建和显示 .该模块主要用于在游戏加载阶段显示过渡画面或加载动画，提供一个灵活的、可自定义的加载屏幕解决方案。

- **核心功能**
	-  TODO

---

## 2 文件结构

CommonLoadingScreen/
├── Source/
│   ├── CommonLoadingScreen/
│   │   ├── Private/                         # 私有实现文件
│   │   │   ├── CommonLoadingScreenModule.cpp
│   │   │   ├── CommonLoadingScreenSettings.cpp
│   │   │   ├── CommonLoadingScreenSettings.h
│   │   │   ├── LoadingScreenManager.cpp
│   │   ├── Public/                          # 公共头文件
│   │   │   ├── LoadingProcessInterface.h
│   │   │   ├── LoadingProcessTask.cpp
│   │   │   ├── LoadingProcessTask.h
│   │   │   ├── LoadingScreenManager.h
│   │   ├── CommonLoadingScreen.Build.cs     # 构建配置
│   ├── CommonStartupLoadingScreen/          # 启动加载屏幕模块
│   │   ├── Private/
│   │   │   ├── CommonPreLoadScreen.cpp
│   │   │   ├── CommonPreLoadScreen.h
│   │   │   ├── CommonStartupLoadingScreen.cpp
│   │   │   ├── SCommonPreLoadScreenWidget.cpp
│   │   │   ├── SCommonPreLoadScreenWidget.h
│   │   ├── CommonStartupLoadingScreen.Build.cs
├── CommonLoadingScreen.uplugin              # 插件配置文件

---

## 3 CommonStartupLoadingScreen Module

### 3.1  CommonStartupLoadingScreen.Build.cs

#### 3.1.1 源码

``` c#
		PublicDependencyModuleNames.AddRange(
			new string[]
			{
				"Core",
				// ... add other public dependencies that you statically link with here ...
			}
			);
			
		PrivateDependencyModuleNames.AddRange(
			new string[]
			{
				"CoreUObject",
				"Engine",
				"Slate",
				"SlateCore",
				"MoviePlayer",
				"PreLoadScreen",
				"DeveloperSettings"
			}
			);
```
#### 3.1.2 关键依赖模块

1. MoviePlayer
2. PreLoadScreen
3. DeveloperSettings

#### 3.1.3 模块依赖图

```mermaid

graph TD
    CommonStartupLoadingScreen --> MoviePlayer
    CommonStartupLoadingScreen --> PreLoadScreen
    
    MoviePlayer --> Engine
    
    PreLoadScreen --> Engine
    
    Engine --> Slate
    
    Slate --> SlateCore
    
    SlateCore --> DeveloperSettings
    
    DeveloperSettings --> CoreUObject
    
    CoreUObject --> Core

```

### 3.2 核心模块

#### 3.2.1 流程图

```mermaid
graph TD    
    StartupModule[模块启动] --> IsRunningDedicatedServer{是否为DS?}
    IsRunningDedicatedServer --True--> End[模块结束]
    IsRunningDedicatedServer --False--> Init[PreLoadScreen初始化]
    Init --> CheckPreLoadScreenAvailability{检查PreLoadScreen环境有效性?}
    CheckPreLoadScreenAvailability --True--> Register[Mananger注册PreLoadScreen]
    CheckPreLoadScreenAvailability --True--> BindOnPreLoadScreenManagerCleanUp[绑定PreLoadScreen清理Delegate]
    CheckPreLoadScreenAvailability --False--> End
    BindOnPreLoadScreenManagerCleanUp --> Wait[等待OnPreLoadScreenManagerCleanUp触发]
    Wait --> OnPreLoadScreenManagerCleanUp[PreLoadScreen清空]
    OnPreLoadScreenManagerCleanUp --> End
```

#### 3.2.2 时序图

```mermaid

sequenceDiagram
    participant Game as LaunchEngineLoop
    participant CommonPreLoadScreenModule as 通用预加载屏幕系统模块   
    participant CommonPreLoadScreen as 通用预加载屏幕对象
    participant SCommonPreLoadingScreenWidget as 通用预加载屏幕UI
    participant PreLoadScreenManager as 预加载屏幕管理器
    participant X as X
    
    Game->> PreLoadScreenManager:Create
    activate PreLoadScreenManager
    Game->> PreLoadScreenManager:Initialize
    Game->>CommonPreLoadScreenModule:模块启动
    activate CommonPreLoadScreenModule
    CommonPreLoadScreenModule->>CommonPreLoadScreen:初始化预加载对象
    activate CommonPreLoadScreen
    CommonPreLoadScreen->>SCommonPreLoadingScreenWidget:创建PreLoadScreenWidget
    activate SCommonPreLoadingScreenWidget
    CommonPreLoadScreenModule->>PreLoadScreenManager:注册预加载对象
    CommonPreLoadScreenModule->>PreLoadScreenManager:绑定CleanUpDelegate
    Game->> PreLoadScreenManager:PlayFirstPreLoadScreen(CustomSplashScreen)
    PreLoadScreenManager-->>PreLoadScreenManager:HandleCustomSplashScreenPlay
    PreLoadScreenManager-->>PreLoadScreenManager:EarlyPlayFrameTick
    PreLoadScreenManager-->>PreLoadScreenManager:StopPreLoadScreen
    Game->> PreLoadScreenManager:PlayFirstPreLoadScreen(EarlyStartupScreen)
    PreLoadScreenManager-->>PreLoadScreenManager:HandleEarlyStartupPlay
    PreLoadScreenManager-->>PreLoadScreenManager:EarlyPlayFrameTick
    PreLoadScreenManager-->>PreLoadScreenManager:StopPreLoadScreen
    Game->> PreLoadScreenManager:PlayFirstPreLoadScreen(EngineLoadingScreen)
    PreLoadScreenManager-->>PreLoadScreenManager:HandleEngineLoadingPlay
    PreLoadScreenManager-->>PreLoadScreenManager:WaitForEngineLoadingScreenToFinish
    Game->> PreLoadScreenManager:Destroy
    PreLoadScreenManager-->>CommonPreLoadScreenModule:OnPreLoadScreenManagerCleanUp
    CommonPreLoadScreenModule->>CommonPreLoadScreenModule:ShutdownModule
    
    CommonPreLoadScreenModule->>CommonPreLoadScreen:Reset
    CommonPreLoadScreen->>SCommonPreLoadingScreenWidget:Destroy
    
    
    
    
    
    
    SCommonPreLoadingScreenWidget-->>X:对象销毁
    deactivate SCommonPreLoadingScreenWidget
    CommonPreLoadScreen-->>X:对象销毁
    deactivate CommonPreLoadScreen
    CommonPreLoadScreenModule -->>X:对象销毁
    deactivate CommonPreLoadScreenModule
    PreLoadScreenManager-->>X:对象销毁
    deactivate PreLoadScreenManager

```

---

## 4 CommonLoadingScreen Module

### 4.1 CommonLoadingScreen.Build.cs

#### 4.1.1 源码

```c#
		PublicDependencyModuleNames.AddRange(
			new string[]
			{
				"Core",
				// ... add other public dependencies that you statically link with here ...
			}
			);
			
		
		PrivateDependencyModuleNames.AddRange(
			new string[]
			{
				"CoreUObject",
				"Engine",
				"Slate",
				"SlateCore",
				"InputCore",
				"PreLoadScreen",
				"RenderCore",
				"DeveloperSettings",
				"UMG"
			}
			);
```

#### 4.1.2 关键依赖模块
1. InputCore
2. PreLoadScreen
3. DeveloperSettings
4. UMG

#### 4.1.3 模块依赖图

```mermaid

graph TD
    CommonLoadingScreen --> PreLoadScreen
    CommonLoadingScreen --> RenderCore
    CommonLoadingScreen --> UMG
    
    PreLoadScreen --> Engine
    
    UMG --> Engine
    
    Engine --> Slate
    
    Slate --> SlateCore
    
    SlateCore --> InputCore
    SlateCore --> DeveloperSettings
    
    InputCore --> CoreUObject
     
    DeveloperSettings --> CoreUObject
    
    RenderCore --> CoreUObject
    
    CoreUObject --> Core
```

### 4.2 核心模块

#### 4.2.1 类图

```mermaid
classDiagram
    class FTickableGameObject{
    	+ void Tick(float DeltaTime)
    	+ ETickableTickType GetTickableTickType() const
    	+ bool IsTickable() const
    	+ TStatId GetStatId() const
    	+ UWorld* GetTickableGameObjectWorld() const
    }
    
    class UGameInstanceSubsystem{
    	+ bool ShouldCreateSubSystem(UObejct* Outer) const
    	+ void Initialize(FSubsystemCollectionBase& Collection)
    	+ void Deinitalize()
    	- FSubsystemCollectionBase* InternalOwningSubsystem
    	+ UGameInstance* GetGameInstance()
    }
    
    class ULoadingScreenManager{
    	+ FString GetDebugReasonForShowingOrHidingLoadingScreen() const
    	+ bool GetLoadingScreenDisplayStatus() const
    	+ bool FOnLoadingScreenVisibilityChangeDelegate()  Delegate
    	+ void RegisterLoadingProcessor(TScriptInterface<ILoadingProcessInterface> Interface)
    	+ void UnregisterLoadingProcessor(TScriptInterface<ILoadingProcessInterface> Interface)
    	
    	- void HandlePreLoadMap(const FWorldContext& WorldContext, const FString& MapName)
    	- void HandlePostLoadMap(UWorld* World)
    	- void UpdateLoadingScreen()
    	- bool CheckForAnyNeedToShowLoadingScreen()
    	- bool ShouldShowLoadingScreen()
    	- bool IsShowingInitialLoadingScreen() const
    	- void ShowLoadingScreen()
    	- void HideLoadingScreen()
    	- void RemoveWidgetFromViewport()
    	- void StartBlockingInput()
    	- void StopBlockingInput()
    	- void ChangePerformanceSettings(bool bEnabingLoadingScreen)
    	
    	- FOnLoadingScreenVisibilityChangedDelegate LoadingScreenVisibilityChanged
    	- TSharedPtr<SWidget> LoadingScreenWidget
    	- TSharedPtr<IInputProcessor> InputPreProcessor
    	- TArray<TWeakInterfacePtr<ILoadingProcessInterface>> ExternalLoadingProcessors
    	- FString DebugReasonForShowingOrHidingLoadingScreen
    	- double TimeLoadingScreenShown = 0.0
    	- double TimeLoadingScreenLastDismissed = -1.0
    	- double TimeUntilNextLogHeartbeatSeconds = 0.0
    	- bool bCurrentlyInLoadMap = false
    	- bool bCurrentlyShowingLoadingScreen = false
    }
    
    class IInputProcessor {
    	+ void Tick()
    	+ bool HandleKeyDownEvent(FSlateApplication& SlateApp, const FKeyEvent& InKeyEvent)
    	+ bool HandleKeyUpEvent(FSlateApplication& SlateApp, const FKeyEvent& InKeyEvent)
    	+ bool HandleAnalogInputEvent(FSlateApplication& SlateApp, const FAnalogInputEvent& InAnalogInputEvent)
    	+ bool HandleMouseMoveEvent(FSlateApplication& SlateApp, const FPointerEvent& MouseEvent)
    	+ bool HandleMouseButtonDownEvent(FSlateApplication& SlateApp, const FPointerEvent& MouseEvent)
    	+ bool HandleMouseButtonUpEvent(FSlateApplication& SlateApp, const FPointerEvent& MouseEvent)
    	+ bool HandleMouseButtonDoubleClickEvent(FSlateApplication& SlateApp, const FPointerEvent& MouseEvent)
    	+ bool HandleMouseWheelOrGestureEvent(FSlateApplication& SlateApp, const FPointerEvent& InWheelEvent, const FPointerEvent* InGestureEvent)
    	+ bool HandleMotionDetectedEvent(FSlateApplication& SlateApp, const FMotionEvent& MotionEvent)
    }
    
    class FLoadingScreenInputPreProcessor{
    	+ bool CanEatInput()	
    }
    
    class ILoadingProcessInterface{
    	+ static bool ShouldShowLoadingScreen(UObject* TestObject, FString& OutReason)
    	+ bool ShouldShowLoadingScreen(FString& OutReason) const
    }
    
    class ULoadingProcessTask{
    	+ static ULoadingProcessTask* CreateLoadingScreenProcessTask(UObject* WorldContextObject, const FString& ShowLoadingScreenReason)
    	+ void Unregister()
    	+ void SetShowLoadingScreenReason(const FString& InReason)
    	+ FString Reason
    }
    
    class UObject {
    
    }
    
    class UDeveloperSettings{
    	# void ImportConsoleVariableValues();
    	# void ExportValuesToConsoleVariables();
    }
    
    class UDeveloperSettingsBackedByCVars{
    	+ void PostInitProperties();
    	+ void PostEditChangeProperty();
    }
    
    class UCommonLoadingScreenSettings{
    	+ FSoftClassPath LoadingScreenWidget
    	+ int32 LoadingScreenZOrder = 10000
    	+ float HoldLoadingScreenAdditionalSecs = 2.0f
    	+ float LoadingScreenHeartbeatHangDuration = 0.0f
    	+ float LogLoadingScreenHeartbeatInterval = 5.0f
    	+ bool LogLoadingScreenReasonEveryFrame = 0
    	+ bool ForceLoadingScreenVisible = false
    	+ bool HoldLoadingScreenAdditionalSecsEvenInEditor = false
    	+ bool ForceTickLoadingScreenEvenInEditor = true
    }
    

    
    UGameInstanceSubsystem <|-- ULoadingScreenManager:IsA
    IInputProcessor <|-- FLoadingScreenInputPreProcessor:IsA
    UObject <|-- ULoadingProcessTask:IsA
    UObject <|-- UDeveloperSettings:IsA
    UDeveloperSettings <|-- UDeveloperSettingsBackedByCVars:IsA
    UDeveloperSettingsBackedByCVars <|-- UCommonLoadingScreenSettings:IsA
    
    FTickableGameObject <|.. ULoadingScreenManager:Implements
    ILoadingProcessInterface <|.. ULoadingProcessTask:Implements
    
    ULoadingScreenManager ..> FLoadingScreenInputPreProcessor : uses
    ULoadingScreenManager ..> UCommonLoadingScreenSettings : uses
    ULoadingProcessTask ..> ULoadingScreenManager : uses
    
    ULoadingScreenManager *-- ILoadingProcessInterface:contains


    
```

#### 4.2.2 数据图

```mermaid
 erDiagram
    %% 定义实体
    UCommonLoadingScreenSettings {
        string LoadingScreenWidget
        int LoadingScreenZOrder
        float HoldLoadingScreenAdditionalSecs
        float LoadingScreenHeartbeatHangDuration
        float LogLoadingScreenHeartbeatInterval
        bool LogLoadingScreenReasonEveryFrame
        bool ForceLoadingScreenVisible
        bool HoldLoadingScreenAdditionalSecsEvenInEditor
        bool ForceTickLoadingScreenEvenInEditor
    }

    ULoadingProcessTask {
        string Reason
    }

    ILoadingProcessInterface {
 	  string Reason
    }

    ULoadingScreenManager {
        string DebugReasonForShowingOrHidingLoadingScreen
        bool bCurrentlyInLoadMap
        bool bCurrentlyShowingLoadingScreen
        double TimeLoadingScreenShown
        double TimeLoadingScreenLastDismissed
        double TimeUntilNextLogHeartbeatSeconds
    }

    FLoadingScreenInputPreProcessor {
        bool CanEatInput()
    }

    %% 定义关系
    UCommonLoadingScreenSettings ||--|| ULoadingScreenManager : "配置数据"
    ULoadingProcessTask ||--o| ULoadingScreenManager : "动态任务数据"
    ILoadingProcessInterface ||--o| ULoadingScreenManager : "外部处理器"
    FLoadingScreenInputPreProcessor ||--|| ULoadingScreenManager : "输入预处理器"

```

```mermaid
graph TD
    subgraph StaticData
        A[UCommonLoadingScreenSettings]
    end
    subgraph DynamicData
        B[ULoadingProcessTask]
        C[ILoadingProcessInterface]
    end
    subgraph CoreManager
        D[ULoadingScreenManager]
    end
    subgraph UI
        E[LoadingScreenWidget]
    end

    %% 数据流关系
    A -->|Provide Config| D
    B -->|Pass Task Data| D
    C -->|External Processors| D
    D -->|Update UI| E
```

#### 4.2.3 流程图

```mermaid
graph TD
    Start[启动] --> Initialize
    Initialize --> PreLoadMapWithContext[加载Map前事件]
    Initialize --> Tick
    Initialize --> PostLoadMapWithWorld[加载Map后事件]
    
    
    PreLoadMapWithContext -->  UpdateLoadingScreen[更新加载界面]
    UpdateLoadingScreen --> ShouldShowLoadingScreen{是否显示加载界面?}
    ShouldShowLoadingScreen --TRUE--> ShowLoadingScreen[显示加载界面]
    ShouldShowLoadingScreen --FASLE--> HideLoadingScreen[隐藏加载界面]
    
    PostLoadMapWithWorld --> SetCurrentlyInLoadMapFalse[设置当前不处于地图加载中]
    
    Tick --> UpdateLoadingScreen
    
    ShowLoadingScreen --> StartBlockingInput[开始阻塞输入]
    ShowLoadingScreen --> ChangePerformanceSettings[改变性能设置]
    
    HideLoadingScreen --> ChangePerformanceSettings
    HideLoadingScreen --> StopBlockingInput[停止阻塞输入]
    
    ChangePerformanceSettings --> IsEnabingLoadingScreen{是否开启加载界面?}
    
    IsEnabingLoadingScreen --TRUE--> ShaderPipelineCacheBatchModeFast[设置着色器Pipeline缓存批处理为Fast]
    ShaderPipelineCacheBatchModeFast --> DisableWorldRendering[禁用世界渲染]
    DisableWorldRendering --> HighPriorityLoadingLocal[高优先级加载当前World]
    HighPriorityLoadingLocal --> SetDurationMultiplier[更新阻塞挂起时间乘数]
    SetDurationMultiplier --> SuspendHeartBeat[暂停阻塞心跳检测]
    
    
    IsEnabingLoadingScreen --FASLE-->ShaderPipelineCacheBatchModeBackground[设置着色器Pipeline缓存批处理为Background]
    ShaderPipelineCacheBatchModeBackground --> EnableWorldRendering[开启大世界渲染]
    EnableWorldRendering --> SetDurationMultiplier1.0[恢复阻塞挂起时间乘数]
    SetDurationMultiplier1.0 -->ResumeHeartBeat[恢复阻塞心跳检测]
    
    Start --> End
    End[结束] --> Deinitialize
```

#### 4.2.4 时序图

```mermaid
sequenceDiagram
    participant UGameInstanceSubsystem as GameInstance
    participant ULoadingScreenManager as LoadingScreenManager
    participant LoadingProcess as LoadingProcess
    participant InputPreProcessor as InputPreProcessor
    participant CommonLoadingScreenSettings as CommonLoadingScreenSettings
    participant LoadingScreenUI as LoadingScreenUI
    
    
    UGameInstanceSubsystem ->> ULoadingScreenManager: Initialize加载界面管理器
    activate ULoadingScreenManager
    ULoadingScreenManager ->> ULoadingScreenManager: Tick
    ULoadingScreenManager ->> ULoadingScreenManager: UpdateLoadingScreen
    ULoadingScreenManager ->> ULoadingScreenManager: ShouldShowLoadingScreen
    ULoadingScreenManager ->> ULoadingScreenManager: CheckForAnyNeedToShowLoadingScreen

    ULoadingScreenManager ->> LoadingProcess:ShouldShowLoadingScreen
    LoadingProcess -->> ULoadingScreenManager:DebugReasonForShowingOrHidingLoadingScreen
    
    ULoadingScreenManager ->> ULoadingScreenManager: ShowLoadingScreen
    ULoadingScreenManager ->> InputPreProcessor : StartBlockingInput
    ULoadingScreenManager ->> CommonLoadingScreenSettings: GetLoadingScreenWidgetClass
    CommonLoadingScreenSettings -->> ULoadingScreenManager: LoadingScreenWidget
    ULoadingScreenManager --> ULoadingScreenManager:ChangePerformanceSettings
    
    ULoadingScreenManager ->> ULoadingScreenManager: HideLoadingScreen
    ULoadingScreenManager ->> InputPreProcessor : StopBlockingInput
    ULoadingScreenManager ->> LoadingScreenUI: RemoveWidgetFromViewport
    ULoadingScreenManager --> ULoadingScreenManager:ChangePerformanceSettings

    
    UGameInstanceSubsystem ->> ULoadingScreenManager: Deinitialize加载界面管理器
    
    deactivate ULoadingScreenManager
```
