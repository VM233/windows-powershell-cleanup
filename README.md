# Windows PowerShell 清理 Skill

供 Codex 等支持 `SKILL.md` 的编码助手使用的 Windows 清理技能，适用于少量、已获授权的任务文件和空目录。

## 工作方式

- 先只读核对绝对路径、文件类型、实际内容及目录链接。
- 将检查、删除、验证拆为独立工具调用。
- 每次只删除一个已核对的固定路径，使用原生 PowerShell 与 `-LiteralPath`。
- 空目录从叶子到父目录逐个非递归删除，随后确认目标不存在。
- 遇到执行拦截时保留原文并判断来源；明确拒绝后停止，不借其他 shell、脚本或工具绕过。

完整操作规则与示例见 [SKILL.md](SKILL.md)。本仓库提供给助手的操作指引，不包含自动删除程序。

## 安装到 Codex

在 PowerShell 中运行，将仓库直接克隆到默认用户技能目录下：

```powershell
git clone https://github.com/VM233/windows-powershell-cleanup.git (Join-Path $env:USERPROFILE '.codex\skills\windows-powershell-cleanup')
```

如果已存在同名 skill，请直接使用现有目录，或在确认其内容后按自己的方式更新；不要覆盖未核对的文件。使用自定义技能目录时，将克隆目标改为对应位置。

## 使用

在助手中调用 `$windows-powershell-cleanup`，并说明本次已授权清理的具体目标，例如：

> 使用 $windows-powershell-cleanup 检查并清理 D:/Workspace/Temp/EmptyArtifact；保留其他文件，完成后验证结果。

Skill 不会扩大用户授权的删除范围。检查发现任务外文件、非预期内容或目录链接时，应先厘清再操作。

## 文件

- `SKILL.md`：技能说明、操作步骤与执行拦截处理规则。
- `agents/openai.yaml`：显示名称、简介及默认调用提示。
