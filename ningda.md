```markdown
# 🎬 polote.top 视频 & 图片生成 API

**统一的 OpenAI 兼容异步视频生成接口 & 图片生成接口**

支持文生视频 · 图生视频 · 多图参考 · 视频参考 · 音频参考 · 多模态参考 · 文生图 · 图生图 · 图片编辑 · 旧版 Banana 兼容

[![API](https://img.shields.io/badge/API-v1-blue)](https://polote.top)
[![Status](https://img.shields.io/badge/Status-Online-brightgreen)](https://polote.top)
[![OpenAI Compatible](https://img.shields.io/badge/OpenAI-Compatible-purple)](https://polote.top)

---

## 📖 目录

- [基础信息](#基础信息)
- [快速开始](#快速开始)
- [图片生成 API](#图片生成-api)
  - [图片接口总览](#图片接口总览)
  - [图片模型发现与名称](#图片模型发现与名称)
  - [文生图](#文生图)
  - [带参考图生成](#带参考图生成)
  - [图片编辑](#图片编辑)
  - [旧版 Banana 兼容](#旧版-banana-兼容)
  - [图片异步任务查询](#图片异步任务查询)
  - [图片请求字段详解](#图片请求字段详解)
  - [图片尺寸与 4K](#图片尺寸与-4k)
- [视频生成 API](#视频生成-api)
  - [创建视频任务](#创建视频任务)
  - [查询任务状态](#查询任务状态)
  - [下载视频](#下载视频)
  - [视频可用模型](#视频可用模型)
  - [视频请求字段详解](#视频请求字段详解)
  - [兼容字段](#兼容字段)
- [错误处理](#错误处理)
- [计费说明](#计费说明)
- [完整示例](#完整示例)
- [渠道配置说明](#渠道配置说明)
- [常见问题](#常见问题)

---

## 🔧 基础信息

| 项目 | 内容 |
|:---|:---|
| **Base URL** | `https://polote.top` |
| **认证方式** | `Authorization: Bearer <API_KEY>` |
| **创建视频任务** | `POST /v1/videos` |
| **查询视频任务** | `GET /v1/videos/{task_id}` |
| **下载视频** | `GET /v1/videos/{task_id}/content` |
| **文生图 / 部分图生图** | `POST /v1/images/generations` |
| **单图 / 多图编辑** | `POST /v1/images/edits` |
| **旧版 Banana 兼容** | `POST /v1/chat/completions` |
| **图片异步任务查询** | `POST /v1/images/result` |
| **查询模型列表** | `GET /v1/models` |
| **服务健康检查** | `GET /api/status` |
| **请求格式** | `application/json` / `multipart/form-data` |
| **任务类型** | 视频：异步任务（提交后需轮询）；图片：同步或异步任务 |

> 💡 **提示**：
> - 视频素材（图片/视频/音频）必须是上游可访问的**公网 URL**。
> - 视频 `seconds` 建议使用字符串形式，例如 `"15"`。
> - 图片推荐使用 `response_format: "b64_json"`，客户端不必依赖短时效图片 URL。
> - API Key 不应写入前端、公开仓库、日志或文档；客户端应从环境变量读取，例如 `API_KEY`。

---

## 🚀 快速开始

### 1. 准备环境变量

```bash
export API_KEY="sk-xxxxxxxxxxxxxxxxxxxx"
export BASE_URL="https://polote.top"
```

### 2. 提交一个最简单的图片任务

```bash
curl "$BASE_URL/v1/images/generations" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "一只戴黄色雨衣的橘猫，电影级柔和光线",
    "size": "2048x1152",
    "n": 1,
    "response_format": "b64_json"
  }'
```

处理图片响应：
- 若返回 `data[].b64_json` → 直接 base64 解码保存。
- 若返回 `data[].url` → 立即下载并保存，不要把临时 URL 当作永久资源。
- 若返回 `code: 0, data.id` → 这是异步任务，调用 `POST /v1/images/result` 轮询。

### 3. 提交一个最简单的视频任务

```bash
curl -X POST "$BASE_URL/v1/videos" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "seedance2.0-720p-渠道1",
    "prompt": "雨夜霓虹街道，镜头缓慢推进，电影感光影",
    "aspect_ratio": "16:9",
    "resolution": "720p",
    "seconds": "15"
  }'
```

### 4. 拿到视频 `task_id`，开始轮询

```bash
curl "$BASE_URL/v1/videos/task_xxx" \
  -H "Authorization: Bearer $API_KEY"
```

---

## 🖼️ 图片生成 API

### 图片接口总览

