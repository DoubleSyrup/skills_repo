---
name: commit
description: >
  分析当前git暂存区的变更，生成符合Conventional Commits规范的
  提交信息。自动识别变更类型(feat/fix/refactor/docs/test)，
  提取影响范围，生成中英双语描述。
allowed-tools: Bash(git:*)
argument-hint: "[可选：补充说明本次变更的背景]"
---

## 任务

分析当前 git diff --cached 的内容，生成规范的commit message。

## 步骤

### 1. 获取暂存区变更

```bash
# 查看变更概览
git diff --cached --stat

# 查看变更文件列表
git diff --cached --name-only

# 查看完整diff
git diff --cached
```

### 2. 分析变更内容

分析变更，判断类型：

| 类型 | 说明 | 触发条件 |
|------|------|----------|
| `feat` | 新功能 | 新增功能代码、新接口、新组件 |
| `fix` | Bug修复 | 修复错误、异常处理、边界条件 |
| `refactor` | 重构 | 代码结构调整，不改变外部行为 |
| `docs` | 文档 | README、注释、API文档 |
| `test` | 测试 | 单元测试、集成测试、测试配置 |
| `chore` | 构建/工具 | 依赖更新、构建配置、CI/CD |
| `style` | 格式 | 代码格式化、空格、缩进 |
| `perf` | 性能 | 性能优化、缓存、算法改进 |
| `ci` | CI/CD | GitHub Actions、GitLab CI配置 |
| `build` | 构建 | Webpack、Vite、打包配置 |
| `revert` | 回滚 | 撤销之前的提交 |

### 3. 提取影响范围

分析变更涉及的模块/组件：

```bash
# 根据文件路径提取范围
git diff --cached --name-only | awk -F'/' '{print $1"/"$2}' | sort -u
```

常见范围：
- `auth` - 认证授权
- `api` - API接口
- `ui` - 用户界面
- `db` - 数据库
- `utils` - 工具函数
- `config` - 配置文件
- `deps` - 依赖
- `ci` - CI/CD

### 4. 生成 Commit Message

#### 格式规范

```
<type>(<scope>): <subject>

<body>

<footer>
```

#### 规则

1. **type**: 必填，小写
2. **scope**: 可选，小写，用括号包裹
3. **subject**: 必填，小写开头，不超过72字符，祈使句
4. **body**: 可选，解释为什么做这个变更
5. **footer**: 可选，关联Issue、Breaking Change

#### 示例

```
feat(auth): add OAuth2 login support

- Implement Google OAuth2 provider
- Add token refresh mechanism
- Update user session handling

Closes #123
```

```
fix(api): handle null response gracefully

Previously, the API would crash when receiving a null response
from the database. Now it returns a proper error message.

Fixes #456
```

### 5. 输出结果

输出以下内容：

```markdown
## Commit Message

**类型**: feat/fix/refactor/docs/test/chore
**范围**: auth/api/ui/db/...

### 中文描述
```代码块
<type>(<scope>): <简短描述>

<详细说明（如果有）>

<关联Issue（如果有）>
```代码块

### English Description
```代码块
<type>(<scope>): <short description>

<detailed explanation (if any)>

<related issues (if any)>
```代码块

### 变更摘要
- 📁 变更文件: N 个
- ➕ 新增行: +X
- ➖ 删除行: -Y
- 📝 主要变更:
  - 文件1: 变更说明
  - 文件2: 变更说明
```

## 特殊情况处理

### 多个不相关变更

如果暂存区包含多个不相关的变更，建议拆分提交：

```markdown
⚠️ 检测到多个不相关变更，建议拆分为多个提交：

**提交 1**: feat(auth): add login validation
- src/auth/validate.js
- src/auth/validate.test.js

**提交 2**: fix(api): handle timeout error
- src/api/client.js
- src/api/retry.js

使用 `git reset HEAD <file>` 取消暂存，然后分别提交。
```

### Breaking Change

如果检测到破坏性变更，在footer中标注：

```代码块
feat(api): change authentication method

BREAKING CHANGE: API now requires OAuth2 instead of API keys.
Update your client code to use the new OAuth2 flow.

Migration guide: docs/migration.md
```代码块

### Revert

如果是回滚操作：

```代码块
revert: feat(auth): add OAuth2 login support

This reverts commit abc123def456.

Reason: Critical security vulnerability found in OAuth2 implementation
```代码块

## 注意事项

- 使用祈使句（"add" 而非 "added"）
- 首字母小写
- 不超过72字符
- 不以句号结尾
- 如果用户提供了补充说明，结合说明生成更准确的描述
- 对于复杂的变更，在body中详细说明
