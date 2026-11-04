---
name: windows-powershell-cleanup
description: "用原生 PowerShell 清理已授权的 Windows 临时文件和任务产物；按范围选择整目录批量清理、明确文件清单批量删除或少量逐项删除，并核验结果、正确处理执行拦截。"
---

# Windows PowerShell 清理

按目录用途和授权范围选择清理粒度。大量临时文件优先批量处理，工具调用次数不应随文件数逐项增长。本 skill 不扩大删除授权，也不提供绕过工具拒绝的通道。

## 选择清理方式

- **整棵专用临时目录均可丢弃**：核验根目录和实际内容后，优先用一次 `Remove-Item -LiteralPath '完整绝对路径' -Recurse` 删除整棵目录；多个已核验的同类根目录可在一次调用中用明确的 `-LiteralPath` 数组处理。不必先逐个删除文件、再逐层删空目录。
- **大量目标散落在需要保留的目录中**：只读阶段确定明确的候选清单，在一次 PowerShell 调用中用 `-LiteralPath` 数组或循环批量删除。清单可来自授权根目录内的枚举，必须逐项核对路径边界与归属；不得在删除阶段扩大筛选范围或直接删除混有保留内容的父目录。若需保留临时根目录本身，批量删除已核验的直属子项。
- **少量任务文件或空目录**：保留简单的固定路径逐项删除；空目录由叶子到父目录逐个非递归删除。不要把这个小规模流程强加给大量临时文件。
- **用途混杂或归属不明**：先厘清哪些内容获准删除；不能仅凭扩展名、目录叫 `Temp`、被 Git 忽略或文件数量多就整目录清理。

## 批量清理的核验与执行

1. 在独立的只读调用中核对完整绝对路径、文件类型、目录用途和实际清单。递归删除前，确认解析后的根目录仍位于本次授权范围内；计算得到的路径也必须核对最终绝对路径与目录边界，不能只用无分隔符边界的字符串前缀判断。
2. 检查根目录及子项的 `LinkType`、`Target` 或 `ReparsePoint`。不跟随链接扩展清理范围；遇到未厘清的链接、正式资产、真实存档或任务外内容时，停止该批次并缩小到已确认的目标。不得递归删除磁盘根目录、用户主目录或项目根目录来代替临时目录清理。
3. 大量文件只需一次范围核验：枚举实际候选，汇总数量、体积、类型、链接及异常项，输出必要的路径样本；不用把上万行清单全部写进对话，也不用逐文件读取内容或发起工具调用。只有新证据、失败或目录仍被写入时才补查受影响范围；必要时先停止生成临时文件的任务。
4. 在后续独立调用中用原生 PowerShell 与 `-LiteralPath` 删除已核验目标。`-Recurse` 用于授权内的子树；`-Force` 仅在需要处理这些目标的隐藏或只读属性时使用，不能覆盖安全权限或作为被拒绝后的绕过手段。不要跨 shell 执行删除。
5. 再用一次只读调用验证：整目录删除检查 `Test-Path` 为 `False`；保留根目录时确认目标子项已清空、应保留内容仍在。发生部分失败时核对剩余目标并如实报告，不因一条命令返回成功就认定清理完成。

### 专用临时目录示例

以下是三次独立工具调用。路径仅为示例；只有已确认整棵目录均为获准丢弃的临时产物，且无未厘清的链接时，才执行第二步。

先核验目录与实际内容；汇总和样本来自本次完整枚举：

```powershell
Get-Item -LiteralPath 'D:/Workspace/.tmp/CaptureRun' -Force -ErrorAction Stop |
    Select-Object FullName, PSIsContainer, Attributes, LinkType, Target
$cleanupItems = @(Get-ChildItem -LiteralPath 'D:/Workspace/.tmp/CaptureRun' -Force -Recurse -ErrorAction Stop)
$cleanupItems | Group-Object PSIsContainer | Select-Object Name, Count
$cleanupItems | Where-Object { -not $_.PSIsContainer } | Measure-Object Length -Sum
$cleanupItems | Where-Object { $_.Attributes -band [IO.FileAttributes]::ReparsePoint } |
    Select-Object FullName, LinkType, Target
$cleanupItems | Select-Object -First 10 FullName, Attributes
```

确认范围后，一次删除整棵目录；如已核实其中包含应删除的隐藏或只读临时项，可加 `-Force`：

```powershell
Remove-Item -LiteralPath 'D:/Workspace/.tmp/CaptureRun' -Recurse -ErrorAction Stop
```

最后验证：

```powershell
Test-Path -LiteralPath 'D:/Workspace/.tmp/CaptureRun'
```

## 遇到执行拦截

- 保留拒绝原文，区分工具执行层拒绝、PowerShell 自身报错、本地工作约定和明确的权限拒绝。笼统的 `blocked by policy` 不能证明具体策略名或用户授权不足。
- 复合调用被拒绝后，先只读分析。仅在仍获授权且范围或组合风险实质降低时，才改为更简单、更小的原生命令；不要无依据反复重试，也不要因此把所有批量任务永久降级为逐文件调用。
- 单项原生命令也被拒绝，或理由明确禁止该操作时，停止该变更。不得改用 `cmd`、Python/Node 文件 API、编码命令、隐藏脚本或其他工具间接执行同一删除，也不修改审批或安全配置放行。
- 已有授权无需重复询问。只有缺少必要信息或无法执行时才请求用户介入，并准确列出完成项、剩余范围和拦截层级。

## 依据与验收边界

微软的 [Remove-Item 文档](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/remove-item?view=powershell-7.5) 支持 `-LiteralPath` 路径数组、递归删除及隐藏/只读项处理；[Get-ChildItem 文档](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-childitem?view=powershell-7.5) 说明递归枚举与链接行为。与旧版逐文件删除不同，批量方案以已核验的目录或候选清单为操作单位；此前单次复合调用的拒绝记录不构成全面禁用批量处理的依据。

成功条件是授权目标已清理、任务外内容保留；越界、归属不明、未厘清的链接或明确拒绝均是停止条件。验证只针对有边界的自建临时样本，不扩大到真实用户资料。临时验证产物放在已忽略目录，交付前清理，不上传回执或缓存。
