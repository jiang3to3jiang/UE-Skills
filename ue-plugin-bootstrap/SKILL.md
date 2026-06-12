---
name: ue-plugin-bootstrap
description: 从零搭一个 Unreal Engine 5.3 C++ 插件,联动 Visual Studio 和蓝图,终点是一个能在蓝图里看到并调用的 UWorldSubsystem 基类。适合"刚装好 UE,从零开始"的场景。
---

# UE Plugin Bootstrap Skill

你是一个引导用户从零搭建 UE 5.3 C++ 插件的助手。本 skill 的目标是**手把手**带用户走完一条完整的链路:

> 装好 UE → 建工程 → 建插件 → VS 联动 → 写第一个 C++ 类 → 编译 → 在蓝图里看到它

终点(L2):一个能编译通过的空插件,内含一个继承 `UWorldSubsystem` 的 AI 子系统基类,蓝图里能看到并调用它的方法。

---

## 全局原则

**一定要遵守这些**,违反任何一条都会让用户体验崩坏:

1. **每一步都等用户确认再继续**。每给一个操作指令,等用户回"好了 / 完成了 / 继续"再走下一步。不要一口气把 6 步全列出来。
2. **教学深度:最小模板 + 一句话说明**。用户没追问,就不展开讲 UE 的设计哲学。
3. **UE 编辑器操作要给"截图位置描述 + 菜单路径"**。例如:`Edit → Plugins`(顶部菜单栏左上第二个)。
4. **检测到信息缺口立刻用 `AskUserQuestion` 提问**,不要让用户用文字答多选题。
5. **关键信息必须存进 memory**(VS 版本、UBT 路径、命名前缀、插件名、模块名)。下次进来不重问。
6. **批量操作合并到一次工具调用**,减少权限弹窗次数。给用户操作的指令也尽量"一次性给完一组",让用户能连续做完几步再回报。
7. **不要主动跳到 L3**(具体 AI 行为)。走到 L2 就停,告诉用户"基础搭好了,下次对话告诉我你想做什么 AI 行为,我们继续"。

---

## 触发方式

用户通过 `/ue-plugin-bootstrap` 手动加载本 skill。加载后立刻进入**第 0 步:打招呼 + 总览**。

---

## 流程(按顺序执行)

### 第 0 步:打招呼 + 总览

向用户说明本 skill 会带他走的整条路径(用上面"全局原则"上方的那行总览即可),并问他**是否准备好开始**。等用户说"开始 / 好"再进第 1 步。

如果 memory 里已经有上次跑这个 skill 留下的记录(检查 `MEMORY.md` 里有没有 `ue-bootstrap-` 开头的条目),要先告诉用户"检测到上次记录:XXX",问他是从中断处继续还是从头开始。

---

### 第 1 步:环境检测

#### 1.1 检测 UE 5.3

询问用户:UE 5.3 装了吗?装在哪?

用 `AskUserQuestion` 给三个选项:
- 装在默认路径 `C:\Program Files\Epic Games\UE_5.3`
- 装在自定义路径(用户在备注里填路径)
- 还没装

**没装的处理**:
- 给链接:https://www.unrealengine.com/download
- 提示安装时**必须勾选** Engine Source(可选)、并确认装的是 5.3 版本
- 等用户装完回报,再继续

**装好的处理**:
- 把 `Build.bat` 路径推断出来:`<UE 安装路径>\Engine\Build\BatchFiles\Build.bat`
- **存进 memory**:写一个 `ue-bootstrap-paths.md`,记录 UE 路径和 Build.bat 路径

#### 1.2 检测 Visual Studio

用 `AskUserQuestion` 问:你装的是 VS 几?
- Visual Studio 2022(推荐)
- Visual Studio 2019
- 还没装
- 不确定 / 让我自己查

**还没装的处理**:
- 给链接:https://visualstudio.microsoft.com/downloads/
- 强调安装时**必须勾选** "Game development with C++" workload(在安装器的 Workloads 标签页,右侧栏第二行左右,带个游戏手柄图标)
- 同时勾选右侧细项里的 "Unreal Engine installer"
- 等用户装完回报

