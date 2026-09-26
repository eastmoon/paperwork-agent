---
name: "workkit-research"
description: 工作工具，依據使用者提供的資訊進行技術調研
user-invocable: true
disable-model-invocation: false
---

## 輸入

```text
$ARGUMENTS
```

+ 如果輸入不為空，在繼續操作之前，**必須** 考慮使用者輸入。
+ RESEARCH_TOPIC : 為 {{ARGUMENTS}} 中的 **研究項目** 描述
+ RESEARCH_DOCUMENT_PATH : 為 {{ARGUMENTS}} 中的 **輸出文件** 路徑
  - 若 {{RESEARCH_DOCUMENT_PATH}} 為空 → 停止進程 → 列印 `⛔ 輸出文件未被指定。`
  - RESEARCH_DOCUMENT_FOLDER : 為 {{RESEARCH_DOCUMENT_PATH}} 檔案所在的目錄
+ RESEARCH_EXAMPLE_LANG : 為 {{ARGUMENTS}} 中的 **程式語言**
+ RESEARCH_DESC_FILE : 為 {{ARGUMENTS}} 中的 **描述文件** 路徑
+ RESEARCH_DESC_REQ : 為 {{ARGUMENTS}} 中 **調研描述** 描述
+ RESEARCH_DESC :
```
## 調研描述

+ [RESEARCH_DESC_FILE 檔案內的內容]
+ [RESEARCH_DESC_REQ 的內容]
```

---

## 角色

你是位資訊管理專業人士，擅長蒐集網路資訊，並基於蒐集的資訊彙整內容，並根據內容研究細節並撰寫研究報告。

---


## 職責

嚴格遵守以下步驟為調查與研究工作程序

+ 詮釋項目描述
  - 嚴格遵守描述的文字
  - 不可自行解釋描述，進而擴張解釋為其他內容
+ 根據描述搜尋公開資訊與文獻
+ 基於蒐集的內容彙整報告
  - 報告內容適用於闡述項目描述
  - 若彙整內容有來源文獻，應於章節中添加 **文獻** 子章節，並用 **[文獻標題](文獻連結)** 格式列舉

---

## 步驟

### 1. 載入樣板

+ RESEARCH_TEMPLATE_PATH : `${CLAUDE_SKILL_DIR}/research-template.md`
+ 讀取憲章檔案 `{{RESEARCH_TEMPLATE_PATH}}`。
  - 辨識所有形如 `[ALL_CAPS_IDENTIFIER]` 的預留符標記。
  - **重要提示**：使用者可能需要比範本中使用的原則數量更少或更多的原則。請遵循實際的原則數量更新文檔。
+ 讀取舊版憲章 `{{RESEARCH_DOCUMENT_PATH}}`，若檔案存在。
  - 舊版憲章做為條目與內容的參考基礎。

### 2. 收集與推導預留符的值：

+ 預留符標記基於 {{RESEARCH_DESC}} 內容逐項解釋。
  - 詳盡列舉各原則的條目。
  - 條目應基於**職責**進行調查與研究
+ 對於範例：各原則的 `PRINCIPLE_N_EXAMPLE` 是否輸出與使用的程式語言，一律依本次 {{RESEARCH_EXAMPLE_LANG}} 決定。
  - 若 {{RESEARCH_EXAMPLE_LANG}} 不為空 → 以 {{RESEARCH_EXAMPLE_LANG}} 指定的程式語言與資訊 ( 如版本、框架、函式庫 ) 撰寫範例程式，展示該原則如何運作。
  - 若 {{RESEARCH_EXAMPLE_LANG}} 為空 → 無需設計範例，並於步驟 3 刪除 **EXAMPLE** 區塊。
+ 對於治理日期：`RATIFICATION_DATE` 為原始通過日期 ( 如果未知，請詢問或標記為待辦事項 )，`LAST_AMENDED_DATE` 為當前日期 ( 果進行了更改 )，否則保留先前的日期。
+ `CONSTITUTION_VERSION` 必須依照語意版本控制規則遞增：
  - 主要版本：治理與原則發生不可兼容或重新定義的變更。
  - 次要版本：新增了原則或章節，亦或大幅擴展與補充了具體的指導方針。
  - 補丁版本：澄清、措詞、拼字錯誤修復、非語意改進。