| 用途 | 方法 | 路径 | 请求格式 | 结果 |
|---|---|---|---|---|
| 文生图、部分图生图 | `POST` | `/v1/images/generations` | JSON | 同步图片或异步任务 |
| 单图/多图编辑 | `POST` | `/v1/images/edits` | `multipart/form-data` | 图片结果 |
| 旧版 Banana 兼容 | `POST` | `/v1/chat/completions` | JSON | 从 `choices[].message.content` 提取图片 |
| 异步图片任务查询 | `POST` | `/v1/images/result` | JSON | 查询并取得图片 URL |
| 模型发现 | `GET` | `/v1/models` | — | 当前可用模型列表 |
| 服务健康检查 | `GET` | `/api/status` | — | 平台状态 |

### 图片模型发现与名称

```bash
curl "$BASE_URL/v1/models" \
  -H "Authorization: Bearer $API_KEY"
```

模型名称分为两类：
1. **平台展示名**：例如 `gpt-image-2-Stable Plus`、`gpt-image-2.5-flare-体验`、`优质渠道-gpt-image-2`。
2. **上游实际名**：由平台渠道映射，例如 `gpt-image-2`、`gpt-image-2.5-flare`、`ph-gpt-image-2`。

调用时应使用 `/v1/models` 或管理端展示的模型名，不要自行猜测上游实际名。线上已验证过的图片展示模型包括：
- `gpt-image-2`
- `gpt-image-2-Stable Plus`
- `gpt-image-2-2/4K-优惠`
- `gpt-image-2-体验`
- `gpt-image-2.5-体验`
- `gpt-image-2.5-flare-体验`
- `优质渠道-gpt-image-2`
- `优质渠道-gpt-image-2.5`
- `优质渠道-gpt-image-2.5-flare`
- `优质渠道-gpt-image-2.5-sunburst`
- `优质渠道2-gpt-image-2K`
- `优质渠道2-gpt-image-4K`

Banana 类模型是否走图片接口取决于展示名和当前兼容配置。旧版客户端通常把以下模型走 Chat Completions：
- `Banana-2-香蕉2`
- `Banana-pro-香蕉pro`
- `nano-banana-2-稳定出图`
- `nano-banana-pro-稳定出图`

不要把 Banana 展示名直接强行发到 `/v1/images/generations`；历史成功/失败记录表明，部分 Banana 上游会返回“requires an image model”。

### 文生图

#### `POST /v1/images/generations`

#### 请求字段

```json
{
  "model": "gpt-image-2",
  "prompt": "一座雾中的东方古城，写实摄影风格",
  "size": "2048x1152",
  "n": 1,
  "response_format": "b64_json"
}
```

| 字段 | 类型 | 必填 | 说明 |
|---|---|---:|---|
| `model` | string | 是 | 平台当前可用的图片模型名 |
| `prompt` | string | 是 | 正向提示词 |
| `size` | string | 否 | 推荐使用 `1024x1024`、`2048x2048`、`2048x1152`、`1152x2048`；实际支持由渠道决定 |
| `n` | integer | 否 | 生成数量，默认 `1` |
| `response_format` | string | 否 | `b64_json` 或 `url`；推荐 `b64_json` |
| `image` | string[] | 否 | 带参考图时使用 data URI 数组 |
| `webHook` | string | 否 | 部分异步渠道兼容字段；不需要回调时可传 `-1` |
| `shutProgress` | boolean | 否 | 部分异步渠道兼容字段 |

`aspect_ratio` 和 `image_size` 是客户端常用的抽象参数，不一定是平台接口的强制字段。插件会将其转换成 `size`，例如：

| 客户端选择 | 发送的 `size` |
|---|---|
| `16:9` + `2K` | `2048x1152` |
| `9:16` + `2K` | `1152x2048` |
| `1:1` + `4K` | `4096x4096` |

#### 同步成功响应

```json
{
  "created": 1790000000,
  "data": [
    {
      "b64_json": "<base64图片数据>"
    }
  ]
}
```

或：

```json
{
  "created": 1790000000,
  "data": [
    {
      "url": "https://<临时图片地址>"
    }
  ]
}
```

如果返回 `url`，建议客户端立即下载；不要把临时上游 URL 当作永久资源保存。

#### 异步创建响应

部分渠道会返回任务 ID：

```json
{
  "code": 0,
  "data": {
    "id": "img_task_xxx"
  }
}
```

之后调用：

```bash
curl -X POST "$BASE_URL/v1/images/result" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"id":"img_task_xxx"}'
```

典型完成响应：

```json
{
  "code": 0,
  "data": {
    "status": "succeeded",
    "progress": 100,
    "results": [
      {"url": "https://<临时图片地址>"}
    ]
  }
}
```

