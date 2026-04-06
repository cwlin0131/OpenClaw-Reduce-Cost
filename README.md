# OpenClaw Token 省錢手冊：新手必讀

**作者：CW ／ [Portaly](https://portaly.cc) 創辦人** — [portaly.cc/cwl](https://portaly.cc/cwl)

> 這是一份寫給 **OpenClaw 新手用戶** 的 Token 消耗管理指南。如果你剛開始用 OpenClaw 搭配 Claude API，卻發現帳單比想像中高很多，這篇會幫你搞懂原因，並提供實測有效的省錢策略。
> 可以將本文章的連結貼給 AI Agent，請他帶著你做！
> 
> 文中的 **「Bot」** 指的是你在 OpenClaw 上設定的機器人（你可能幫它取了自己的名字）。

**本文章節：**

1. [為什麼 API 費用這麼高？](#一為什麼-api-費用這麼高) — 費用暴增的根本原因：Context Window 機制
2. [固定 Context 開銷拆解](#二固定-context-開銷拆解) — 每次 API call 背後到底送了什麼進去
3. [解決方案](#三解決方案) — 六個省錢策略，從最有效的開始
4. [監控 Usage](#四監控-usage) — 怎麼追蹤你的花費
5. [常用 Command 列表](#五常用-command-列表) — 日常操作必備指令
6. [省錢效果估算](#六省錢效果估算) — 優化前後的成本差距

---

## 一、為什麼 API 費用這麼高？

### 問題發現

以下是一個真實的使用範例：某天光是做 OAuth 設定、測試發信、加行事曆、寫簡單 skill 規格，就花了 **$14 美金**（使用 Sonnet 4.6）。

查看 Anthropic Console 的 usage 數據後發現關鍵（以下為範例數據）：

- **Total tokens in：約 2,600 萬**
- **Total tokens out：約 15 萬**
- Input 是 output 的 **約 170 倍**

Bot 每次呼叫 API 都送進大量 context，但實際回覆很短。

### 根本原因：Context Window 的運作方式

每次在 Telegram 發一句話，API call 送進去的不只是那條訊息，而是：

```
[System Prompt + Tool Schema + Skill 定義 + Workspace 檔案]  ← 固定開銷
[第 1 輪] 你的訊息 + Bot 的回覆
[第 2 輪] 你的訊息 + Bot 的回覆
[第 3 輪] 你的訊息 + Bot 的回覆
...全部對話歷史，每次重新送一遍
```

所以即使只打了「查今天行程」幾個字（約 50 tokens），實際送進去的可能是幾萬甚至幾十萬 tokens。

---

## 二、固定 Context 開銷拆解

以下是一個範例拆解，實際數字會因你啟用的工具和 Workspace 設定而不同：

| 類別 | 內容 | 範例 Tokens | 能否調整 |
|------|------|------------|---------|
| **Tool Schemas** | 工具定義（Gmail、Calendar、訊息、瀏覽器等） | ~6,000 | ❌ 由 OpenClaw 自動生成，除非減少啟用工具 |
| **OpenClaw 說明文字** | 執行規則、格式說明、安全限制等 boilerplate | ~3,500 | ❌ OpenClaw 核心，不該改 |
| **Workspace 注入檔** | AGENTS.md / MEMORY.md / SOUL.md 等 | ~4,500 | ✅ 可精簡 |
| **Skills compact XML** | Skill 的摘要清單 | ~500 | ⚠️ 可調但效益不大 |
| **合計固定開銷（範例）** | | **~14,500** | |

> ⚠️ 以上數字為範例，你的實際固定開銷取決於啟用了多少工具、Workspace 檔案大小、安裝了幾個 Skill。可以透過讓 Bot 自查來取得你自己的數據。

### 對話歷史的累積效應

固定開銷之外，對話歷史是最大的變數。以固定開銷約 15,000 tokens 為例：

| 輪數 | 估計 Input Tokens | 說明 |
|------|------------------|------|
| 第 1 輪 | ~15,000 + 訊息 | 只有固定開銷 |
| 第 10 輪 | ~15,000 + 10 輪歷史 | 可能 20,000–30,000 |
| 第 30 輪（含 tool call） | ~15,000 + 所有歷史 | 可能 100,000+ |

**重點：** Tool use（查 Gmail、查 Calendar）的 JSON 回傳資料通常很長，一次 Calendar 查詢回傳可能就 2,000–5,000 tokens，這些全部存在對話歷史裡，每次都重送。

---

## 三、解決方案

### 方案 1：勤用 `/reset` 和 `/compact`（效果最大 ⭐）

省錢最有效的方式。

- **`/reset`** — 清空對話歷史，從零開始。做完一個任務就 reset。
- **`/compact`** — 壓縮對話歷史成摘要。任務還在進行但 context 太大時用。
- **`/new`** — 開一個全新的 session。

**使用時機：**

| 場景 | 動作 |
|------|------|
| 簡單查詢（查信、查行程） | 做完直接 `/reset` |
| 長任務進行中但 context 變肥 | `/compact` |
| 完全換一個主題 | `/new` 或 `/reset` |
| 同一主題持續討論 | 不用清，繼續聊 |

**注意：** Reset 會清掉對話記憶，但不影響持久資料（`~/.openclaw/` 裡的檔案、skill、設定）。重要結論記得在 reset 前讓 Bot 存到檔案裡。

### 方案 2：啟用 Prompt Cache（效果顯著 ⭐）

Cache 不會減少 token 數量，但會降低重複 context 的計費。

| 狀態 | 計費方式 |
|------|---------|
| 無 cache | 每次固定開銷按原價算 |
| Cache 命中 | 同樣的 tokens，按原價 **10%** 算 |

**OpenClaw 的 cache 設定：**

API Key 模式預設就有 5 分鐘的 short cache（寫入成本 1.25x），不用額外設定。

如果要升級到 1 小時的 long cache（寫入成本 2x），需要手動改設定檔。做法是在 Telegram 直接跟 Bot 說「幫我把 cacheRetention 改成 long」，Bot 會去修改 `~/.openclaw/openclaw.json`。也可以自己 SSH 進終端機用 `nano ~/.openclaw/openclaw.json` 手動加入以下設定：

```json
{
  "agents": {
    "defaults": {
      "models": {
        "anthropic/claude-sonnet-4-6": {
          "params": { "cacheRetention": "long" }
        },
        "anthropic/claude-haiku-4-5": {
          "params": { "cacheRetention": "long" }
        }
      }
    }
  }
}
```

改完重啟 Bot 生效。Long cache 的一小時效期是 Anthropic API 端自動管理的，不需要自己設定過期時間。

**怎麼選 short vs long：**

- 5 分鐘內會再次呼叫 → short 夠用（預設）
- 間隔可能超過 5 分鐘但在 1 小時內 → long 划算
- 一小時內 5–10 次互動的模式 → long 最適合

### 方案 3：精簡 Workspace 檔案（一次性優化）

OpenClaw 預設會產生多個 persona / memory 檔案，內容有不少重疊。你可以讓 Bot 自己分析哪些內容重複，然後合併精簡。

**合併範例（實際數字因人而異）：**

| 合併前 | 範例 Tokens |
|--------|-----------|
| AGENTS.md（舊） | ~2,000 |
| SOUL.md | ~400 |
| USER.md | ~150 |
| IDENTITY.md | ~150 |
| 合計 | ~2,700 |

| 合併後 | 範例 Tokens |
|--------|-----------|
| AGENTS.md（新，精簡版） | ~1,000 |

**做法：** 直接跟 Bot 說「把 SOUL.md、USER.md、IDENTITY.md 的內容去重後精簡合併進 AGENTS.md，其他檔案刪除」。Bot 會自己比對重複內容、保留獨特資訊、刪除冗餘描述。合併完再跟 Bot 說「進一步瘦身 AGENTS.md，刪掉不必要的段落」，可以再壓一輪。

合併的好處不只是省 token，更重要的是去除重複資訊、集中管理、未來只改一個檔案。

### 方案 4：Skill 管理策略

一般來說，Skill 定義本身的 token 佔比不大（範例中 6 個 Skill 合計只有約 500 tokens）。真正肥的是 Tool Schema（範例中約 6,000 tokens），這跟 Skill 開關無直接關係。

所以**少量 Skill 不需要特別關閉**。但如果未來 Skill 越裝越多，可以考慮用 `disable-model-invocation: true`，讓不常用的 Skill 變成手動呼叫（slash command 觸發），不注入 prompt。

**Skill 的兩種模式：**

| 模式 | 行為 | Token 消耗 |
|------|------|-----------|
| 常駐（預設） | Bot 自動判斷要不要用 | 每次佔 context |
| 手動（加 flag） | 用 /command 才觸發 | 不佔 context |

### 方案 5：模型搭配策略

| 模型 | Input / MTok | Output / MTok | 適用場景 |
|------|-------------|--------------|---------|
| **Haiku 4.5** | $1 | $5 | 日常執行：查信、摘要、行事曆、簡單回覆 |
| **Sonnet 4.5/4.6** | $3 | $15 | 寫 Skill、複雜分析、重要信件撰寫 |
| **Opus 4.5** | $5 | $25 | 一般不需要用到 |

> ⚠️ 以上價格為撰文時的參考，請以 [Anthropic 官方定價](https://www.anthropic.com/pricing) 為準。

**建議做法：**

- Bot 預設用 **Haiku**，處理日常執行任務
- 需要寫 Skill 或複雜判斷時，在 Telegram 用 `/models` 臨時切到 Sonnet
- 深度思考和策略討論放在 **Claude.ai**（訂閱制，不按 token 計費）

在 Telegram 裡直接用 `/models` 即時切換，不需要到終端機改設定。

### 方案 6：用 Gemini CLI 代理搜尋（進階）

Bot 需要上網查資料時，搜尋結果會整包灌進 context，一次搜尋可能就吃掉數千 tokens。解法是把「搜尋」這件事交給 Gemini CLI，搜完再把精簡結果餵回 Claude。

**為什麼用 Gemini 搜尋：**

- Google 搜尋本來就是 Gemini 的主場，查資料又快又準
- 搜尋過程不消耗 Claude 的 token
- Gemini 有免費額度，每天 1,000 次

**分工邏輯：搜尋歸 Gemini，思考歸 Claude。**

**基本設定流程：**

1. 在 OpenClaw 的主機上安裝 Gemini CLI：`npm install -g @google/gemini-cli`
2. 安裝完跑 `gemini --version` 確認成功
3. 透過 Skill 或腳本讓 Bot 在需要搜尋時呼叫 Gemini CLI
4. 限制 Gemini 只做三件事：找資料、列來源、摘錄原文
5. 搜尋結果回傳後，由 Claude 做分析和判斷

**降低幻覺的關鍵：** 不要讓 Gemini 做推論或總結，只讓它當「搜尋引擎 + 搬運工」，分析的工作留給 Claude。

> 💡 有訂閱 Google AI Pro 的話，Gemini 可以優先使用較強的模型，搜尋品質會更好。

---

## 四、監控 Usage

### Anthropic Console

直接查看：`https://console.anthropic.com/settings/usage`

可以看到：
- Total tokens in / out
- 按模型、按天的 token 用量圖表
- Cache creation / cache read 數據
- Rate limit 狀態和 tier

### Telegram 內建指令

- **`/usage`** — 可設定顯示模式：off / tokens / cost / full
- **`/status`** — 查看當前 session 的 context 大小、模型、連線狀態

建議一開始設 `/usage full`，每次回覆都會顯示 token 和費用，培養對成本的感覺。習慣後可以關掉。

---

## 五、常用 Command 列表

### 設定方式

在 Telegram 找 **@BotFather** → 發送 `/setcommands` → 選擇你的 bot → 貼上指令清單。設定完後在 bot 對話裡點左下角 Menu 按鈕即可快速選取。

### 推薦指令

```
models - 切換AI模型
reset - 清空這次對話，重新開始
new - 開一段新對話
compact - 壓縮對話歷史
status - 查看 context 現況
usage - 查看花了多少 token
think - 調整思考深度
```

### 各指令說明

| 指令 | 用途 | 使用時機 |
|------|------|---------|
| `/models` | 在 Haiku / Sonnet / Opus 之間切換 | 需要切換模型時 |
| `/reset` | 清空當前對話歷史 | 做完一個任務 |
| `/new` | 開一段全新對話 | 完全換主題 |
| `/compact` | 壓縮對話歷史成摘要 | 長任務進行中但 context 太肥 |
| `/status` | 顯示目前 context 大小、模型等 | 確認目前 context 大小 |
| `/usage` | 顯示 token 用量和費用 | 監控花費 |
| `/think` | 調整思考深度（off/minimal/low/medium/high） | low 日常，medium 複雜任務 |

---

## 六、省錢效果估算

以下是一個估算範例。假設每天跟 Bot 互動 50 次，固定開銷約 15,000 tokens/次：

| 優化方式 | 每日 Input Tokens（範例） | Haiku 每日成本（範例） |
|---------|------------------------|---------------------|
| 不優化（長對話不 reset） | ~5,000,000+ | ~$5+ |
| 勤 reset（每次約 15k） | ~750,000 | ~$0.75 |
| 勤 reset + cache 命中 | ~750,000（但 90% 按 0.1x 計費） | ~$0.10 |

> ⚠️ 以上為估算範例，實際數字取決於你的固定開銷大小、對話長度、模型選擇和 cache 命中率。

**結論：勤 reset + cache 可以讓每日成本降低數十倍，是最值得優先執行的兩個策略。**

---

*本文基於 OpenClaw + Claude API + Telegram 的實際使用經驗整理。文中所有 token 數據和費用皆為範例或估算，請以你自己的 Anthropic Console 數據為準。*

---

<a rel="license" href="https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh_TW"><img alt="創用 CC 授權條款" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a>

本著作由 [CW](https://portaly.cc/cwl) 製作，以[姓名標示-非商業性-相同方式分享 4.0 國際 (CC BY-NC-SA 4.0)](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh_TW) 授權條款釋出。

**您可以自由：** 分享、引用、改作本文，但必須標示原作者並附上原文連結，且不得用於商業目的。改作後的內容須以相同授權條款釋出。
