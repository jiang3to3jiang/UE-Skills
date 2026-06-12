# ue-plugin-bootstrap

一个 Cowork / Claude skill,用来**手把手**带你从零搭一个 Unreal Engine C++ 插件,联动 Visual Studio 和蓝图。**支持 UE 5.0 / 5.1 / 5.2 / 5.3 / 5.4 / 5.5**(skill 第一步会询问你的版本并自动适配)。

## 这个 skill 解决什么问题

第一次写 UE C++ 插件时,要踩的坑包括:
- UE 装了但 VS 没装对 workload(没装 "Game development with C++")
- 工程建错了类型(C++ 项目 vs 蓝图项目)
- `.uproject` 右键找不到 "Generate Visual Studio project files"
- 写完 C++ 类不知道怎么在蓝图里看到它
- UCLASS/UPROPERTY/UFUNCTION 的最小可用写法

这个 skill 把整条路径压缩成一个引导式对话。

## 终点(L2)

走完后你拥有:
- 一个能编译通过的 UE C++ 工程
- 一个空的 C++ 插件
- 一个继承 `UWorldSubsystem` 的 AI 子系统基类
- 在蓝图 Class Picker 里能搜到这个类
- 在 Level Blueprint 里能调用它的方法,Output Log 看到打印

更深入的 AI 行为(具体的 AI 子系统逻辑,L3)留给后续对话扩展。

## 在 Cowork 里怎么用

把整个 `ue-plugin-bootstrap-skill` 目录放进你的 Cowork skills 目录下,然后在 Cowork 里:

```
/ue-plugin-bootstrap
```

Cowork 会加载 `SKILL.md`,从第 0 步开始引导你。

## 在 AionUI / DeepSeek 里怎么用

DeepSeek 没有 Cowork 的工具调用能力(无法 `Write` 文件、`AskUserQuestion`),但 `SKILL.md` 本身就是一段引导 prompt:

1. 把 `SKILL.md` 的全文复制进 AionUI 的对话
2. 让 DeepSeek 当作"系统指令"执行
3. DeepSeek 会切换到**纯文字模式**:用对话形式给你每步指令,你自己手动建文件、跑命令

效果略弱于 Cowork 模式(没法自动建文件),但流程引导一样可用。

## 目录结构

```
ue-plugin-bootstrap-skill/
├── SKILL.md                    # 主流程,Cowork 加载这个
├── README.md                   # 本文件
└── templates/                  # 代码模板,SKILL.md 第 7 步会用
    ├── AISubsystemBase.h.template
    ├── AISubsystemBase.cpp.template
    └── gitignore.template
```

## 适用条件

- Unreal Engine **5.0 ~ 5.5**(已覆盖主流 5.x 版本,skill 内部按你选的版本适配)
- Windows 10 / 11
- Visual Studio 2019 或 2022
  - UE 5.4 / 5.5:**强烈推荐 VS 2022**(VS 2019 已不再官方支持)
  - UE 5.0 ~ 5.3:VS 2019 / VS 2022 都行
- 必须装 VS 的 "Game development with C++" workload

## 不在范围内

- macOS / Linux 上的 UE 开发(菜单和路径完全不同)
- 蓝图项目(本 skill 强制 C++ 项目)
- UE 4.x(API 差异较大,部分模板会编译失败)
- UE 5.6+(发布后未测试,菜单可能有调整,使用时留意)
- 把已存在的蓝图工程"升级"成 C++ 工程(场景太复杂,容易出错)

## License

MIT(或你自己定)