状态通常包括 `running`、`succeeded`、`failed`。客户端建议 2 秒开始轮询，逐步退避到不超过 10 秒，并设置总超时（例如 300 秒）。

### 带参考图生成

对支持参考图的 generations 渠道，参考图放在 JSON 的 `image` 数组中。图片必须是真实的 data URI，不是本地路径：

```json
{
  "model": "gpt-image-2",
  "prompt": "保留人物脸部特征，将服装改成白色西装，商业摄影",
  "size": "2048x1152",
  "image": [
    "data:image/png;base64,<PNG_BASE64>",
    "data:image/jpeg;base64,<JPEG_BASE64>"
  ],
  "n": 1,
  "response_format": "b64_json"
}
```

注意：
- `image` 是数组，多个参考图按数组顺序传递。
- data URI 的 MIME 必须和真实文件一致，例如 `image/png`、`image/jpeg`。
- 不要把 Windows 路径、`file://` 路径或未公开的本地 URL 放进去。
- 参考图过大时，客户端应先压缩；不要把未经限制的超大 base64 写入日志。
- 线上插件会对请求日志中的 base64 和 URL 做脱敏，第三方客户端也应遵循这一点。

### 图片编辑

#### `POST /v1/images/edits`

适合需要上传参考图片、并由模型进行重绘/编辑的场景。请求使用 `multipart/form-data`，不要手动把整个请求拼成 JSON。

#### 单图示例

```bash
curl -X POST "$BASE_URL/v1/images/edits" \
  -H "Authorization: Bearer $API_KEY" \
  -F "model=gpt-image-2" \
  -F "prompt=把背景改成日落海边，保留主体不变" \
  -F "size=2048x1152" \
  -F "n=1" \
  -F "image=@./reference.png;type=image/png"
```

#### 多图示例

平台兼容插件对多参考图使用重复的 `image[]` 字段：

```bash
curl -X POST "$BASE_URL/v1/images/edits" \
  -H "Authorization: Bearer $API_KEY" \
  -F "model=gpt-image-2-Stable Plus" \
  -F "prompt=使用第一张图的人物、第二张图的场景进行合成" \
  -F "size=2048x1152" \
  -F "n=1" \
  -F "image[]=@./person.png;type=image/png" \
  -F "image[]=@./scene.png;type=image/png"
```

线上插件对参考图片会先转换为 PNG，再以 `image/png` 上传；这比直接把不确定的扩展名交给上游更稳定。

#### 响应

通常与 generations 相同：

```json
{
  "data": [
    {"b64_json": "<base64图片数据>"}
  ]
}
```

部分渠道也可能直接返回 `image/*` 二进制响应，客户端应同时支持：
- JSON + `data[].b64_json`
- JSON + `data[].url`
- `Content-Type: image/png`、`image/jpeg` 等二进制响应

### 旧版 Banana 兼容

#### `POST /v1/chat/completions`

仅在模型说明或当前渠道要求时使用。典型请求：

```json
{
  "model": "nano-banana-pro-稳定出图",
  "messages": [
    {
      "role": "user",
      "content": [
        {"type": "text", "text": "生成一张产品主视觉"},
        {
          "type": "image_url",
          "image_url": {"url": "https://<公开参考图地址>"}
        }
      ]
    }
  ],
  "n": 1
}
```

兼容客户端应从以下位置提取图片：
1. `choices[0].message.content` 中的 Markdown 图片链接：`![...](https://...)`
2. `choices[0].message.content` 中的纯图片 URL
3. `choices[0].message.content` 中的 `data:image/...;base64,...`
4. `data[0].b64_json` 或 `data[0].url`
5. 异步响应中的任务 ID，再调用 `/v1/images/result`

### 图片异步任务查询

#### `POST /v1/images/result`

如果创建请求返回了任务 ID：

```json
{
  "code": 0,
  "data": {
    "id": "img_task_xxx"
  }
}
```

则调用查询接口：

```bash
curl -X POST "$BASE_URL/v1/images/result" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"id":"img_task_xxx"}'
```

典型完成响应：

```json
{
  "code": 0,
  "data": {
    "status": "succeeded",
    "progress": 100,
    "results": [
      {"url": "https://<临时图片地址>"}
    ]
  }
}
```

| 状态 | 含义 | 处理建议 |
|---|---|---|
| `running` | 生成中 | 继续轮询 |
| `succeeded` | ✅ 已完成 | 获取 `results[].url` 下载 |
| `failed` | ❌ 失败 | 查看错误信息，不要盲目重试 |

> 💡 **建议**：2 秒开始轮询，逐步退避到不超过 10 秒，设置总超时（例如 300 秒）。

