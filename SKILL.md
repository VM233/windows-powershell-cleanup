---
name: windows-powershell-cleanup
description: "用原生 PowerShell 清理已授权的少量 Windows 文件和空目录；在复合清理命令被拒绝时，区分命令组织问题与权限边界，采用独立核验、固定路径单项删除和事后验证。"
---

# Windows PowerShell 清理

适用于少量、归属已确认的任务产物清理。它不授权扩大删除范围，也不提供绕过工具拒绝的替代通道。

## 调用方式

把只读检查与删除放在不同的 `exec_command` 调用中。不要将条件判断、目录枚举和删除包装成一段复合 PowerShell 脚本；需要根据检查结果决定下一步时，由工具调用之间的编排作判断。

1. 只读核对目标的完整绝对路径、文件类型、实际内容及 LinkType/Target；目录链接、非预期内容和任务外改动必须先厘清。删除父目录前重新检查父目录，不沿用删除子目录前的清单。
2. 每次变更调用只删除一个已核对的固定目标。使用原生 PowerShell 和 `-LiteralPath`；不从枚举结果、通配符或拼接变量生成删除目标。
3. 少量文件逐个删除；目录确认已空后，从叶子到父目录逐个非递归删除。不要增加 `-Recurse`、`-Force` 或自动确认来让命令通过。
4. 在后续只读调用中用 `Test-Path -LiteralPath` 验证目标不存在。工具成功返回与实际结果要同时成立。

下面三个代码块表示三次独立工具调用，不能拼进一个 shell 脚本；路径仅为示例，应替换为本次已授权并核对过的目标。

先检查：

```powershell
Get-Item -LiteralPath 'D:/Workspace/Temp/EmptyArtifact' -Force | Format-List FullName,Attributes,LinkType,Target
Get-ChildItem -LiteralPath 'D:/Workspace/Temp/EmptyArtifact' -Force
```

确认是普通空目录后，只执行：

```powershell
Remove-Item -LiteralPath 'D:/Workspace/Temp/EmptyArtifact'
```

再验证：

```powershell
Test-Path -LiteralPath 'D:/Workspace/Temp/EmptyArtifact'
```

## 遇到 `blocked by policy`

- 保留拒绝原文，并区分 shell 根本未启动、PowerShell 自身报错、本地工作约定及明确的权限/安全拒绝。只有笼统的 `blocked by policy` 时，不虚构具体策略名或断言用户授权不足。
- 复合脚本被拒绝，不足以证明其中每个单项操作都被禁止。先做只读分析；若仍在授权范围内，可以把范围和组合风险实质降低为已核对的单个空目录、单次非递归原生命令，而不是原样重试复合脚本。
- 单项原生命令也被拒绝，或拒绝理由明确禁止该操作时，停止该变更。不得改用 `cmd`、Python/Node 文件 API、编码命令、保存的隐藏脚本或另一工具间接执行同一删除，也不修改审批/安全配置来放行。
- 只有检查后仍缺少必要信息或确实无法执行时才请求用户介入；已有授权无需重复询问。准确列出已完成项、剩余目标与拦截层级。

2026-10-24 的实测：带 `if` 空目录判断与 `Remove-Item` 的复合调用被工具执行层拒绝；分离只读核验后，同一 PowerShell 工具上的固定路径单项 `Remove-Item -LiteralPath` 成功，两个任务空目录及测试空目录均读回为不存在。这证明了该实例的可用调用方式，不代表对所有 policy 拒绝的通用解释。
