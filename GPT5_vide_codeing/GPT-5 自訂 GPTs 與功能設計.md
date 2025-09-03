# Command Module 模板 - 自訂 GPTs

## 1. 模板說明

這份模板用於設計自訂 GPTs 的最上層 **Command Module**，負責管理核心流程、指令解讀與路由分派，適用於 GPT-5 的物件導向架構。

---

## 2. 指令模組 (Command Module)

```yaml
command_module:
  name: "MainController"
  version: 1.0
  description: "自訂 GPTs 的核心指令模組，負責處理用戶請求與功能分配。"

  language: zh-TW  # 預設語言，可設為 zh-TW / en / ko
  voice_profile: "default_female"  # 語音設定，可自訂角色語氣

  inputs:
    - type: text
    - type: voice
    - type: image

  routes:
    voice_chat: "modules/voice_chat.py"
    live_analysis: "modules/live_analysis.py"
    photo_to_text: "modules/photo_to_text.py"
    memory_manager: "modules/memory_manager.py"
    api_bridge: "modules/api_bridge.py"

  knowledge_base:
    format: json
    location: "data/knowledge_base/"
    mode: isolated  # 每個子程式專屬資料庫，避免污染

  logging:
    enable: true
    level: debug
```

---

## 3. Command Module 流程

```mmd
flowchart TD
    A[使用者輸入] --> B[Command Module]
    B --> C{分析請求類型}
    C -->|語音聊天| D[Voice Chat Module]
    C -->|Live 功能| E[Live Analysis Module]
    C -->|拍照轉文字| F[Photo2Text Module]
    C -->|查詢記憶| G[Memory Manager]
    C -->|API 呼叫| H[API Bridge]
    F --> I[更新知識庫]
    G --> I[更新知識庫]
    H --> I[更新知識庫]
    I --> J[回傳結果]
```

---

## 4. API 整合設計

```yaml
api_bridge:
  description: "負責外部 API 整合，如圖片轉換、提醒、日曆等"
  endpoints:
    - name: "image_to_text"
      url: "https://api.example.com/ocr"
      method: POST
      headers:
        Content-Type: "application/json"
    - name: "calendar_reminder"
      url: "https://api.example.com/reminder"
      method: POST
      headers:
        Content-Type: "application/json"
```

---

## 5. 下一步建議

1. 定義 **JSON 知識庫結構**。
2. 實作 `voice_chat.py` 與 `photo_to_text.py` 子模組。
3. 整合 OCR API，實現 **拍照→文字描述→知識庫更新**。
4. 增加模擬長期記憶功能。

---

你想讓我接下來幫你設計 **JSON 知識庫結構** 嗎？

---

## 5. Command Module（指令模組）模板

> 這是一份可直接複用／擴充的模板，用於啟動你的三層物件導向架構：Command Module → Router → 子程式庫。

### 5.1 設計目標

* 明確宣告**角色／語氣**、**多語言**、**功能旗標**（Voice/Live/Memory/Reminders/Email）。
* 定義**路由契約**（如何組合子程式 1-2-3…）。
* 提供**反污染機制**（每次只載入必要的知識片段）。
* 規範**知識庫 Schema**（JSON/YAML）與**資料切片**策略。

### 5.2 Command Module 標頭（可貼在 System / Instruction 區）

```yaml
meta:
  name: "Assistant-Secretary"
  version: 1.0
  language: zh-TW            # zh-TW | en | ko | ja
  persona:
    display_name: "20 歲女生"
    style: "輕鬆、禮貌、口語化，但保持準確"
  capabilities:
    voice: true              # 已啟用語音回應
    live: true               # 已啟用 Live/視覺串流
    memory: soft             # none | soft | strict（soft=以外部庫模擬）
    reminders: external      # 透過外部 API
    email: external          # 透過外部 API

routing:
  # 定義高階功能與所需子程式組合（以 id 參照）
  features:
    smalltalk:
      chain: [chat.basic]
    photo_describe_and_store:
      chain: [vision.describe, kb.write]
    schedule_meeting:
      chain: [nlp.extract_time, calendar.create]
    write_email:
      chain: [nlp.outline, email.compose, email.send]

  # 反污染：每條 chain 僅能讀取其白名單資料域
  guards:
    scope_policy: strict
    allow_kb_namespaces:
      chat.basic:          ["kb/chat/persona_20f", "kb/chat/common"]
      vision.describe:     ["kb/vision/schemas", "kb/vision/labels"]
      kb.write:            ["kb/write/index"]
      nlp.extract_time:    ["kb/nlp/time_parsing"]
      calendar.create:     ["kb/calendar/schema"]
      email.compose:       ["kb/email/templates"]
      email.send:          ["kb/email/smtp"]

io_contracts:
  # 子程式 I/O 統一格式，方便路由拼裝
  message:
    input: { user_text: string, lang?: string }
    output: { reply_text: string, traces?: array }
  vision:
    input: { image_bytes?: base64, image_url?: string }
    output: { description: string, tags: string[] }
  memory:
    input: { namespace: string, key: string, value: any }
    output: { ok: boolean, id: string }
```