### 图片请求字段详解

| 字段 | 类型 | 必填 | 说明 |
|:---|:---:|:---:|:---|
| `model` | string | ✅ | 平台当前可用的图片模型名 |
| `prompt` | string | ✅ | 正向提示词 |
| `size` | string | ❌ | 推荐 `1024x1024`、`2048x2048`、`2048x1152`、`1152x2048`；实际支持由渠道决定 |
| `n` | integer | ❌ | 生成数量，默认 `1` |
| `response_format` | string | ❌ | `b64_json` 或 `url`；推荐 `b64_json` |
| `image` | string[] | ❌ | 带参考图时使用 data URI 数组 |
| `webHook` | string | ❌ | 部分异步渠道兼容字段；不需要回调时可传 `-1` |
| `shutProgress` | boolean | ❌ | 部分异步渠道兼容字段 |

### 图片尺寸与 4K

客户端可以把用户界面选项映射为 `size`：

```text
1K + 1:1  -> 1024x1024
2K + 16:9 -> 2048x1152
2K + 9:16 -> 1152x2048
4K + 1:1  -> 4096x4096
4K + 16:9 -> 4096x2304（仅在渠道支持时使用）
```

“4K”是目标尺寸，不代表所有渠道都会返回真正 4K。客户端必须检查最终图片的像素尺寸；插件成功日志曾验证过 `2048×1152`。如果渠道不支持某个比例/尺寸，应使用该模型支持的尺寸，不要仅在 prompt 中写“4K”来代替 `size`。

---

## 🎥 视频生成 API

### 创建视频任务

#### 场景 1：纯文生视频

```bash
curl -X POST "$BASE_URL/v1/videos" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "seedance2.0-720p-渠道1",
    "prompt": "一只猫在阳光下弹钢琴，镜头缓慢推进，电影感",
    "aspect_ratio": "16:9",
    "resolution": "720p",
    "seconds": "15"
  }'
```

#### 场景 2：单图参考（图生视频）

```bash
curl -X POST "$BASE_URL/v1/videos" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "seedance2.0-720p-渠道1",
    "prompt": "保持人物外貌和服装一致，自然缓慢走动",
    "image_url": "https://example.com/input-image.jpg",
    "aspect_ratio": "16:9",
    "resolution": "720p",
    "seconds": "15"
  }'
```

#### 场景 3：多图参考

`image_url` 是主参考图，额外参考图使用 `reference_image_urls`。

```bash
curl -X POST "$BASE_URL/v1/videos" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "seedance2.0-720p-渠道1",
    "prompt": "保持人物身份、妆容和服装一致，生成电影感产品展示片",
    "image_url": "https://example.com/main.jpg",
    "reference_image_urls": [
      "https://example.com/reference-1.jpg",
      "https://example.com/reference-2.jpg"
    ],
    "aspect_ratio": "9:16",
    "resolution": "720p",
    "seconds": "15"
  }'
```

#### 场景 4：多模态全能参考

图片 + 视频 + 音频 混合参考，生成电影感广告片。

```bash
curl -X POST "$BASE_URL/v1/videos" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "seedance2.5-720p-渠道1-次",
    "prompt": "参考图片中的人物与场景、参考视频中的运镜、参考音频中的节奏，生成电影感广告片",
    "image_url": "https://example.com/main.jpg",
    "reference_videos": ["https://example.com/camera-motion.mp4"],
    "audio_urls": ["https://example.com/rhythm.mp3"],
    "aspect_ratio": "16:9",
    "resolution": "720p",
    "seconds": "15"
  }'
```

#### 场景 5：首帧模式

```bash
curl -X POST "$BASE_URL/v1/videos" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "seedance2.0-720p-渠道1",
    "prompt": "从这个画面开始，镜头缓慢推进",
    "image_url": "https://example.com/first-frame.jpg",
    "aspect_ratio": "16:9",
    "resolution": "720p",
    "seconds": "15"
  }'
```

#### 场景 6：首尾帧模式

```bash
curl -X POST "$BASE_URL/v1/videos" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "seedance2.0-720p-渠道1",
    "prompt": "从第一张画面平滑过渡到第二张画面",
    "reference_image_urls": [
      "https://example.com/start-frame.jpg",
      "https://example.com/end-frame.jpg"
    ],
    "aspect_ratio": "16:9",
    "resolution": "720p",
    "seconds": "15"
  }'
```

#### 场景 7：1080p 高清输出

```bash
curl -X POST "$BASE_URL/v1/videos" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "seedance2.0-hs-1080p",
    "prompt": "雨夜霓虹街道，镜头缓慢推进，电影感光影",
    "image_url": "https://example.com/input.jpg",
    "aspect_ratio": "16:9",
    "resolution": "1080p",
    "seconds": "15"
  }'
```

