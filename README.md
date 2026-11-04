# Windows PowerShell 清理 Skill

供 Codex 等支持 `SKILL.md` 的编码助手使用的 Windows 清理技能，适用于已获授权的临时文件、任务产物和空目录，支持大量文件批量清理。

## 工作方式

- 整棵专用临时目录均可丢弃时，核验后一次递归清理；大量明确目标可在一次 PowerShell 调用中批量删除。
- 少量文件和空目录仍可逐项处理；混有正式资料的目录先区分授权范围。
- 使用原生 PowerShell 与 `-LiteralPath`，检查实际路径、内容和目录链接，随后验证清理结果及保留内容。
- 大量候选采用一次枚举、汇总和必要样本，避免逐文件工具调用与反复输出完整清单。
- 遇到执行拦截时保留原文并判断来源；明确拒绝后停止，不借其他 shell、脚本或工具绕过。

完整操作规则与批量清理示例见 [SKILL.md](SKILL.md)。本仓库提供给助手的操作指引，不包含自动删除程序。

## 安装到 Codex

在 PowerShell 中运行，将仓库直接克隆到默认用户技能目录下：

```powershell
git clone https://github.com/VM233/windows-powershell-cleanup.git (Join-Path $env:USERPROFILE '.codex\skills\windows-powershell-cleanup')
```

如果已存在同名 skill，请直接使用现有目录，或在确认其内容后按自己的方式更新；不要覆盖未核对的文件。使用自定义技能目录时，将克隆目标改为对应位置。

## 使用

在助手中调用 `$windows-powershell-cleanup`，并说明本次已授权清理的具体目标，例如：

> 使用 $windows-powershell-cleanup 清理 D:/Workspace/.tmp/CaptureRun，其中整棵目录都是本次任务的临时截图；批量处理，保留其他目录并验证结果。

Skill 不会扩大用户授权的删除范围。检查发现任务外文件、非预期内容或目录链接时，应先厘清再操作。

## 文件

- `SKILL.md`：清理方式选择、批量核验示例与执行拦截处理规则。
- `agents/openai.yaml`：显示名称、简介及默认调用提示。
