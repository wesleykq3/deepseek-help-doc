# DeepSeek 使用操作文档

> 一份可直接上手的中文操作手册：覆盖**网页版、API 接入、模型与价格、服务状态监控、错误码排障与最佳实践**。
>
> 适用对象：开发者 / IT 运维 / 想快速接入 DeepSeek 的个人与团队
> 文档更新：2026-10-12

---

## 目录

- [一、官方链接速查](#一官方链接速查)
- [二、快速开始](#二快速开始)
- [三、API 基础信息](#三api-基础信息)
- [四、调用示例](#四调用示例)
- [五、模型与价格](#五模型与价格)
- [六、常用参数说明](#六常用参数说明)
- [七、思考模式（Thinking）](#七思考模式thinking)
- [八、服务状态与可用性监控](#八服务状态与可用性监控)
- [九、错误码与排障](#九错误码与排障)
- [十、限速与并发](#十限速与并发)
- [十一、安全与最佳实践](#十一安全与最佳实践)
- [十二、常见问题 FAQ](#十二常见问题-faq)
- [十三、参考链接汇总](#十三参考链接汇总)

---

## 一、官方链接速查

### 1. 三个核心链接（最常用）

| 用途 | 链接 | 说明 |
|---|---|---|
| 🌐 **DeepSeek 网页版** | <https://chat.deepseek.com> | 在线对话入口，无需 API，登录即用；支持深度思考 / 联网搜索等模式 |
| 📘 **API 文档（开发指引）** | <https://api-docs.deepseek.com/zh-cn/> | 官方 API 文档中文站，含快速开始、API 参考、指南、错误码 |
| 🟢 **服务状态（可用性）** | <https://status.deepseek.com/> | 官方状态页，实时查看 API 服务 / 网页对话服务的可用率与故障公告 |

> 备注：**网页版和 API 是两个独立服务**，在状态页上是两个独立组件，可能一个正常、一个异常。排查时先看清是哪一个。

### 2. 完整官方入口

| 项目 | 链接 | 说明 |
|---|---|---|
| 官网 | <https://www.deepseek.com> | 产品与公司主页 |
| 网页对话 | <https://chat.deepseek.com> | 消费级对话产品 |
| API 开放平台（控制台） | <https://platform.deepseek.com> | API Key 管理、用量、充值 |
| API Key 管理 | <https://platform.deepseek.com/api_keys> | 创建 / 删除 / 查看密钥 |
| 充值页面 | <https://platform.deepseek.com/top_up> | 余额充值 |
| API 文档（中文） | <https://api-docs.deepseek.com/zh-cn/> | 推荐中文读者 |
| API 文档（英文） | <https://api-docs.deepseek.com/> | 英文原站 |
| 服务状态页 | <https://status.deepseek.com/> | 可用性 / 故障公告 / 历史事件 |

### 3. 文档站内常用章节（直达）

| 章节 | 链接 |
|---|---|
| 首次调用 API（快速开始） | <https://api-docs.deepseek.com/zh-cn/> |
| 模型 & 价格 | <https://api-docs.deepseek.com/zh-cn/quick_start/pricing> |
| 错误码 | <https://api-docs.deepseek.com/zh-cn/quick_start/error_codes> |
| 限速与隔离 | <https://api-docs.deepseek.com/zh-cn/quick_start/rate_limit> |
| 思考模式 | <https://api-docs.deepseek.com/zh-cn/guides/thinking_mode> |
| Anthropic API 格式 | <https://api-docs.deepseek.com/zh-cn/guides/anthropic_api> |
| JSON Output | <https://api-docs.deepseek.com/zh-cn/guides/json_mode> |
| Tool Calls（函数调用） | <https://api-docs.deepseek.com/zh-cn/guides/tool_calls> |
| 上下文硬盘缓存 | <https://api-docs.deepseek.com/zh-cn/guides/kv_cache> |
| Agent 工具接入（Claude Code 等） | <https://api-docs.deepseek.com/zh-cn/quick_start/agent_integrations/claude_code> |

> 链接提示：文档站支持中英文切换，把 `/zh-cn/` 去掉即为英文页面。若某条链接失效，请从文档首页 <https://api-docs.deepseek.com/zh-cn/> 的侧边导航进入。

---

## 二、快速开始

### 步骤 1 · 注册账号

打开 <https://platform.deepseek.com>，用手机号或邮箱注册并登录开放平台。

### 步骤 2 · 充值（调用 API 需要余额）

新账号通常有少量赠送额度，用尽后需在 <https://platform.deepseek.com/top_up> 充值。
> 消费级网页版（chat.deepseek.com）免费使用，**API 调用按 token 计费**，两者余额不通用。

### 步骤 3 · 创建 API Key

1. 进入 <https://platform.deepseek.com/api_keys>
2. 点击「创建 API Key」，命名后生成
3. **复制并立即保存** —— 密钥只在创建时完整显示一次，关闭后无法再次查看

Key 形如 `sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`。

### 步骤 4 · 设为环境变量（推荐，避免硬编码）

**Linux / macOS（bash/zsh）**

```bash
export DEEPSEEK_API_KEY="sk-your-key-here"
```

**Windows PowerShell**

```powershell
$env:DEEPSEEK_API_KEY = "sk-your-key-here"
```

**Windows CMD**

```cmd
set DEEPSEEK_API_KEY=sk-your-key-here
```

> 永久生效请写入用户级环境变量或 `.env` 文件；`.env` 必须加入 `.gitignore`，**绝不要提交到仓库**。

### 步骤 5 · 首次调用

```bash
curl https://api.deepseek.com/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${DEEPSEEK_API_KEY}" \
  -d '{
        "model": "deepseek-flash",
        "messages": [
          {"role": "system", "content": "You are a helpful assistant."},
          {"role": "user", "content": "Hello!"}
        ],
        "stream": false
      }'
```

返回一段标准 OpenAI 兼容的 JSON 即为成功。

---

## 三、API 基础信息

| 项 | 值 |
|---|---|
| **Base URL（OpenAI 格式）** | `https://api.deepseek.com` |
| **Base URL（Anthropic 格式）** | `https://api.deepseek.com/anthropic` |
| **对话端点（OpenAI 兼容）** | `POST /chat/completions` |
| **模型列表端点** | `GET /models` |
| **认证方式** | HTTP Header：`Authorization: Bearer <API_KEY>` |
| **请求格式** | `Content-Type: application/json` |
| **兼容性** | 兼容 **OpenAI** 与 **Anthropic** 两套 API 格式，可直接复用官方 SDK |

### 为什么"OpenAI 兼容"很重要

DeepSeek 的接口与 OpenAI 格式保持一致，因此：

- 现有使用 OpenAI SDK 的代码，**只改 `base_url` 和 `model` 即可切换**；
- 所有"支持自定义 OpenAI 兼容端点"的框架/工具（LangChain、LlamaIndex、n8n、Dify、OpenClaw、各类客户端）都能直接接入；
- 可继续使用 OpenAI 生态的调试、日志、评测工具链。

---

## 四、调用示例

### 4.1 cURL

**非流式**

```bash
curl https://api.deepseek.com/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${DEEPSEEK_API_KEY}" \
  -d '{
        "model": "deepseek-flash",
        "messages": [
          {"role": "user", "content": "用三句话解释什么是向量数据库"}
        ],
        "stream": false
      }'
```

**流式（SSE，边生成边返回）**

```bash
curl https://api.deepseek.com/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${DEEPSEEK_API_KEY}" \
  -d '{
        "model": "deepseek-flash",
        "messages": [{"role": "user", "content": "写一首关于秋天的五言绝句"}],
        "stream": true
      }'
```

### 4.2 Python（OpenAI SDK，推荐）

安装：

```bash
pip install openai
```

代码：

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["DEEPSEEK_API_KEY"],
    base_url="https://api.deepseek.com",
)

# 非流式
resp = client.chat.completions.create(
    model="deepseek-flash",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Hello!"},
    ],
    stream=False,
)
print(resp.choices[0].message.content)

# 流式
stream = client.chat.completions.create(
    model="deepseek-flash",
    messages=[{"role": "user", "content": "讲个冷笑话"}],
    stream=True,
)
for chunk in stream:
    delta = chunk.choices[0].delta.content
    if delta:
        print(delta, end="", flush=True)
```

### 4.3 Python（Anthropic SDK）

```bash
pip install anthropic
```

```python
import os
from anthropic import Anthropic

client = Anthropic(
    api_key=os.environ["DEEPSEEK_API_KEY"],
    base_url="https://api.deepseek.com/anthropic",
)

msg = client.messages.create(
    model="deepseek-flash",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello!"}],
)
print(msg.content[0].text)
```

### 4.4 Node.js

```bash
npm install openai
```

```javascript
import OpenAI from "openai";

const client = new OpenAI({
  apiKey: process.env.DEEPSEEK_API_KEY,
  baseURL: "https://api.deepseek.com",
});

const resp = await client.chat.completions.create({
  model: "deepseek-flash",
  messages: [{ role: "user", content: "Hello!" }],
});
console.log(resp.choices[0].message.content);
```

### 4.5 健康检查 / 连通性自测

一句话确认"密钥对不对、网络通不通"：

```bash
curl -s https://api.deepseek.com/models \
  -H "Authorization: Bearer ${DEEPSEEK_API_KEY}" | head -c 400
```

- 返回模型列表 → 认证与网络均正常
- 返回 `401` → 密钥错误或未携带
- 连接超时 → 网络/代理问题（见第八节）

---

## 五、模型与价格

> **重点：模型与价格可能随官方调整而变化，请始终以官方定价页 <https://api-docs.deepseek.com/zh-cn/quick_start/pricing> 为准。**
> 下表为 2026-10-12 核对的信息，仅作参考。

| 模型名 | 版本 | 上下文 | 最大输出 | 并发限制 |
|---|---|---|---|---|
| `deepseek-flash` | DeepSeek-V4.1-Flash | 1M | 384K | 2500 |
| `deepseek-v4-pro` | DeepSeek-V4-Pro-0813 | 1M | 384K | 500 |

**价格（元 / 百万 tokens，空闲时段为高峰时段的一半）**

| 计费项 | deepseek-flash（空闲 / 高峰） | deepseek-v4-pro（空闲 / 高峰） |
|---|---|---|
| 输入 · 缓存命中 | 0.02 / 0.04 | 0.15 / 0.30 |
| 输入 · 缓存未命中 | 1 / 2 | 4.5 / 9.0 |
| 输出 | 4 / 8 | 13.5 / 27.0 |

**时段定义（北京时间）**

- **高峰时段**：周一至周五（不含法定节假日）**9:00–12:00、14:00–18:00**
- **空闲时段**：其余全部时间，含周末及法定节假日全天 → **价格减半**

**模型名兼容说明**

- 请始终使用 `deepseek-flash`；旧模型名（如 `deepseek-v4-flash`、`deepseek-v4-flash-vision-exp`）仍可调用，但对应模型已下线，请求会由 DeepSeek-V4.1-Flash 承接并按 Flash 价格计费。
- 更早的模型名（如 `deepseek-chat`、`deepseek-reasoner`）已逐步退役，**新项目请勿依赖**，以官方定价页列出的模型名为准。

**省钱要点**

1. 尽量安排在**空闲时段**跑批量任务，成本直接减半；
2. 利用**上下文硬盘缓存**：把稳定不变的前缀（系统提示、长文档）放在 messages 前部，缓存命中后输入价格可低至未命中的 1/50；
3. 用 `max_tokens` 限制输出长度，输出是单价最贵的一项。

---

## 六、常用参数说明

| 参数 | 类型 | 说明 |
|---|---|---|
| `model` | string | 模型名，如 `deepseek-flash` |
| `messages` | array | 对话消息列表，`role` 为 `system` / `user` / `assistant` / `tool` |
| `stream` | bool | `true` 时以 SSE 流式返回，适合实时输出 |
| `max_tokens` | int | 限制生成的最大 token 数 |
| `temperature` | float | 采样温度，越高越随机（0~2） |
| `top_p` | float | 核采样，与 temperature 二选一调 |
| `thinking` | object | 思考模式开关，如 `{"type": "enabled"}` |
| `reasoning_effort` | string | 思考投入强度（如 `low` / `medium` / `high`） |
| `response_format` | object | 设为 `{"type": "json_object"}` 可强制 JSON 输出（需在提示词中明确要求 JSON） |
| `tools` | array | 函数调用（Tool Calls）定义 |
| `stop` | string/array | 停止词 |

> 完整参数以官方 API 参考为准：<https://api-docs.deepseek.com/zh-cn/>

---

## 七、思考模式（Thinking）

DeepSeek 的模型支持**思考（Reasoning）**与**非思考**两种模式，可按任务难度切换：

| 模式 | 适用场景 | 特点 |
|---|---|---|
| **思考模式** | 数学、复杂推理、代码调试、多步规划 | 思维链更长，准确率更高，输出更慢、更贵 |
| **非思考模式** | 闲聊、摘要、简单改写、低延迟场景 | 响应快、成本低 |

调用时通过 `thinking` 字段控制（示例）：

```json
{
  "model": "deepseek-flash",
  "messages": [{"role": "user", "content": "证明 √2 是无理数"}],
  "thinking": {"type": "enabled"},
  "reasoning_effort": "high"
}
```

详细说明见：<https://api-docs.deepseek.com/zh-cn/guides/thinking_mode>

---

## 八、服务状态与可用性监控

### 8.1 人工查看

遇到"服务繁忙 / 无响应 / 调用失败"时，**先看状态页**再排查自己的代码：

👉 **<https://status.deepseek.com/>**

状态页提供：

- 各组件当前状态（**API 服务**、**网页对话服务** 等，独立显示）
- 可用率（uptime）百分比
- 历史故障事件与时间线（含中文说明）

### 8.2 程序化监控

如需把 DeepSeek 可用性接入自己的监控体系，可关注两处：

**① 状态页数据端点（Atlassian Statuspage 托管）**

状态页由 Statuspage 提供，通常暴露标准 JSON 端点，例如：

```bash
curl -s https://status.deepseek.com/api/v2/status.json
curl -s https://status.deepseek.com/api/v2/summary.json
```

> 该端点为第三方托管平台的通用接口，**并非官方承诺的正式 API**，字段与可用性可能变动，仅建议作趋势参考，关键判断仍以状态页页面为准。

**② API 自身健康检查**

最直接的方式是定时调用一个轻量端点并判断 HTTP 状态码：

```bash
# 返回 200 = API 及认证正常；401 = 密钥问题；429 = 触发限速；5xx = 服务端异常
curl -s -o /dev/null -w "%{http_code}\n" \
  https://api.deepseek.com/models \
  -H "Authorization: Bearer ${DEEPSEEK_API_KEY}"
```

可将此命令接入 Zabbix / Prometheus（blackbox_exporter）/ n8n 等监控工具，实现故障告警。

### 8.3 故障时的处理顺序

1. 打开 <https://status.deepseek.com/>，确认是否为官方**服务端问题**
2. 若是服务端问题 → **等待 + 重试**（指数退避），无其他办法
3. 若非服务端问题 → 依次排查：密钥（401/403）、余额（402）、参数（400/422）、限速（429）、本地网络/代理
4. 详见下一节错误码

---

## 九、错误码与排障

| 状态码 | 含义 | 原因 | 解决方法 |
|---|---|---|---|
| **400** | 格式错误 | 请求体格式错误 | 按返回的错误信息修改请求体 |
| **401** | 认证失败 | API Key 错误 | 检查密钥是否正确、是否过期；无 Key 请前往 <https://platform.deepseek.com/api_keys> 创建 |
| **402** | 余额不足 | 账户余额不足 | 前往充值页 <https://platform.deepseek.com/top_up> 充值 |
| **422** | 参数错误 | 请求体参数有误 | 按返回信息修正相关参数 |
| **429** | 请求速率达上限 | 超出 TPM / RPM 限额 | 合理规划请求速率，或进行重试退避 |
| **500** | 服务器故障 | 服务端内部故障 | 等待后重试；持续存在则联系官方 |
| **503** | 服务器繁忙 | 服务端负载过高 | 稍后重试 |

### 排障速查

```text
调用失败
 ├─ 401 / 403  → 密钥问题：检查环境变量、是否带 Bearer、是否已删除
 ├─ 402        → 余额不足：去充值页
 ├─ 429        → 触发限速：降低并发 / 加重试退避 / 升级并发额度
 ├─ 400 / 422  → 参数或 JSON 格式错误：核对 model 名与字段
 ├─ 500 / 503  → 服务端问题：看状态页，退避重试
 └─ 超时 / 连不上
      ├─ 先看状态页是否故障
      ├─ 检查本地网络、DNS、代理/VPN
      └─ 用 curl -v 打印握手过程定位卡在哪一步
```

**推荐的重试策略（指数退避 + 抖动）**：

```python
import time, random
def backoff(attempt, base=1.0, cap=30.0):
    delay = min(cap, base * (2 ** attempt))
    return delay * (0.5 + random.random() / 2)
```

仅对 **429 / 500 / 503 / 网络超时** 重试；对 **400 / 401 / 402 / 422** 重试没有意义，应先修正请求。

---

## 十、限速与并发

- **并发限制**：`deepseek-flash` 约 2500、`deepseek-v4-pro` 约 500（以官方为准）
- 触发限速会返回 **429**，应按上一节的退避策略处理
- 对大流量场景，建议：
  - 客户端做**请求队列 + 并发上限**控制
  - 使用**指数退避 + 抖动**重试
  - 批量任务尽量安排到**空闲时段**（见第五节，价格减半且负载更低）
- 详见：<https://api-docs.deepseek.com/zh-cn/quick_start/rate_limit>

---

## 十一、安全与最佳实践

### 密钥安全（最重要）

- ❌ **绝不把 API Key 写进代码或提交到 Git 仓库**
- ✅ 使用环境变量 / 密钥管理服务（Vault、KMS、CI 的 Secrets）
- ✅ `.gitignore` 必须包含 `.env`、`*.key` 等
- ✅ 若怀疑密钥泄露，**立即到 <https://platform.deepseek.com/api_keys> 删除并重建**
- ✅ 为不同项目/环境创建**独立密钥**，便于单独吊销与用量归因

### 工程实践

- **设置超时与重试上限**，避免请求悬挂拖垮服务
- **记录响应中的 `usage` 字段**（prompt/completion tokens），便于成本核算与监控
- **流式输出**用于交互式界面，**非流式**用于批处理，按场景选择
- 长文档/固定提示词前置，**提升缓存命中率**以降本
- 生产环境接入**可观测性**（延迟、错误率、token 消耗、429 比例）

### 合规提醒

- 遵守 DeepSeek 的服务条款与当地法律法规
- 不要向模型提交敏感个人信息、他人隐私或涉密数据
- 生成内容需自行审核，不得用于违法或侵权用途

---

## 十二、常见问题 FAQ

**Q1：网页版和 API 是一回事吗？**
不是。网页版 <https://chat.deepseek.com> 面向终端用户，免费；API 面向开发者，在 <https://platform.deepseek.com> 管理并按 token 计费。两者是**独立服务**，余额不通用，状态页上也分别监控。

**Q2：可以直接用 OpenAI 的代码吗？**
可以。DeepSeek 兼容 OpenAI 格式，只需把 `base_url` 改为 `https://api.deepseek.com`，`api_key` 换成 DeepSeek 的 Key，`model` 换成 DeepSeek 模型名即可。

**Q3：支持 Anthropic 格式吗？**
支持。Base URL 为 `https://api.deepseek.com/anthropic`，可直接用 Anthropic SDK 或支持该格式的工具（如 Claude Code）。

**Q4：怎么知道现在服务是否正常？**
打开 <https://status.deepseek.com/> 查看官方状态；或按第八节做程序化健康检查。

**Q5：为什么我的费用比预期高？**
常见原因：输出 token 太多（单价最高）、处于高峰时段（价格翻倍）、缓存未命中（输入按原价计）。可通过限制 `max_tokens`、调度到空闲时段、优化提示词前缀提升缓存命中来控制。

**Q6：报 429 怎么办？**
说明触发了速率/并发上限。降低并发、加入指数退避重试；若长期不足，需评估并发额度。

**Q7：旧模型名还能用吗？**
部分旧名仍可调用但会被路由到新模型，且计费按新模型价格。**新项目请直接使用定价页列出的当前模型名**。

**Q8：文档有没有英文版？**
有。把文档站路径中的 `/zh-cn/` 去掉即为英文：<https://api-docs.deepseek.com/>

---

## 十三、参考链接汇总

| 名称 | 链接 |
|---|---|
| DeepSeek 官网 | <https://www.deepseek.com> |
| DeepSeek 网页版 | <https://chat.deepseek.com> |
| API 开放平台 | <https://platform.deepseek.com> |
| API Key 管理 | <https://platform.deepseek.com/api_keys> |
| 充值 | <https://platform.deepseek.com/top_up> |
| **API 文档（中文）** | <https://api-docs.deepseek.com/zh-cn/> |
| API 文档（英文） | <https://api-docs.deepseek.com/> |
| 模型 & 价格 | <https://api-docs.deepseek.com/zh-cn/quick_start/pricing> |
| 错误码 | <https://api-docs.deepseek.com/zh-cn/quick_start/error_codes> |
| 限速与隔离 | <https://api-docs.deepseek.com/zh-cn/quick_start/rate_limit> |
| 思考模式 | <https://api-docs.deepseek.com/zh-cn/guides/thinking_mode> |
| Anthropic API | <https://api-docs.deepseek.com/zh-cn/guides/anthropic_api> |
| JSON Output | <https://api-docs.deepseek.com/zh-cn/guides/json_mode> |
| Tool Calls | <https://api-docs.deepseek.com/zh-cn/guides/tool_calls> |
| 上下文硬盘缓存 | <https://api-docs.deepseek.com/zh-cn/guides/kv_cache> |
| Agent 工具接入 | <https://api-docs.deepseek.com/zh-cn/quick_start/agent_integrations/claude_code> |
| **服务状态** | <https://status.deepseek.com/> |

---

## 免责声明

- 本文档为使用经验整理，**非 DeepSeek 官方文档**；模型名称、价格、限额、链接均可能随官方调整而变化，**请以官方文档与状态页为准**。
- 文档中的示例代码仅供学习参考，请自行评估并承担使用风险。
- 请遵守 DeepSeek 服务条款及相关法律法规。

---

> 如果本文档对你有帮助，欢迎 Star 支持 ⭐
