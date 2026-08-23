---
name: review
description: >
  审查当前分支相对于主分支的所有代码变更，从安全性、
  性能、代码质量三个维度输出结构化审查报告，标注
  严重等级(P0-P3)和具体修复建议。
allowed-tools: Bash(git:*), Read, Grep, Glob
---

## 任务

对当前分支的代码变更做全面审查。

## 步骤

### 1. 获取变更概览

```bash
# 查看当前分支
git branch --show-current

# 获取变更文件列表
git diff main...HEAD --name-only

# 获取变更统计
git diff main...HEAD --stat

# 获取完整diff
git diff main...HEAD
```

### 2. 逐文件分析

对每个变更文件，从三个维度审查：

#### 维度一：安全性（权重最高）

检查以下风险模式：

**注入漏洞**
- SQL拼接：`SELECT * FROM users WHERE id = '${userId}'`
- 命令注入：`exec('rm -rf ' + userInput)`
- eval使用：`eval(userInput)`

**认证授权**
- 硬编码密码/密钥
- 缺少权限检查
- JWT密钥暴露

**数据安全**
- 敏感信息日志输出
- 未加密传输
- 不安全的数据存储

```bash
# 检查SQL注入风险
grep -n "SELECT.*FROM.*WHERE" $file | grep -v "?"

# 检查命令注入
grep -n "exec\|eval\|spawn" $file

# 检查硬编码密钥
grep -n "password\|secret\|token\|key" $file | grep -i "="
```

#### 维度二：性能

检查以下性能问题：

**数据库**
- N+1查询：循环中执行数据库查询
- 缺少索引：WHERE/JOIN条件字段
- SELECT *：查询不需要的字段

**内存**
- 未关闭的连接/流
- 大数组无限增长
- 事件监听器泄漏

**计算**
- 不必要的循环内I/O
- 重复计算
- 同步阻塞操作

```bash
# 检查N+1查询（循环中的await）
grep -n "for.*await\|forEach.*await\|map.*await" $file

# 检查SELECT *
grep -n "SELECT \* FROM" $file

# 检查未关闭资源
grep -n "\.open\|\.connect\|createStream" $file
```

#### 维度三：代码质量

检查以下质量问题：

**设计原则**
- 重复代码（DRY违反）
- 函数过长（>50行）
- 类过大（>300行）
- 过深嵌套（>3层）

**可维护性**
- 魔法数字
- 硬编码配置
- 缺少注释
- 命名不规范

**健壮性**
- 缺少错误处理
- 空值未检查
- 异常吞没
- 资源未释放

```bash
# 检查函数长度
awk '/^function|^def|^fn|^func/{start=NR} /^}/{if(start) print FILENAME":"NR" ("NR-start+1" lines)"; start=0}' $file

# 检查魔法数字
grep -n "[^a-zA-Z_][0-9]\{2,\}[^a-zA-Z_]" $file

# 检查TODO/FIXME
grep -n "TODO\|FIXME\|HACK\|XXX" $file
```

### 3. 生成审查报告

## 输出格式

```
# 代码审查报告

**分支**: feature/xxx → main
**变更文件**: N 个文件, +X 行, -Y 行
**审查时间**: YYYY-MM-DD HH:MM

---

## 🔴 P0 — 必须修复（阻塞合并）

### [文件名:行号] 问题标题

**问题**: 详细问题描述

**风险**: 可能导致的安全漏洞/性能问题/线上故障

**修复**:
```代码语言
// 修复前
原代码

// 修复后
修复代码
```

---

## 🟡 P1 — 建议修复

### [文件名:行号] 问题标题

**问题**: 详细问题描述
**影响**: 可能导致的问题
**建议**: 修复建议

---

## 🟢 P2 — 可以改进

### [文件名:行号] 问题标题

**问题**: 详细问题描述
**建议**: 改进建议

---

## 💡 P3 — 锦上添花

- [文件名:行号] 改进建议
- [文件名:行号] 改进建议

---

## 审查总结

| 维度 | P0 | P1 | P2 | P3 |
|------|----|----|----|----|
| 安全性 | N | N | N | N |
| 性能 | N | N | N | N |
| 代码质量 | N | N | N | N |

**结论**:
- [ ] 有P0问题，必须修复后合并
- [ ] 无P0问题，可以合并（建议修复P1）
```

## 注意事项

- P0问题必须给出可直接使用的修复代码
- 如果没有发现P0问题，明确说明"无阻塞性问题"
- 不要为了凑数量而列出无关紧要的问题
- 审查意见要具体，指出问题所在行号
- 修复建议要可操作，最好是完整的代码片段
- 对于不确定的问题，标注"建议确认"而非直接列为P0
