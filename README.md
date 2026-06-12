# UE-Skills

一组用于 Unreal Engine 5.3 C++ 开发的 Cowork / Claude skill。

每个 skill 是一个引导式对话流程,让 Claude 在 Cowork 或 AionUI(DeepSeek)里手把手带你完成一个具体的 UE 开发任务,而不是只回答"应该怎么做"。

## 已有 skill

| Skill | 用途 | 状态 |
|---|---|---|
| [ue-plugin-bootstrap](./ue-plugin-bootstrap/) | 从零搭一个 UE 5.3 C++ 插件,联动 VS 和蓝图,终点是一个能在蓝图里看到并调用的 `UWorldSubsystem` 基类 | ✅ 可用 |

## 计划中的 skill

| Skill | 用途 |
|---|---|
| ue-cpp-class | 在已有插件里快速生成一个新 UCLASS,自动放对目录、加对依赖 |
| ue-build | 调用 UnrealBuildTool 编译,自动解析报错并定位行号 |

## 在 Cowork 里怎么用

1. 把对应 skill 目录(例如 `ue-plugin-bootstrap/`)放进 Cowork 的 skills 目录
2. 在对话里输入 `/<skill-name>`,例如 `/ue-plugin-bootstrap`
3. Cowork 加载 `SKILL.md`,从第 0 步开始引导你

## 在 AionUI / DeepSeek 里怎么用

DeepSeek 没有 Cowork 的工具调用能力,但 `SKILL.md` 本身就是一段引导 prompt:

1. 把对应 `SKILL.md` 的全文复制进 AionUI 的对话
2. 让 DeepSeek 当作"系统指令"执行
3. DeepSeek 切换到**纯文字模式**:用对话形式给你每步指令,你手动建文件、跑命令

## License

MIT