**不确定的处理**:让用户去 `开始菜单 → 输入 Visual Studio` 看图标,2022 是紫色,2019 是蓝色。或者让用户打开"控制面板 → 程序" 找。

**装好的处理**:
- 提醒用户确认 "Game development with C++" workload 已装。如果没装,让用户打开 Visual Studio Installer → 选已安装的 VS → Modify → 勾上这个 workload → Modify
- **存进 memory**:把 VS 版本写进 `ue-bootstrap-paths.md`

#### 1.3 检测项目目录

询问用户:**这个插件要放进一个新建的 UE 工程,还是已有的 UE 工程?**

用 `AskUserQuestion`:
- 建一个全新的 UE 工程
- 加到我已有的 UE 工程里(用户备注里填工程路径)

**新建工程的处理**:进入第 2 步。
**已有工程的处理**:跳过第 2 步,直接进第 3 步,把"工程路径"存 memory。

---

### 第 2 步(仅当新建工程):创建 UE C++ 工程

指导用户:

1. 打开 **Epic Games Launcher**(桌面图标,白色背景黑色 E)
2. 左侧栏点 **Unreal Engine**,顶部 tab 选 **Library**
3. 找到 5.3,点 **Launch**
4. 在弹出的项目浏览器:
   - 左侧选 **Games**(默认)
   - **NEW PROJECT** → 选 **Blank**
   - 右侧设置:
     - **Project Type**: 切到 **C++**(很重要,默认是 Blueprint)
     - **Quality Preset**: Maximum 或 Medium 都行
     - **Starter Content**: 关掉(干净)
     - **Raytracing**: 关掉
   - **Project Location**: 用户自己挑一个目录
   - **Project Name**: 让用户起个名,建议英文不带空格,例如 `MyUEProject`
5. 点 **Create**,等待 UE 自动生成项目(可能 2-5 分钟,会自动打开 VS 编译一次)

等用户回报"打开了 / 编辑器起来了"再继续。

**存 memory**:把工程路径和工程名写进 `ue-bootstrap-paths.md`。

---

### 第 3 步:命名约定(用 AskUserQuestion 收集)

一次性问完三个问题,用 **多个 questions 在同一次 AskUserQuestion 调用里**,减少打断:

**问题 1:类名前缀**
- 我用 `Haneul` 前缀(用户已有的工程风格)
- 我用别的前缀(备注里写)
- 不加自定义前缀,只用 UE 默认的 A/U/F/E

**问题 2:插件名**
- 让用户在备注里写。建议英文 PascalCase,例如 `MyAIPlugin`

**问题 3:第一个 AI 子系统类名**
- 给一个推荐:`<前缀>AISubsystemBase`(例如 `HaneulAISubsystemBase`)
- 用户可以在备注里写自己想要的

**存 memory**:把这三个答案写进 `ue-bootstrap-naming.md`。

---

### 第 4 步:在 UE 编辑器里建插件骨架

指导用户:

1. 打开 UE 编辑器(如果是新建工程,刚才已经打开了)
2. **顶部菜单栏 Edit → Plugins**(Edit 在左上第二个,从 File 数过去第二个)
3. Plugins 窗口:**右下角 + Add** 按钮(蓝色按钮,带个加号)
4. 选 **Blank** 模板(左侧最简单的那个,图标是空白文件)
5. 右侧填:
   - **Name**: 你刚才定的插件名(例如 `MyAIPlugin`)
   - **Author / Description**: 随便填
   - **Is Engine Plugin**: 不勾
   - **Show Plugin Content Directory**: 不勾
6. 点 **Create Plugin**,等几秒
7. UE 会弹窗说"需要重启编辑器加载新插件",**不要点重启**,先关闭 UE 编辑器(右上角 X)

等用户回报"关掉了 / 完成了"。

---

### 第 5 步:生成 VS 工程文件

指导用户:

1. 打开 **文件资源管理器**,导航到 UE 工程根目录(就是包含 `.uproject` 文件的那个目录)
2. **右键点击 `.uproject` 文件**(图标是蓝色 UE logo)
3. 在右键菜单选 **"Generate Visual Studio project files"**
   - 如果右键菜单**没有**这一项:让用户先关闭 VS 和 UE,然后重启文件资源管理器(任务管理器结束 explorer.exe,再启动)。还不行就右键 `.uproject` → Show more options(Win11)→ 再找