#### 场景 8：Seedance 2.5 高清输出

```bash
curl -X POST "$BASE_URL/v1/videos" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "seedance2.5-720p-渠道1",
    "prompt": "让参考图中的人物在街头自然走动，电影感镜头",
    "image_url": "https://example.com/ref.jpg",
    "aspect_ratio": "16:9",
    "resolution": "720p",
    "seconds": "15"
  }'
```

#### 场景 9：SD 2.5 Plus 系列（满血全参）

```bash
curl -X POST "$BASE_URL/v1/videos" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "sd-2.5-720p-plus",
    "prompt": "A fashion model walking through a minimalist gallery, smooth dolly shot",
    "image_url": "https://example.com/main.jpg",
    "aspect_ratio": "16:9",
    "resolution": "720p",
    "seconds": "15"
  }'
```

#### 场景 10：SD 2.5 Pro 系列（原生过人脸）

```bash
curl -X POST "$BASE_URL/v1/videos" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "sd-2.5-720p-pro",
    "prompt": "A futuristic city at sunset, cinematic camera movement",
    "image_url": "https://example.com/main.jpg",
    "audio_urls": ["https://example.com/rhythm.mp3"],
    "aspect_ratio": "16:9",
    "resolution": "720p",
    "seconds": "10"
  }'
```

### 查询任务状态

#### 请求

```bash
curl "$BASE_URL/v1/videos/{task_id}" \
  -H "Authorization: Bearer $API_KEY"
```

#### 状态说明

| 状态 | 含义 | 处理建议 |
|:---:|:---|:---|
| `queued` / `pending` / `not_start` | 已进入队列 | 继续轮询 |
| `processing` / `in_progress` / `running` | 生成中 | 继续轮询 |
| `completed` / `succeeded` / `success` | ✅ 已完成 | 获取 `video_url` 下载 |
| `failed` / `failure` / `error` | ❌ 失败 | 查看 `error.message` |

> 💡 **建议**：每 5-10 秒轮询一次，直到成功或失败。轮询 HTTP 错误不会被伪装成成功状态。

#### 进行中响应

```json
{
  "id": "task_xxx",
  "task_id": "task_xxx",
  "object": "video",
  "model": "seedance2.0-720p-渠道1",
  "status": "processing",
  "progress": 45,
  "progress_text": "45%",
  "created_at": 1776936222,
  "seconds": "15",
  "size": "1280x720"
}
```

#### 完成响应

```json
{
  "id": "task_xxx",
  "task_id": "task_xxx",
  "object": "video",
  "model": "seedance2.0-720p-渠道1",
  "status": "completed",
  "progress": 100,
  "created_at": 1776936222,
  "seconds": "15",
  "size": "1280x720",
  "video_url": "https://polote.top/v1/videos/task_xxx/content"
}
```

#### 失败响应

```json
{
  "id": "task_xxx",
  "task_id": "task_xxx",
  "object": "video",
  "status": "failed",
  "progress": 0,
  "error": {
    "code": "invalid_request_error",
    "message": "提示词包含敏感违规内容或参数格式错误"
  }
}
```

### 下载视频

任务完成后，可以通过两种方式获取视频：

#### 方式 1：直接使用 `video_url`（推荐）

任务完成响应里的 `video_url` 直接可用。

#### 方式 2：通过 content 接口

```bash
curl -L "$BASE_URL/v1/videos/{task_id}/content" \
  -H "Authorization: Bearer $API_KEY" \
  -o result.mp4
```

> 💡 `/content` 接口会优先使用完成响应中的 `video_url`、`download_url`、`result_url` 或 `content_url`。

### 视频可用模型

#### 🎬 Seedance 2.0 系列

| 模型 | 分辨率 | 时长 | 参考素材 | 计费 |
|:---|:---:|:---:|:---:|:---:|
| `seedance2.0-480p-渠道1` | 480p | 4-15s | 9 图 / 3 视频 / 3 音频 | 按次 |
| `seedance2.0-720p-渠道1` | 720p | 4-15s | 9 图 / 3 视频 / 3 音频 | 按次 |
| `seedance2.0-hs-1080p` | 1080p | 4-15s | 9 图 / 3 视频 / 3 音频 | 按次 |

#### 🎬 Seedance 2.5 系列

