## 繪製關聯圖

使用 `/workkit-relationship-diagram`，並於對話提供關聯圖描述

+ 雙向圖
```
	-- 設計規範、編寫指南 -->
需求文件			程式碼
	<-- 摘要規範、編寫只能 --
```

+ 單向圖

```
Coding Guideline Constitution -- 指定程式語言 --> coding.md -- 指定軟體框架 --> framework.md -- 指定框架與依賴庫版本 --> framework-dependence.md
```

+ 多來源彙整
```
LeetCode Issue -- 題目描述 -↘          LeetCode Issue -- 約束條件 -↘
					PseudoCode -- 邏輯描述 --> Program
PseudoCode Constitution -- 設計原則 -↗	Coding Guideline -- 約束條件 -↗
```

## 生成 PDF 檔案

嚴格遵守以下原則匯出 PDF 檔案：

+ 以**極簡現代風 (Minimalist Modern)**設計 PDF 格式
+ 章節標題，以簡潔扼要方式描述，且請勿超過 15 個字

內容來源：

+ 詮釋提供的文件檔案
+ 請詳盡列舉或搜尋網路公開資訊，以提供 **虛擬碼**、**程式碼**、**測試碼** 的新舊改動風險評估方式，若有第三方軟體亦可提供並說明。
+ 最後章節**結論**，請總結與彙整所有對話內容
