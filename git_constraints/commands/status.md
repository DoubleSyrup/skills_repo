---
name: status
description: 显示当前Git仓库状态和待处理事项
allowed-tools: Bash(git:*)
---

## 任务

显示当前Git仓库的完整状态信息。

## 输出内容

1. **分支信息**: 当前分支、跟踪的远程分支
2. **工作区状态**: 修改的文件、新增的文件、删除的文件
3. **暂存区状态**: 已暂存的变更
4. **最近提交**: 最近3次提交记录
5. **待处理事项**: TODO/FIXME统计

```bash
# 分支信息
echo "📌 分支信息"
git branch -vv

# 工作区状态
echo ""
echo "📁 工作区状态"
git status --short

# 暂存区状态
echo ""
echo "📋 暂存区状态"
git diff --cached --stat

# 最近提交
echo ""
echo "📝 最近提交"
git log --oneline -5

# TODO/FIXME统计
echo ""
echo "⚠️ 待处理事项"
grep -rn "TODO\|FIXME\|HACK" --include="*.{js,ts,py,java,go}" . 2>/dev/null | wc -l
echo "个 TODO/FIXME 待处理"
```
