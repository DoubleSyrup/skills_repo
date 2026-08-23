# Git Constraints Plugin

Git工作流约束插件，为Claude Code提供完整的Git提交前检查和代码质量保障能力。

## 📦 插件结构

```
git_constraints/
├── .claude-plugin/              # 插件元数据
│   └── plugin.json              # 插件清单
│
├── skills/                      # Skills定义
│   ├── security/
│   │   └── SKILL.md            # 安全扫描技能
│   ├── review/
│   │   └── SKILL.md            # 代码审查技能
│   └── commit/
│       └── SKILL.md            # 规范提交技能
│
├── commands/                    # 自定义命令
│   └── status.md               # 仓库状态命令
│
├── agents/                      # Subagent定义
│   └── security-reviewer.md    # 安全审查Agent
│
├── hooks/                       # Hook配置
│   └── hooks.json              # Hook定义
│
├── scripts/                     # 辅助脚本
│
└── README.md                    # 本文档
```

## 🚀 安装

### 方式一：Skills目录Plugin（推荐）

将插件目录放到 `~/.claude/skills/` 或项目的 `.claude/skills/` 目录：

```bash
# 用户级别（所有项目可用）
cp -r git_constraints ~/.claude/skills/

# 项目级别（仅当前项目）
cp -r git_constraints .claude/skills/
```

### 方式二：CLI安装

```bash
# 初始化插件
claude plugin init git_constraints

# 或从本地目录安装
claude plugin install ./git_constraints
```

## 📖 Skills说明

### 🔐 /security - 安全扫描

全面扫描项目的安全风险：
- 硬编码密钥（API Key、Password、Token）
- 依赖漏洞（npm audit、pip audit）
- OWASP Top 10 代码模式（SQL注入、XSS、命令注入）
- 不安全配置文件

**使用**:
```bash
/security
```

**输出**: 按严重等级排序的安全报告
- 🔴 严重（立即修复）
- 🟡 高危（本周修复）
- 🔵 中危（下个迭代修复）
- ⚪ 低危（有空再说）

---

### 👀 /review - 代码审查

审查当前分支相对于主分支的代码变更：
- 安全性审查（SQL注入、XSS、命令注入、硬编码密钥）
- 性能审查（N+1查询、内存泄漏、循环I/O）
- 代码质量审查（DRY、函数长度、错误处理）

**使用**:
```bash
/review
```

**输出**: 结构化审查报告
- 🔴 P0 — 必须修复（阻塞合并）
- 🟡 P1 — 建议修复
- 🟢 P2 — 可以改进
- 💡 P3 — 锦上添花

---

### 📝 /commit - 规范提交

生成符合Conventional Commits规范的提交信息：
- 自动识别变更类型
- 提取影响范围
- 生成中英双语描述

**使用**:
```bash
/commit
/commit "补充说明本次变更的背景"
```

**输出**: 中英双语commit message

## 🔄 推荐工作流

```bash
# 1. 开发功能
git checkout -b feature/my-feature
# ... 编写代码 ...

# 2. 暂存变更
git add .

# 3. 安全扫描
/security
# 修复发现的安全问题

# 4. 代码审查
/review
# 修复P0问题，考虑P1建议

# 5. 生成提交信息
/commit

# 6. 提交代码
git commit -m "feat(scope): description"

# 7. 推送并创建PR
git push origin feature/my-feature
```

## 🤝 贡献

欢迎提交Issue和Pull Request！

## 📄 License

MIT

## 🔗 相关资源

- [Conventional Commits](https://www.conventionalcommits.org/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Claude Code Plugins Documentation](https://code.claude.com/docs/zh-CN/plugins)