| 模型 | 分辨率 | 时长 | 参考素材 | 计费 |
|:---|:---:|:---:|:---:|:---:|
| `seedance2.5-480-渠道1` | 480p | 4 / 5 / 6 / 8 / 10 / 12 / 15 / 20 / 25 / 30s | 9 图 / 1 视频 / 1 音频 | 按秒 |
| `seedance2.5-720p-渠道1` | 720p | 4 / 5 / 6 / 8 / 10 / 12 / 15 / 20 / 25 / 30s | 9 图 / 1 视频 / 1 音频 | 按秒 |
| `seedance2.5-720p-渠道1-次` | 720p | 4-30s | 30 图 / 10 视频 / 10 音频 | 按次 |

#### 🎞️ SD 2.5 Plus 系列

| 模型 | 分辨率 | 时长 | 参考素材 | 计费 |
|:---|:---:|:---:|:---:|:---:|
| `sd-2.5-480p-plus` | 480p | 4-30s | 30 图 / 10 视频 / 10 音频 | 按秒 |
| `sd-2.5-720p-plus` | 720p | 4-30s | 30 图 / 10 视频 / 10 音频 | 按秒 |

#### 🎞️ SD 2.5 Pro 系列

| 模型 | 分辨率 | 时长 | 参考素材 | 计费 |
|:---|:---:|:---:|:---:|:---:|
| `sd-2.5-480p-pro` | 480p | 5-30s | 30 图 / 0 视频 / 10 音频 | 按秒 |
| `sd-2.5-720p-pro` | 720p | 5-30s | 30 图 / 0 视频 / 10 音频 | 按秒 |

> 📌 **时长说明**：
> - `seedance2.5-480-渠道1` 和 `seedance2.5-720p-渠道1` 仅支持**固定时长**：4 / 5 / 6 / 8 / 10 / 12 / 15 / 20 / 25 / 30 秒，不支持其他值。
> - 其他模型支持**区间内任意整数**。
>
> 📌 **分辨率锁定**：模型名包含 `480p` / `720p` / `1080p` 时，网关会锁定对应分辨率，客户端传入的冲突值不会覆盖。
>
> 📌 **模型列表动态变化**：实际可用模型以 `GET /v1/models` 返回为准。

### 视频请求字段详解

#### 标准字段

| 字段 | 类型 | 必填 | 说明 |
|:---|:---:|:---:|:---|
| `model` | string | ✅ | 模型名称 |
| `prompt` | string | ✅ | 视频生成提示词 |
| `seconds` | string / number | ✅ | 视频时长（秒），建议用字符串 |
| `aspect_ratio` | string | ❌ | 画面比例：`16:9` / `9:16` / `1:1` / `3:4` / `4:3` / `21:9` |
| `resolution` | string | ❌ | 分辨率：`480p` / `720p` / `1080p` |
| `size` | string | ❌ | 尺寸，如 `1280x720`（会自动推导比例） |
| `image_url` | string | ❌ | 主参考图（公网 URL） |
| `reference_image_urls` | string[] | ❌ | 额外参考图数组 |
| `reference_video` | string | ❌ | 单个参考视频 URL |
| `reference_videos` | string[] | ❌ | 多个参考视频 URL |
| `audio_url` | string | ❌ | 单个参考音频 URL |
| `audio_urls` | string[] | ❌ | 多个参考音频 URL |

> ⚠️ **重要**：`seconds` 在 NewAPI 内部被定义为 **string 类型**。传数字（如 `10`）可能报 `json: cannot unmarshal number`。**建议统一使用字符串**，例如 `"10"`。

### 兼容字段（自动转换）

| 标准字段 | 兼容别名 |
|:---|:---|
| `seconds` | `duration` |
| `aspect_ratio` | `ratio` |
| `image_url` | `image`、`images`、`image_urls`、`input_reference` |
| `reference_image_urls` | `reference_images` |
| `reference_video` | `video_url`、`input_video` |
| `reference_videos` | `videos`、`video_urls` |
| `audio_url` | `input_audio` |
| `audio_urls` | `audios` |

### 🔀 Sora 兼容字段

本站兼容标准 Sora 风格的 `size` 和 `input_reference`：

```json
{
  "model": "seedance2.0-720p-渠道1",
  "prompt": "保持人物外貌一致，自然缓慢走动",
  "seconds": "10",
  "size": "1280x720",
  "input_reference": { "image_url": "https://example.com/person.jpg" },
  "reference_images": ["https://example.com/reference.jpg"]
}
```

**转换规则**：
- `size` 按宽高比转换为 `aspect_ratio`（`1280x720` → `16:9`）
- `input_reference` 支持字符串或 `{ "image_url": "..." }` 对象
- 第一张图进 `image_url`，后续图进 `reference_image_urls`

---

## ⚠️ 错误处理

### 标准错误结构

