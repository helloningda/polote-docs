<div align="center">

# 🎬 polote.top 视频生成 API

**统一的 OpenAI 兼容异步视频生成接口**

支持文生视频 · 图生视频 · 多图参考 · 视频参考 · 音频参考 · 多模态参考

[![API](https://img.shields.io/badge/API-v1-blue)](https://polote.top)
[![Status](https://img.shields.io/badge/Status-Online-brightgreen)](https://polote.top)
[![OpenAI Compatible](https://img.shields.io/badge/OpenAI-Compatible-purple)](https://polote.top)

</div>

---

## 📖 目录

- [基础信息](#基础信息)
- [快速开始](#快速开始)
- [创建视频任务](#创建视频任务)
- [查询任务状态](#查询任务状态)
- [下载视频](#下载视频)
- [可用模型](#可用模型)
- [请求字段详解](#请求字段详解)
- [兼容字段](#兼容字段)
- [错误处理](#错误处理)
- [计费说明](#计费说明)
- [完整示例](#完整示例)

---

## 🔧 基础信息

| 项目 | 内容 |
|:---|:---|
| **Base URL** | `https://polote.top` |
| **认证方式** | `Authorization: Bearer <API_KEY>` |
| **创建任务** | `POST /v1/videos` |
| **查询任务** | `GET /v1/videos/{task_id}` |
| **下载视频** | `GET /v1/videos/{task_id}/content` |
| **查询模型列表** | `GET /v1/models` |
| **请求格式** | `application/json` |
| **任务类型** | 异步任务（提交后需轮询） |

> 💡 **提示**：所有素材（图片/视频/音频）必须是上游可访问的**公网 URL**。`seconds` 建议使用字符串形式，例如 `"15"`。

---

## 🚀 快速开始

### 1. 准备环境变量

```bash
export API_KEY="sk-xxxxxxxxxxxxxxxxxxxx"
export BASE_URL="https://polote.top"
```

### 2. 提交一个最简单的视频任务

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

### 3. 拿到 `task_id`，开始轮询

```bash
curl "$BASE_URL/v1/videos/task_xxx" \
  -H "Authorization: Bearer $API_KEY"
```

---

## 🎥 创建视频任务

### 场景 1：纯文生视频

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

### 场景 2：单图参考（图生视频）

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

### 场景 3：多图参考

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

### 场景 4：多模态全能参考

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

### 场景 5：首帧模式

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

### 场景 6：首尾帧模式

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

### 场景 7：1080p 高清输出

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

### 场景 8：Seedance 2.5 高清输出

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

### 场景 9：SD 2.5 Plus 系列（满血全参）

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

### 场景 10：SD 2.5 Pro 系列（原生过人脸）

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

---

## 🔍 查询任务状态

### 请求

```bash
curl "$BASE_URL/v1/videos/{task_id}" \
  -H "Authorization: Bearer $API_KEY"
```

### 状态说明

| 状态 | 含义 | 处理建议 |
|:---:|:---|:---|
| `queued` / `pending` / `not_start` | 已进入队列 | 继续轮询 |
| `processing` / `in_progress` / `running` | 生成中 | 继续轮询 |
| `completed` / `succeeded` / `success` | ✅ 已完成 | 获取 `video_url` 下载 |
| `failed` / `failure` / `error` | ❌ 失败 | 查看 `error.message` |

> 💡 **建议**：每 5-10 秒轮询一次，直到成功或失败。轮询 HTTP 错误不会被伪装成成功状态。

### 进行中响应

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

### 完成响应

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

### 失败响应

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

---

## 📥 下载视频

任务完成后，可以通过两种方式获取视频：

### 方式 1：直接使用 `video_url`（推荐）

任务完成响应里的 `video_url` 直接可用。

### 方式 2：通过 content 接口

```bash
curl -L "$BASE_URL/v1/videos/{task_id}/content" \
  -H "Authorization: Bearer $API_KEY" \
  -o result.mp4
```

> 💡 `/content` 接口会优先使用完成响应中的 `video_url`、`download_url`、`result_url` 或 `content_url`。

---

## 📊 可用模型

### 🎬 Seedance 2.0 系列

| 模型 | 分辨率 | 时长 | 参考素材 | 计费 |
|:---|:---:|:---:|:---:|:---:|
| `seedance2.0-480p-渠道1` | 480p | 4-15s | 9 图 / 3 视频 / 3 音频 | 按次 |
| `seedance2.0-720p-渠道1` | 720p | 4-15s | 9 图 / 3 视频 / 3 音频 | 按次 |
| `seedance2.0-hs-1080p` | 1080p | 4-15s | 9 图 / 3 视频 / 3 音频 | 按次 |

### 🎬 Seedance 2.5 系列

| 模型 | 分辨率 | 时长 | 参考素材 | 计费 |
|:---|:---:|:---:|:---:|:---:|
| `seedance2.5-480-渠道1` | 480p | 4 / 5 / 6 / 8 / 10 / 12 / 15 / 20 / 25 / 30s | 9 图 / 1 视频 / 1 音频 | 按秒 |
| `seedance2.5-720p-渠道1` | 720p | 4 / 5 / 6 / 8 / 10 / 12 / 15 / 20 / 25 / 30s | 9 图 / 1 视频 / 1 音频 | 按秒 |
| `seedance2.5-720p-渠道1-次` | 720p | 4-30s | 30 图 / 10 视频 / 10 音频 | 按次 |

### 🎞️ SD 2.5 Plus 系列

| 模型 | 分辨率 | 时长 | 参考素材 | 计费 |
|:---|:---:|:---:|:---:|:---:|
| `sd-2.5-480p-plus` | 480p | 4-30s | 30 图 / 10 视频 / 10 音频 | 按秒 |
| `sd-2.5-720p-plus` | 720p | 4-30s | 30 图 / 10 视频 / 10 音频 | 按秒 |

### 🎞️ SD 2.5 Pro 系列

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

---

## 📋 请求字段详解

### 标准字段

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

---

## 🔀 Sora 兼容字段

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

> 💡 具体价格见后台 **「模型管理」→「定价」** 页面。

---

## 🧪 完整示例

### Python 完整调用示例

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

### Node.js 调用示例

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

对于自定义模型名，推荐使用 **任务插件渠道** 并绑定本站配置的视频插件：

| 配置项 | 值 |
|:---|:---|
| **渠道类型** | 任务插件（Task Plugin，类型 61） |
| **插件** | `aicost-channel1` 等 |
| **上游地址** | 由管理员在渠道配置中填写 |
| **客户端地址** | 始终使用 `https://polote.top` |
| **模型列表** | 上游实际模型名，逗号分隔 |
| **渠道分组** | 包含令牌所属分组 |

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

---

<div align="center">

**📮 如有问题，请联系管理员**

Made with ❤️ by polote.top

</div>