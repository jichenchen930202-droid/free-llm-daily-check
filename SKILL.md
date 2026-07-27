---
name: free-llm-daily-check
description: 'Retrieves large language models that are free to use on a given day. Strictly filters models that are permanently free or have daily-reset free quotas; excludes trials, time-limited, paid, or expiring resources. Agent-agnostic: depends only on a configurable web-search/web-fetch capability and a set of explicit input parameters, with a defined input/output contract for integration by any caller. Use when the user asks for free AI model recommendations, free LLM lists, or no-cost AI options.'
metadata:
  argument-hint: '[--as_of_date YYYY-MM-DD] [--region global|cn] [--categories api,chat,aggregator] [--output markdown|json] [--lang en|zh]'
---

# 当日免费可用大模型检索 Skill

## 概述
检索**指定日期**仍可免费使用的大语言模型（LLM），严格筛选「永久免费」或「每日重置免费额度」的模型，排除试用、限时、需付费、当日/近期到期的资源。

**设计原则（agent 无关 / 解耦）：**
- 不依赖任何调用方的会话上下文、环境变量或私有状态。
- 仅依赖一组**显式声明的输入参数**和一个**调用方提供的联网检索能力**。
- 输入 / 输出均有明确契约，可直接被任意 agent 以程序化方式集成调用。

## 接口契约 (Interface Contract)

### 输入参数 (Input)
| 参数 | 类型 | 默认值 | 必填 | 说明 |
|---|---|---|---|---|
| `as_of_date` | string `YYYY-MM-DD` | 执行当日日期 | 否 | 检索基准日。所有时效判断（有效期 ≥ 该日、未到期）以此为准。调用方不传则自动取运行环境当前日期。 |
| `region` | enum `global` \| `cn` | `global` | 否 | `cn`：额外标注「中国大陆是否可直连」。 |
| `categories` | list[`api`,`chat`,`aggregator`] | 全部 | 否 | 限定输出类别；可多选（逗号分隔）。 |
| `output` | enum `markdown` \| `json` | `markdown` | 否 | 输出格式；`json` 便于 agent 程序化消费。 |
| `lang` | enum `en` \| `zh` | `zh` | 否 | 检索词与输出语言。 |

> 调用方必须以约定方式传入上述参数（命令行参数、结构化入参或对话中提取），本 skill 不读取任何隐含上下文或环境变量。

### 依赖能力 (Required Capabilities)
本 skill 需要调用方提供以下两类能力（**工具名由调用方绑定，本 skill 不假定具体实现**）：
- `web_search(query: str, keyword_groups?: list[str], max_results?: int) -> list[{title, url, snippet}]`
- `web_fetch(url: str, prompt?: str) -> str`

调用方在集成时填写 `tool_binding`（模板，非运行时依赖）：
```
## tool_binding（由调用方填写）
web_search: <调用方实际提供的联网检索工具>
web_fetch:  <调用方实际提供的网页抓取工具>
```
执行步骤中以 `{{web_search}}` / `{{web_fetch}}` 指代上述能力。

### 输出规范 (Output Contract)
- `markdown`：按类别输出表格（见「输出格式」），并附「证据来源」区块。
- `json`：返回对象数组，每条含字段 `category, name, benefit, entry, validity, limits, evidence`，结构与表格列一一对应。
- 仅输出通过全部筛选条件的模型；不编造、不输出过期/付费/限时失效资源。

## 触发条件
当用户询问「今天有哪些免费大模型可以用」「免费 AI 模型推荐」「不用付费的 AI」「免费大模型清单」「free LLM list」等类似需求时触发。

## 执行步骤

### Step 1: 多渠道检索
使用调用方提供的 `{{web_search}}` 进行多维检索。检索词中的 `{year}` 取 `as_of_date` 的年份，`{date}` 取 `as_of_date`：
1. 英文永久免费 API：`free LLM API permanent free tier {year} no credit card`
2. 中文永久免费 API：`免费大模型API 永久免费 {year} 无需登录`
3. 在线免登录对话：`free AI chat no sign up no login {year}`
4. 国内可用免费模型：`国内免费大模型 永久免费 {year}`
5. 聚合平台免费模型：`Poe.com free models OpenRouter free {year}`