### 5.3 Router 參考實作（偽代碼）

```python
class Router:
    def __init__(self, registry, guards):
        self.registry = registry  # { name: callable }
        self.guards = guards      # { name: [namespaces] }

    def run_chain(self, chain, payload, feature_name):
        # 設定資料作用域（反污染）
        allowed_ns = self.guards.get(feature_name, [])
        scoped_kb = KnowledgeBase(namespaces=allowed_ns)
        ctx = {"kb": scoped_kb, "traces": []}
        x = payload
        for step in chain:
            fn = self.registry[step]
            x = fn(x, ctx)  # 每個子程式遵守 io_contracts
        return x
```

### 5.4 子程式最小介面（Python 範例）

```python
def chat_basic(input_msg, ctx):
    persona = ctx["kb"].read("kb/chat/persona_20f")
    reply = style_as_persona(input_msg["user_text"], persona)
    return {"reply_text": reply, "traces": ctx["traces"]}

def vision_describe(payload, ctx):
    desc = image_to_text(payload["image_bytes"] or payload["image_url"])  # 外部 API
    tags = tagger(desc)
    return {"description": desc, "tags": tags}

def kb_write(payload, ctx):
    ns = "kb/vision/room_index"
    key = hash_of(payload["description"])[:12]
    ctx["kb"].write(ns, key, payload)
    return {"ok": True, "id": f"{ns}:{key}"}
```

### 5.5 反污染機制（實務要點）

* **命名空間（namespace）隔離**：每個 feature/子程式僅可讀取其白名單命名空間。
* **會話工作集（working set）**：Router 在 chain 期間僅掛載必要片段。
* **檔級標籤**：在知識文件頂部加入 `scopes: [kb/chat/*]` 之類標籤；載入時檢查。
* **拼裝前清空**：每次執行 chain 前，清空非白名單緩存。

### 5.6 知識庫 Schema（JSON）

```json
{
  "_meta": {
    "namespace": "kb/vision/room_index",
    "schema_version": "1.0",
    "scopes": ["kb/vision/*"],
    "language": "zh-TW"
  },
  "items": [
    {
      "id": "living_room_fan_v1",
      "type": "object",
      "labels": ["風扇", "客廳"],
      "features": {
        "color": "白",
        "brand": "ABC",
        "position": "窗邊左側",
        "special_mark": "旋鈕上有藍色貼紙"
      },
      "description": "一台白色立式電風扇，位於客廳窗邊左側，旋鈕貼藍色貼紙。"
    }
  ]
}
```

### 5.7 圖片→文字→寫入知識庫（流程）

```mmd
sequenceDiagram
  participant U as 使用者
  participant L as Live/Camera
  participant V as vision.describe
  participant K as kb.write
  U->>L: 按下拍照
  L->>V: 傳影像資料
  V-->>L: 產生文字描述 + 標籤
  L->>K: 寫入 {namespace, key, description, tags}
  K-->>U: 回傳 ID/確認
```

### 5.8 多語言切換（標頭驅動）

```yaml
meta:
  language: en
persona:
  display_name: "20yo Girl"
style_rules:
  - "Use casual but respectful tone"
  - "Keep sentences short"
```

> Command Module 讀取 `meta.language` 後，決定輸出語言；資料層建議保持語言標註欄位。

### 5.9 提醒與郵件（外部 API 介面契約）

```yaml
apis:
  reminders:
    base_url: https://api.example.com/reminders
    endpoints:
      create:
        method: POST
        path: /create
        body: { title: string, time_iso: string, note?: string, user_id: string }
  email:
    base_url: https://api.example.com/email
    endpoints:
      send:
        method: POST
        path: /send
        body: { to: string, subject: string, html: string }
```

### 5.10 檔案大小與切片策略

* **Manifest + 分片**：

  * `kb/index.manifest.json`：列出所有分片（hash、namespace、語言、範圍）。
  * 分片檔例：`kb/vision/room_index.part-000.json`。
* **Router 載入規則**：依 feature 白名單 + 使用情境（房間/區域）載入指定分片。

### 5.11 角色語氣模板（20 歲女生）

```yaml
persona:
  display_name: "20 歲女生"
  talk_rules:
    - 用詞自然不做作，但避免俚語過多
    - 優先給出直接答案；必要時再補充細節
    - 提問時一次只問一件事
  safety:
    - 不臆測未授權資料
    - 無相關資料時，回覆「我不確定，需要查證」
```

### 5.12 驗收清單（Checklist）

* [ ] 不同 feature 下，非白名單命名空間不可讀取。
* [ ] 同一張圖片多次寫入會去重（以 hash 做 key）。
* [ ] 語言標頭切換後，回覆語言一致。
* [ ] Reminders/Email API 超時或錯誤有重試與回退機制。
* [ ] 日誌含 chain/子程式/命名空間與輸入長度，方便除錯。

---

如果你願意，我可以根據你現有的 3 個按鈕（3 個子程式）幫你**實際填好上述模板**，要不要把按鈕名稱與期望行為告訴我？
