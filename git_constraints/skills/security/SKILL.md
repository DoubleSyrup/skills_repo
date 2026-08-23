---
name: security
description: >
  全面扫描项目的安全风险：硬编码密钥、依赖漏洞、
  OWASP Top 10代码模式、不安全的配置文件。输出
  按严重等级排序的安全报告。
allowed-tools: Read, Grep, Glob, Bash(npm:audit*, pip:audit*, grep:*)
---

## 任务

全面审计项目安全风险。

## 扫描项目

### 1. 硬编码密钥扫描

搜索以下模式：
- API Key / Secret / Token / Password 的赋值语句
- AWS_ACCESS_KEY、GITHUB_TOKEN等环境变量的硬编码
- .env文件是否被加入版本控制（检查.gitignore）
- 私钥文件（*.pem, *.key）是否在仓库中

执行扫描：

```bash
# 检查硬编码密钥
grep -rn "(?i)(password|passwd|pwd)\s*[:=]" --include="*.{js,ts,py,java,go,rb,php}" .
grep -rn "(?i)(api[_-]?key|apikey)\s*[:=]" --include="*.{js,ts,py,java,go,rb,php}" .
grep -rn "(?i)(token|secret)\s*[:=]" --include="*.{js,ts,py,java,go,rb,php}" .
grep -rn "AKIA[0-9A-Z]{16}" .
grep -rn "-----BEGIN.*PRIVATE KEY-----" .

# 检查私钥文件
find . -name "*.pem" -o -name "*.key" -o -name "*.p12" 2>/dev/null

# 检查.env文件是否在版本控制中
git ls-files | grep -E "\.env$|\.env\."
```

### 2. 依赖漏洞检查

```bash
# Node.js 项目
if [ -f "package.json" ]; then
    npm audit --json 2>/dev/null || echo "npm audit failed"
fi

# Python 项目
if [ -f "requirements.txt" ] || [ -f "Pipfile" ]; then
    pip audit --format json 2>/dev/null || echo "pip-audit not installed"
fi
```

检查要点：
- 已知CVE的依赖版本
- 超过2年未更新的依赖
- 已废弃的依赖包

### 3. OWASP Top 10 代码模式

扫描以下危险模式：

```bash
# SQL注入 - SQL拼接
grep -rn "SELECT.*FROM.*WHERE.*["']\s*+\s*" --include="*.{js,ts,py,java,php}" .
grep -rn "query\s*(.*\$\{" --include="*.{js,ts}" .
grep -rn "execute\s*(.*%\s*" --include="*.py" .

# 命令注入 - eval/exec
grep -rn "\beval\s*(" --include="*.{js,ts,py}" .
grep -rn "\bexec\s*(" --include="*.{js,ts,py}" .
grep -rn "\bnew\s*Function\s*(" --include="*.{js,ts}" .

# XSS - 未转义输出
grep -rn "innerHTML\s*=" --include="*.{js,ts}" .
grep -rn "dangerouslySetInnerHTML" --include="*.{js,ts,jsx,tsx}" .
grep -rn "document\.write\s*(" --include="*.{js,ts}" .

# 路径遍历
grep -rn "readFile\s*(.*\.\.\/" --include="*.{js,ts}" .
grep -rn "open\s*(.*\.\.\/" --include="*.py" .

# 弱加密
grep -rn "createHash\s*(\s*['\"]md5['\"]" --include="*.{js,ts}" .
grep -rn "createHash\s*(\s*['\"]sha1['\"]" --include="*.{js,ts}" .
grep -rn "hashlib\.md5\|hashlib\.sha1" --include="*.py" .

# 不安全随机数用于安全场景
grep -rn "Math\.random\s*(" --include="*.{js,ts}" .
```

### 4. 配置文件检查

```bash
# CORS配置
grep -rn "Access-Control-Allow-Origin.*\*" --include="*.{js,ts,json,yml,yaml}" .
grep -rn "origin:\s*['\"]?\*['\"]?" --include="*.{js,ts,json,yml,yaml}" .

# Debug模式
grep -rn "DEBUG\s*[:=]\s*true" --include="*.{js,ts,json,yml,yaml,env}" .
grep -rn "debug\s*[:=]\s*true" --include="*.{js,ts,json,yml,yaml}" .

# 数据库连接
grep -rn "mongodb://[^?]*[^s]$" --include="*.{js,ts,json,yml,yaml}" .
grep -rn "mysql://[^?]*[^s]$" --include="*.{js,ts,json,yml,yaml}" .

# JWT密钥
grep -rn "jwt[_-]?secret\s*[:=]" --include="*.{js,ts,json,yml,yaml}" .
```

## 输出格式

整理扫描结果，按以下格式输出：

**🔴 严重（立即修复）** — 密钥泄露、已知高危CVE
```
[文件:行号] 问题描述
风险: 具体风险说明
修复:
```代码块
修复代码
```代码块
```

**🟡 高危（本周修复）** — SQL注入、XSS、弱加密
```
[文件:行号] 问题描述
风险: 具体风险说明
修复:
```代码块
修复代码
```代码块
```

**🔵 中危（下个迭代修复）** — 过时依赖、配置不当
```
[文件:行号] 问题描述
风险: 具体风险说明
修复:
```代码块
修复代码
```代码块
```

**⚪ 低危（有空再说）** — 代码风格安全建议
```
[文件:行号] 问题描述
建议: 改进建议
```

## 注意事项

- 只报告真正存在的问题，不要猜测
- 如果没有发现问题，明确说明"未发现安全风险"
- 修复建议必须是可直接使用的代码
- 对于误报风险较高的模式，标注"可能误报"
