# GitHub 仓库发布指南 — free-llm-daily-check

## 前提条件

你需要在电脑上安装 Git。如果尚未安装，请访问 https://git-scm.com/download/win 下载安装。

---

## 第一步：创建 GitHub 仓库

1. 登录 [GitHub](https://github.com)
2. 点击右上角 **+** → **New repository**
3. 填写信息：
   - **Repository name**: `free-llm-daily-check`
   - **Description**: `Agent-agnostic skill: daily free LLM retrieval`
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
git commit -m "Initial commit: free-llm-daily-check skill"

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
- ✅ `SKILL.md` — Skill 核心指令（agent 无关）
- ✅ `README.md` — 项目说明文档
- ✅ `LICENSE` — MIT 许可证
- ✅ `package.json` — 项目元数据
- ✅ `.gitignore` — Git 忽略规则
- ✅ `PUBLISH_GUIDE.md` — 本指南

---

## 六、接入任意 Agent（tool_binding 示例）

本 Skill **不绑定任何具体工具实现**，仅声明需要两类能力，由调用方在 `tool_binding` 中把抽象能力映射到自己的真实工具。

### 1. 依赖能力签名
- `web_search(query: str, keyword_groups?: list[str], max_results?: int) -> list[{title, url, snippet}]`
- `web_fetch(url: str, prompt?: str) -> str`

### 2. tool_binding 模板（由调用方填写）
```
## tool_binding（由调用方填写）
web_search: <调用方实际提供的联网检索工具>
web_fetch:  <调用方实际提供的网页抓取工具>
```
Skill 执行步骤中以 `{{web_search}}` / `{{web_fetch}}` 指代上述绑定。

### 3. 各 agent 填写样例

**① WorkBuddy（通用 agent）**
```yaml
tool_binding:
  web_search: WebSearch      # 参数: query + query_keyword_groups(最多5组) + maxItems
  web_fetch:  WebFetch       # 参数: url + prompt
```
> 注：Skill 的 `keyword_groups` 对应 WebSearch 的 `query_keyword_groups`。

**② Agnes（历史环境，兼容）**
```yaml
tool_binding:
  web_search: web_search     # Agnes 内置联网检索
  web_fetch:  read_webpage   # Agnes 内置网页读取
```

**③ Python / LangChain agent（自实现绑定层）**
```python
# 调用方实现两个函数，供 skill 步骤中的 {{web_search}} / {{web_fetch}} 调用
def web_search(query: str, keyword_groups=None, max_results: int = 10):
    # 接入 SerpAPI / Tavily / Bing 等
    return [{"title": t, "url": u, "snippet": s} for t, u, s in ...]

def web_fetch(url: str, prompt: str = None):
    # 接入 BeautifulSoup / Playwright / Jina Reader 等
    return page_text
```

**④ Claude / Anthropic agent（tool-use）**
```json
{
  "tools": [
    {"name": "web_search", "input_schema": {"type": "object", "properties": {
      "query": {"type": "string"},
      "keyword_groups": {"type": "array", "items": {"type": "string"}},
      "max_results": {"type": "integer"}
    }, "required": ["query"]}},
    {"name": "web_fetch", "input_schema": {"type": "object", "properties": {
      "url": {"type": "string"}, "prompt": {"type": "string"}
    }, "required": ["url"]}}
  ]
}
```

### 4. 带参数调用示例
```
free-llm-daily-check --as_of_date 2026-07-27 --region cn --categories api,chat,aggregator --output markdown --lang zh
```
- `output=json` 时返回对象数组（字段：`category, name, benefit, entry, validity, limits, evidence`），便于程序化消费。

### 5. 集成检查清单
- [ ] 绑定 `tool_binding.web_search` 与 `tool_binding.web_fetch` 到真实工具
- [ ] 透传输入参数：`as_of_date` / `region` / `categories` / `output` / `lang`
- [ ] 读取输出：Markdown 三表（人读）或 JSON 数组（程序消费）
- [ ] 不向 Skill 注入任何会话上下文 / 环境变量

---

## 常见问题

### Q: 推送时提示认证失败？
A: 如果使用 HTTPS，GitHub 需要使用 [Personal Access Token](https://github.com/settings/tokens) 代替密码。

### Q: 想修改 GitHub 用户名？
A: 运行以下命令更新远程地址：
```powershell
git remote set-url origin https://github.com/NEW_USERNAME/free-llm-daily-check.git
```

### Q: 如何把 Skill 接入我自己的 agent？
A: 见上方「六、接入任意 Agent（tool_binding 示例）」。核心是把 `web_search` / `web_fetch` 两个能力绑定到你的工具，并透传输入参数即可，无需修改 Skill 内部。