每次检索后，对重要来源使用 `{{web_fetch}}` 验证实际内容与时效性，并将来源 URL 记入「证据来源」。

### Step 2: 硬性筛选（全部满足才收录）
1. **使用模式**：永久免费 / 每日重置免费额度。不接受一次性试用、新用户专享、限时活动。
2. **时效要求**：免费权益有效期 ≥ `as_of_date`。排除 `as_of_date` 当日/之前到期的资源。
3. **可用性**：无需特殊资格认证、无地区锁（或按 `region` 标注可直连）、能直接调用对话/推理。
4. **黑名单**：
   - ❌ 仅新用户注册赠送额度（用完即止）
   - ❌ 需信用卡验证的「免费」
   - ❌ 当日/当月有效限时活动
   - ❌ 额度耗尽后无法使用
   - ❌ 需付费会员
   - ❌ 隐性扣费平台
   - ❌ 仅限特定地区（如仅 US）
   - ❌ 需企业邮箱/学生认证

### Step 3: 时效校验
- 确认免费状态为永久 / 每日重置 / 长期有效；排除明确标注截止日期的限时免费。
- 区分「免费层」与「试用额度」。

### Step 4: 可用性核验
- 优先收录官方 API/控制台入口。
- 标注是否需注册、是否需信用卡；`region=cn` 时标注中国大陆是否可直连；标注 API 兼容性（OpenAI 兼容 / 原生 SDK 等）。

### Step 5: 信息整理
- 去重合并相同模型的不同入口。
- 按 `categories` 分到三类，每类独立成表（见输出格式）。
- 标注每个模型的具体免费额度上限。

## 输出格式

### Markdown（默认）
按类别输出三张表（仅输出 `categories` 指定的类别）：

**API 类**
| 模型/平台名称 | 免费权益详情 | 使用入口 | 免费有效期 | 使用限制 | 证据来源 |
|---|---|---|---|---|---|

**在线对话类**
| 模型/平台名称 | 免费权益详情 | 使用入口 | 免费有效期 | 使用限制 | 证据来源 |
|---|---|---|---|---|---|

**聚合平台类**
| 模型/平台名称 | 免费权益详情 | 使用入口 | 免费有效期 | 使用限制 | 证据来源 |
|---|---|---|---|---|---|

**证据来源**（可选区块）：逐条列出上文引用的官方/第三方来源 URL，便于核验与追溯。

字段说明：
- **模型/平台名称**：模型名或平台名
- **免费权益详情**：具体免费内容（如「DeepSeek R1 永久免费」「每日 150 条消息」）
- **使用入口**：官网 URL 或 API 接入方式
- **免费有效期**：永久免费 / 每日重置 / 具体截止日期
- **使用限制**：RPM、RPD、Token 上限、是否需注册等
- **证据来源**：该条信息的来源 URL（多个用逗号分隔）

输出约束：
- 只输出通过全部筛选条件的模型；禁止输出过期、付费、限时失效模型。
- 不编造不存在的免费资源。
- **官方源优先**：额度与有效期以官方文档为最终准；非官方来源（博客/测评）一律标注「需自行验证」。
- 信息不确定时标注「需自行验证」。

### JSON（output=json）
```json
[
  {
    "category": "api",
    "name": "模型/平台名称",
    "benefit": "免费权益详情",
    "entry": "使用入口",
    "validity": "永久免费 | 每日重置 | YYYY-MM-DD",
    "limits": "使用限制",
    "evidence": ["https://..."]
  }
]
```

## 注意事项
- 免费额度政策可能随时变动，提醒用户以官方最新说明为准。
- 部分免费 API 可能使用用户数据训练，需注明。
- 需注册的免费服务，注明注册方式（邮箱 / GitHub / 手机号）。
- 本 skill 不依赖任何调用方上下文或环境变量；所有行为由上述输入参数与依赖能力决定，可被任意 agent 解耦集成。