```json
{
  "error": {
    "message": "错误说明",
    "type": "invalid_request_error",
    "param": "model",
    "code": null
  }
}
```

### 常见 HTTP 状态码

| 状态码 | 含义 | 客户端处理 |
|---:|---|---|
| `400` | 参数、模型或请求格式错误 | 修正请求，不要盲目重试 |
| `401` | API Key 无效或缺失 | 检查 Bearer Token |
| `403` | 余额不足或无权访问 | 检查账户余额或令牌权限 |
| `404` | 路径或任务不存在 | 检查接口路径和任务 ID |
| `413` | 请求体过大 | 压缩图片/减少参考图 |
| `429` | 上游限流或并发不足 | 退避重试，避免立即循环 |
| `500` | 平台内部错误 | 记录 request id 后有限重试 |
| `502/503` | 上游暂不可用/无可用渠道 | 延迟重试或换模型 |
| `504` | 上游超时 | 增大客户端超时或轮询异步任务 |

### 常见错误码

| 错误码 | 说明 | 处理建议 |
|:---|:---|:---|
| `401` | API Key 无效或未携带 | 检查 `Authorization` 头 |
| `403` | 余额不足或无权访问 | 检查账户余额或令牌权限 |
| `model_not_found` | 模型不存在或未配置 | 检查模型名和分组 |
| `invalid_request` | 请求参数错误 | 检查 JSON 格式和字段类型 |
| `model_price_error` | 模型未配置价格 | 联系管理员配置价格 |
| `artifact_gone` | 视频已过期或地址失效 | **重新提交任务**生成新视频 |

> **`artifact_gone` 说明**：上游返回的视频链接有效期有限，过期后无法再下载。请**重新提交任务**生成新的视频，并尽快下载保存到本地。

### 已知图片错误示例

```json
{"error":{"message":"images endpoint requires an image model, got \"nano-banana-2\"","type":"invalid_request_error"}}
```

这表示模型与 `/v1/images/generations` 不匹配，应改用该模型要求的 Chat Completions 兼容路径或换用图片模型。

### 素材要求

| 类型 | 支持格式 | 说明 |
|:---|:---|:---|
| 图片 | JPG / PNG / WebP | 公网 URL，单张 < 20MB |
| 视频 | MP4 / MOV / WebM | 公网 URL，单条 2-15 秒 |
| 音频 | MP3 / WAV / M4A | 公网 URL，单条 2-15 秒 |

---

## 💰 计费说明

本站视频模型采用**按次计费**或**按秒计费**，具体单价请查看模型价格页。

- **按次计费**：每次任务生成成功扣一次固定费用
- **按秒计费**：按视频实际秒数 × 每秒单价扣费
- **任务失败不扣费**：失败任务会自动退还配额
- **余额不足**：提交任务前请确保账户余额充足

图片模型通常按次/按张计费，具体单价请查看模型价格页。

> 💡 具体价格见后台 **「模型管理」→「定价」** 页面。

---

## 🧪 完整示例

### Python 图片调用示例

```python
import base64
import os
import requests

BASE_URL = "https://polote.top"
API_KEY = os.environ["API_KEY"]

payload = {
    "model": "gpt-image-2",
    "prompt": "一只戴黄色雨衣的橘猫，电影级柔和光线",
    "size": "2048x1152",
    "n": 1,
    "response_format": "b64_json",
}

resp = requests.post(
    f"{BASE_URL}/v1/images/generations",
    headers={"Authorization": f"Bearer {API_KEY}"},
    json=payload,
    timeout=300,
)
resp.raise_for_status()
data = resp.json()
item = data["data"][0]

if item.get("b64_json"):
    image_bytes = base64.b64decode(item["b64_json"])
elif item.get("url"):
    image_bytes = requests.get(item["url"], timeout=300).content
else:
    raise RuntimeError(f"无法识别图片响应：{data}")

with open("output.png", "wb") as f:
    f.write(image_bytes)
```

### Python 视频完整调用示例

```python
import requests
import time

API_KEY = "sk-xxxxxxxxxxxx"
BASE_URL = "https://polote.top"
HEADERS = {
    "Authorization": f"Bearer {API_KEY}",
    "Content-Type": "application/json"
}

# 1. 创建任务
payload = {
    "model": "seedance2.0-720p-渠道1",
    "prompt": "一只猫在阳光下弹钢琴，镜头缓慢推进",
    "aspect_ratio": "16:9",
    "resolution": "720p",
    "seconds": "15"
}
resp = requests.post(f"{BASE_URL}/v1/videos", headers=HEADERS, json=payload)
task = resp.json()
task_id = task["task_id"]
print(f"任务已提交: {task_id}")

# 2. 轮询任务
while True:
    time.sleep(5)
    r = requests.get(f"{BASE_URL}/v1/videos/{task_id}", headers=HEADERS)
    data = r.json()
    status = data.get("status")
    print(f"状态: {status}, 进度: {data.get('progress', 0)}%")

    if status in ("completed", "succeeded"):
        video_url = data.get("video_url") or data.get("result_url")
        print(f"✅ 视频生成完成: {video_url}")
        break
    elif status == "failed":
        print(f"❌ 任务失败: {data.get('error')}")
        break
```