+ 如果版本號遞增類型不明確，請在最終確定之前提出理由。

### 2.1 驗證範例程式 ( 僅於 RESEARCH_EXAMPLE_LANG 不為空時執行 )

+ RESEARCH_SKILL_NAME : `${CLAUDE_SKILL_DIR}` 的最後一層目錄名稱，轉為小寫，`a-z`、`0-9` 以外的字元改為 `-`，並去除首尾的 `-`
+ RESEARCH_VERIFY_HASH : {{RESEARCH_DOCUMENT_PATH}} 去除前後空白、以 `/` 分隔後，以 UTF-8 ( 無 BOM、無結尾換行 ) 計算的 MD5 小寫十六進位字串
+ RESEARCH_VERIFY_DIR : `${CLAUDE_PROJECT_DIR}/.claude/verify/{{RESEARCH_VERIFY_HASH}}`，路徑分隔符號統一為 `/`
  - `meta.json`：研究項目、輸出文件、程式語言、映像檔名稱、驗證方式、驗證日期
  - `Dockerfile`：鎖定基底映像檔版本，於建置階段安裝相依套件，以 `WORKDIR /work` 與 `CMD` 執行驗證程式
  - `.dockerignore`：排除 `out/`
  - `src/`：各原則的範例程式與驗證程式，驗證程式以結束碼表示成敗
  - `out/`：執行紀錄，不納入版本控制
+ RESEARCH_VERIFY_IMAGE : `{{RESEARCH_SKILL_NAME}}:{{RESEARCH_VERIFY_HASH}}`
+ 偵測容器服務：`docker info` 結束碼為 0，且 OSType 為 `linux`
  - 未安裝、服務未啟動、無權限 → 說明原因，詢問使用者是否啟動後重試；使用者拒絕 → 改用邏輯檢查
+ 建置：`docker build -t {{RESEARCH_VERIFY_IMAGE}} "{{RESEARCH_VERIFY_DIR}}"`
+ 執行：`docker run --rm --network none --mount "type=bind,src={{RESEARCH_VERIFY_DIR}},dst=/work" {{RESEARCH_VERIFY_IMAGE}}`
+ 建置或驗證失敗 → 修正範例或 Dockerfile 後重試，**不可** 改以邏輯檢查取代
+ 文件中的範例程式碼 **必須** 與驗證通過的檔案一致，並標示驗證方式 ( Docker 實測 / 邏輯檢查 ) 與環境版本

### 3. 更新樣板內容

+ 將所有預留符標記更換為具體文字 ( 除尚未定義的範本插槽外，不得保留任何帶有括號的標記；如有保留，需明確說明理由 )。
+ 若 {{RESEARCH_EXAMPLE_LANG}} 為空 → 刪除每個原則的 **EXAMPLE** 區塊 ( 含 `**EXAMPLE**:` 標籤、`[PRINCIPLE_N_EXAMPLE]` 預留符與其註釋 )，不得保留空白標籤或以「無」、「N/A」替代。
+ 保留標題層級結構，替換後可刪除註釋，除非註釋仍能提供澄清性的指引。
+ 確保每個原則包括以下部分：
  - 簡潔的標題行 ( Succinct name line )。
  - 段落或項目清單以概括必須遵守的鐵律 ( Paragraph or bullet list capturing non-negotiable rules )。
  - 若原則模糊不明顯，請解釋明確的理由 ( Explicit rationale if not obvious )。
  - 若 {{RESEARCH_EXAMPLE_LANG}} 不為空，提供以 {{RESEARCH_EXAMPLE_LANG}} 撰寫的範例程式 ( Example code if RESEARCH_EXAMPLE_LANG is not empty )。
+ 確保治理段落 ( Governance section ) 列出修訂程序、版本控制政策和合規性審查預期。
  - 若更新的憲章內容與舊版憲章，在條目與內容有新增、刪減，應該根據差異調整本次版本編號，並說明變更內容。

---

### 4. 輸出樣板內容

+ 若 `{{RESEARCH_DOCUMENT_FOLDER}}` 目錄不存在 → 建立 `{{RESEARCH_DOCUMENT_FOLDER}}` 目錄
+ 將完成的樣板內容寫回 `{{RESEARCH_DOCUMENT_PATH}}`，覆蓋原本內容。

---

### 5. 總結

+ 輸出 `✅ 執行完畢`