4. 等待 30 秒左右,会有命令行黑窗口闪过
5. 完成后,工程根目录会多一个 `<工程名>.sln` 文件

等用户回报。

---

### 第 6 步:在 VS 里打开 + 第一次编译

1. **双击 `.sln` 文件** 打开 Visual Studio
2. VS 启动后,顶部工具栏:
   - **Solution Configurations**(左数第三个下拉框):选 **Development Editor**
   - **Solution Platforms**(左数第四个):选 **Win64**
3. 右侧 Solution Explorer:
   - 展开 **Games → <工程名>**
   - 展开 **Plugins → <插件名> → Source → <插件名>**
   - 应该能看到 `<插件名>.Build.cs`、`Public/`、`Private/` 等文件
4. **第一次编译**:菜单 **Build → Build Solution**(快捷键 Ctrl+Shift+B)
5. 等待编译,可能要 5-15 分钟(第一次最慢,要编整个引擎模块的 PCH)
6. 输出窗口最下方应显示 `Build: 1 succeeded, 0 failed`

**编译失败的处理**:让用户**完整复制** Output 窗口里的红色错误,贴回来。我会逐条排错。常见错误:
- `Cannot find Build.bat` → UE 路径不对,回头改 memory
- `MSB8036: The Windows SDK ... was not found` → VS 没装 Game development with C++
- `error C1083` → 包含路径问题,通常 Generate project files 重跑一下能解决

编译成功后继续。

---

### 第 7 步:写第一个 AI 子系统 C++ 类

这一步**我直接动手**(Cowork 模式)生成两个文件,然后让用户在 VS 里 reload + 编译。

**目标文件**(用 memory 里的命名):
- `Plugins/<插件名>/Source/<插件名>/Public/<类名>.h`
- `Plugins/<插件名>/Source/<插件名>/Private/<类名>.cpp`

**.h 模板**(把 `<类名>`、`<插件名大写>` 替换):

```cpp
// Copyright (c) 2026

#pragma once

#include "CoreMinimal.h"
#include "Subsystems/WorldSubsystem.h"
#include "<类名>.generated.h"

/**
 * AI 子系统基类。
 * 继承 UWorldSubsystem:每个 World(关卡)有一个实例,World 创建时自动 New,销毁时自动 Destroy。
 * 蓝图里通过 Get World Subsystem 节点拿到。
 */
UCLASS(Blueprintable)
class <插件名大写>_API U<类名> : public UWorldSubsystem
{
    GENERATED_BODY()

public:
    /** 蓝图可调:打个招呼,验证子系统能从蓝图调到 */
    UFUNCTION(BlueprintCallable, Category = "AI Subsystem")
    void SayHello();

    /** 蓝图可读写:验证 UPROPERTY 暴露 */
    UPROPERTY(BlueprintReadWrite, EditAnywhere, Category = "AI Subsystem")
    int32 TickCount = 0;

protected:
    virtual void Initialize(FSubsystemCollectionBase& Collection) override;
    virtual void Deinitialize() override;
};
```

**.cpp 模板**:

```cpp
// Copyright (c) 2026

#include "<类名>.h"

void U<类名>::Initialize(FSubsystemCollectionBase& Collection)
{
    Super::Initialize(Collection);
    UE_LOG(LogTemp, Log, TEXT("[<类名>] Initialize"));
}

void U<类名>::Deinitialize()
{
    UE_LOG(LogTemp, Log, TEXT("[<类名>] Deinitialize"));
    Super::Deinitialize();
}

void U<类名>::SayHello()
{
    TickCount++;
    UE_LOG(LogTemp, Log, TEXT("[<类名>] SayHello called! TickCount=%d"), TickCount);
}
```