### Node.js 视频调用示例

```javascript
const API_KEY = "sk-xxxxxxxxxxxx";
const BASE_URL = "https://polote.top";

async function generateVideo() {
  // 1. 创建任务
  const createResp = await fetch(`${BASE_URL}/v1/videos`, {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${API_KEY}`,
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      model: "seedance2.0-720p-渠道1",
      prompt: "一只猫在阳光下弹钢琴",
      aspect_ratio: "16:9",
      resolution: "720p",
      seconds: "15"
    })
  });
  const task = await createResp.json();
  const taskId = task.task_id;
  console.log(`任务已提交: ${taskId}`);

  // 2. 轮询
  while (true) {
    await new Promise(r => setTimeout(r, 5000));
    const statusResp = await fetch(`${BASE_URL}/v1/videos/${taskId}`, {
      headers: { "Authorization": `Bearer ${API_KEY}` }
    });
    const data = await statusResp.json();
    console.log(`状态: ${data.status}, 进度: ${data.progress || 0}%`);

    if (["completed", "succeeded"].includes(data.status)) {
      console.log(`✅ 视频完成: ${data.video_url || data.result_url}`);
      return;
    }
    if (data.status === "failed") {
      throw new Error(`任务失败: ${JSON.stringify(data.error)}`);
    }
  }
}

generateVideo().catch(console.error);
```

---

## 📌 渠道配置说明


图片模型同样通过渠道映射到实际上游模型。客户端始终调用 `https://polote.top`，不要硬编码上游地址。

> 📌 **不要硬编码模型列表**：实际可用模型以 `GET /v1/models` 返回为准。

---

## 📮 常见问题

### Q1: `seconds` 为什么要用字符串？

NewAPI 的内部字段强类型为 string，传数字可能报 `json: cannot unmarshal number` 错误。建议始终用 `"15"` 这样的字符串。

### Q2: 素材 URL 有什么要求？

必须是**公网可访问的 HTTPS URL**，服务端能直接读取。不支持本地文件上传、data URI、Base64。

### Q3: 视频什么时候过期？

上游返回的视频链接**有效期有限**，建议生成成功后**尽快下载**保存到本地。若链接已过期（返回 `artifact_gone`），请重新提交任务。

### Q4: 如何查看实际可用模型？

```bash
curl "$BASE_URL/v1/models" -H "Authorization: Bearer $API_KEY"
```

### Q5: `seedance2.5-480-渠道1` 的时长为什么是固定值？

该模型仅支持**离散时长**：4 / 5 / 6 / 8 / 10 / 12 / 15 / 20 / 25 / 30 秒。传入其他值（如 `"7"`）会报错。请从前述固定值中选择。

### Q6: 图片 `response_format` 应该用 `b64_json` 还是 `url`？

推荐 `b64_json`。这样客户端不必依赖短时效图片 URL。如果平台或具体渠道只返回 URL，客户端应立即下载并自行保存。

### Q7: 图片参考图怎么传？

- **Generations 接口**：`image` 数组，每个元素是真实 data URI，如 `data:image/png;base64,...`
- **Edits 接口**：`multipart/form-data`，单图用 `image`，多图用重复的 `image[]`

不要传本地路径、`file://` 路径或未公开的本地 URL。

### Q8: 图片异步任务怎么处理？

1. 创建请求返回 `data.id` → 这是任务 ID
2. 调用 `POST /v1/images/result`，传入 `{"id": "img_task_xxx"}`
3. 轮询直到 `status` 变为 `succeeded` 或 `failed`
4. 从 `results[].url` 获取图片

### Q9: 为什么 Banana 模型不能走 `/v1/images/generations`？

部分 Banana 上游会返回 `requires an image model` 错误。Banana 类模型是否走图片接口取决于展示名和当前兼容配置，历史成功/失败记录表明应走 Chat Completions 路径。

### Q10: 如何确认图片 2K/4K 是否真实生效？

记录最终图片的宽高，与请求的 `size` 进行对比。4K 是目标尺寸，不代表所有渠道都会返回真正 4K。

---

**📮 如有问题，请联系管理员**

Made with ❤️ by polote.top
```
