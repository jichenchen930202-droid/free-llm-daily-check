# GitHub 仓库发布指南 — free-llm-daily-check

## 前提条件

你需要在电脑上安装 Git。如果尚未安装，请访问 https://git-scm.com/download/win 下载安装。

---

## 第一步：创建 GitHub 仓库

1. 登录 [GitHub](https://github.com)
2. 点击右上角 **+** → **New repository**
3. 填写信息：
   - **Repository name**: `free-llm-daily-check`
   - **Description**: `Agnes AI Skill - Daily Free LLM Retrieval`
   - **Public**（公开）
4. 不要勾选 "Initialize with README"
5. 点击 **Create repository**
6. 记下你的仓库地址，格式为：
   ```
   https://github.com/YOUR_USERNAME/free-llm-daily-check.git
   ```
   （将 `YOUR_USERNAME` 替换为你的 GitHub 用户名）

---

## 第二步：推送代码到 GitHub

打开 **命令提示符 (cmd)** 或 **PowerShell**，依次执行以下命令：

```powershell
# 1. 进入项目目录
cd "C:\Users\Administrator\Documents\AgnesCode\free-llm-daily-check"

# 2. 初始化 Git 仓库
git init

# 3. 添加所有文件
git add .

# 4. 提交更改
git commit -m "Initial commit: free-llm-daily-check Agnes AI skill"

# 5. 关联远程仓库（请将 YOUR_USERNAME 替换为你的实际 GitHub 用户名）
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/free-llm-daily-check.git

# 6. 推送到 GitHub
git push -u origin main
```

> **注意**：如果你使用 SSH 方式推送，第 5 步改为：
> ```
> git remote add origin git@github.com:YOUR_USERNAME/free-llm-daily-check.git
> ```
> 第 6 步直接使用 `git push -u origin main`

---

## 验证

推送完成后，访问以下地址确认仓库内容：
```
https://github.com/YOUR_USERNAME/free-llm-daily-check
```

你应该能看到：
- ✅ `SKILL.md` — Agnes AI 核心指令
- ✅ `README.md` — 项目说明文档
- ✅ `LICENSE` — MIT 许可证
- ✅ `package.json` — 项目元数据
- ✅ `.gitignore` — Git 忽略规则
- ✅ `PUBLISH_GUIDE.md` — 本指南

---

## 常见问题

### Q: 推送时提示认证失败？
A: 如果使用 HTTPS，GitHub 需要使用 [Personal Access Token](https://github.com/settings/tokens) 代替密码。

### Q: 想修改 GitHub 用户名？
A: 运行以下命令更新远程地址：
```powershell
git remote set-url origin https://github.com/NEW_USERNAME/free-llm-daily-check.git
```