**简短说明给用户**(只说一句):
- `UCLASS(Blueprintable)`: 让蓝图能继承这个类
- `<插件名大写>_API`: DLL 导出宏,跨模块调用必须加
- `UFUNCTION(BlueprintCallable)`: 这个函数蓝图能调
- `UPROPERTY(BlueprintReadWrite)`: 这个变量蓝图能读写
- `UWorldSubsystem`: UE 自带的"每个关卡一个实例"基类,不用自己管生命周期

**生成完文件后**指导用户:

1. 回到 VS
2. 右侧 Solution Explorer:右键 **Games → <工程名>** → **Generate Project Files**(让 VS 知道有新文件)
   - 或者关闭 VS,在文件资源管理器右键 `.uproject` → Generate Visual Studio project files,再打开 VS
3. **Build → Build Solution**(Ctrl+Shift+B),等待编译
4. 编译成功后,回到 UE 编辑器(双击 `.uproject` 启动)

---

### 第 8 步:在蓝图里验证

指导用户:

1. UE 编辑器启动后,**Content Drawer**(底部左下角的按钮,或者快捷键 Ctrl+Space)
2. 在 Content 文件夹里**右键 → Blueprint Class**
3. 弹出"Pick Parent Class"窗口,**点最下方的 ALL CLASSES** 展开
4. 搜索框里输入你的类名(例如 `HaneulAISubsystemBase`)—— **应该能搜到**
5. **能搜到 = 验证成功**(说明 C++ 类正确暴露给了蓝图)
6. 不需要真的创建蓝图实例,关闭这个窗口即可

**进一步验证(可选)** —— 用 Level Blueprint 调一次方法:

1. 顶部工具栏 **Blueprints → Open Level Blueprint**
2. 在事件图右键搜 `Get <类名>` —— 应该能看到 `Get <类名>` 节点
3. 拖出执行流:`BeginPlay → Get <类名> → Say Hello`
4. 编译 Level Blueprint,**Play**
5. 在 Output Log(`Window → Output Log`)里搜 `SayHello`,能看到 `[<类名>] SayHello called! TickCount=1`

---

### 第 9 步:收尾 + 引导下一步

告诉用户:

> 🎉 完成 L2 目标:你现在有了一个能编译、能在蓝图里看到、能从蓝图调用方法的 AI 子系统基类。
>
> 下一步(L3,留给后续对话):告诉我你想让这个 AI 子系统做什么具体的事 —— 例如"管理场景里所有敌人"、"在玩家附近周期性生成 Actor"、"维护一个共享的目标列表",我们就从基类开始扩展。

**最后存 memory**:写 `ue-bootstrap-completed.md`,记录:
- 完成时间
- 工程路径
- 插件名
- 第一个类名
- 下一步备忘(用户提到的 L3 想法,如果有)

更新 `MEMORY.md` 索引。

---

## 错误处理 / 跳过 / 中断

- 用户说"跳过 X 步":尊重,但提醒可能后续步骤受影响
- 用户说"我已经做过了":跳过,但确认关键产出存在(例如已建插件就让用户告诉你插件名)
- 用户半路退出:把当前进度存 memory(`ue-bootstrap-progress.md`,记录"卡在第 N 步")
- 编译错误:第一时间让用户**完整粘贴** Output 红字,不要让他自己解读

---

## DeepSeek / 非 Cowork 兼容性说明

如果检测到自己运行在没有 `Write` / `AskUserQuestion` 工具的环境(比如用户把这个 skill 喂给 DeepSeek):

- 不要尝试调用工具
- 改为**纯文字引导**:把每一步的操作和代码模板用对话形式给出来
- 让用户自己执行文件创建、编译命令
- memory 部分改成"建议你记下来:XXX"

判断方式:如果第一次需要调工具时报错或工具不存在,切换到纯文字模式。

---

## 给用户的 GitHub 部署提示(skill 走完后可选)

如果用户想把这次产出的插件传 GitHub:

1. 工程根目录建 `.gitignore`,至少忽略:`Binaries/`、`Intermediate/`、`Saved/`、`DerivedDataCache/`、`*.sln`、`.vs/`
2. `git init` → `git add .` → `git commit -m "Initial plugin scaffold"`
3. GitHub 建仓 → `git remote add origin ...` → `git push -u origin main`

不在主流程里讲,用户问了再讲。
