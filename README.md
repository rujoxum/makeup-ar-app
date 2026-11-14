# BlushNote：基於靜態臉部辨識與端點 AI 之美妝 AR 教學系統

> **指導教授**：劉譯閎 教授  
> **專案小組成員**：盧恩佳、洪語欣、廖冠筑、簡偉玲  
> **個人核心職責**：特徵分析與推薦演算法原型研發 (Python / Jupyter Notebook)、雲端資料庫建置 (Supabase / PostgreSQL / RLS)、iOS 通訊串流與 API 文件整理、專案進度管理

---

## 📌 專案背景與核心價值

針對美妝新手對自身臉型特徵認知不足、妝容選品決策成本高及化妝步驟缺乏即時反饋等痛點，本專案開發 iPad 原生美妝引導應用 **BlushNote**。

系統結合裝置端臉部特徵分析演算法、個人化加權推薦模型與 AR 即時步驟疊加引導，打造兼顧個人隱私與互動體驗的美妝學習方案：
1. **Privacy by Design（零影像外傳）**：影像處理與特徵擷取 100% 於 iPad 本地端 NPU/GPU 運算，照片不回傳雲端，自源頭杜絕生物特徵洩漏風險。
2. **多標準決策推薦（MCDM Algorithm）**：結合生理客觀特徵與主觀風格偏好，毫秒級計算出具可解釋性的個人化妝容推薦。
3. **空間化 3D 步驟引導**：於視訊畫面上實時疊加 3D 塗抹範圍與暈染方向，實現「對鏡跟練」的沉浸式教學。

---

## 👩‍💻 個人核心貢獻與技術實作 (Key Personal Contributions)

作為負責專案的演算法研發與後端資料庫，本人主要主導以下模組之設計與實作：

### 1. 臉部特徵分析與多維度推薦引擎研發 (Python & Hybrid MCDM)
* **雙層多標準決策推薦架構**：設計結合「客觀生理硬體適配度 (50%)」與「主觀美妝偏好適配度 (50%)」的雙層加權評分模型，並支援排斥風格 (`avoidStyles`) 硬過濾與連續親和度矩陣計算。
* **MMR 多樣性重排與可解釋性 (XAI)**：導入 Maximal Marginal Relevance (MMR, $\lambda=0.86$) 演算法重排 Top-3 推薦清單，解決推薦結果同質化問題；同時設計動態理由生成器與「高/中/低」三級信心度評估機制（綜合考量總分門檻、Top1-Top2 分差與核心特徵匹配數）。
* **離線模擬評估框架 (Offline Evaluation)**：基於 Dirichlet 分佈建構市場模板與 1,000 名模擬用戶生成器，並設計包含行為雜訊的隱藏真值 (Hidden Ground Truth)，執行 30 次可重現隨機種子實驗，全面評估 `Precision@3`、`Recall@3`、`NDCG@3`、目錄覆蓋率 (`CatalogCoverage`) 與曝光基尼係數 (`ExposureGini`)。

### 2. Supabase 雲端資料庫建置與資安政策 (PostgreSQL & RLS)
* **資料庫 Schema 規劃**：於 Supabase (PostgreSQL) 建立並正規化 7 張核心資料表（涵蓋 Profiles、Templates、Steps、收藏夾與社群互動）。
* **生物特徵防護與 RLS 列級安全**：落實 Privacy by Design 原則，並於 PostgreSQL 撰寫 Row-Level Security (RLS) 安全政策，配合 JWT 鑑權實作企業級用戶權限隔離。

### 3. 前後端 API 串接文檔與開發協作 (API & Developer Documentation)
* **跨端串接指南整理**：負責撰寫與整理 Supabase RESTful API 通訊規範、PostgREST 查詢範例及資料結構規格書，提供給負責 iOS 環境開發的同學進行端對端串接與整合。

### 4. 專案進度管理與資服競賽籌備 (Project Management)
* **進度控制與里程碑交付**：統籌團隊 4 人之開發時程與任務指派，控管專題進度會議內容之準時交付。

---

## 🧠 核心推薦演算法與數學模型 (Recommendation Engine)

推薦引擎採用**雙層多標準決策分析 (MCDM)**、**Jaccard 相似度**與 **MMR 多樣性重排**架構，將感性的彩妝經驗轉化為可解釋的量化指標，計算最終契合度得分（`Score` 介於 `0` 至 `1` 之間）：

**最終推薦總分公式**：  
`Final Score = 0.50 × S_hardware + 0.50 × S_preference`
---

### 雙層指標與權重拆解

#### 第一層：客觀生理硬體適配度 (`S_hardware` — 佔總分 50%)
`S_hardware = 0.35 × S_face + 0.25 × S_skin + 0.25 × S_tone + 0.15 × S_feature`

- **臉型骨骼親和度 (`S_face`，權重 35%)**：建立 `5 × 5` 連續相容度矩陣 (`face_affinity`)，衡量彩妝技法之跨臉型轉移性（完全相同為 `1.0`，如「鵝蛋臉 ↔ 心形臉」給予 `0.85`、「圓臉 ↔ 鵝蛋臉」給予 `0.80`）
- **膚色深淺適配度 (`S_skin`，權重 25%)**：依序評估「白皙、自然、健康小麥」三階距離，同階給予 `1.0`、相鄰一階給予 `0.50`、跨兩階給予 `0.10`
- **色彩調性相容性 (`S_tone`，權重 25%)**：完全同調（如暖調 ↔ 暖調）給予 `1.0`；包含中性色調過渡給予 `0.65`；冷暖衝突色調給予 `0.20`
- **五官細節重合度 (`S_feature`，權重 15%)**：針對眼型（單眼皮、內雙等）與局部特徵（高顴骨、唇形等）計算 **Jaccard 相似度**

#### 第二層：主觀風格偏好適配度 (`S_preference` — 佔總分 50%)
*(前置過濾：若模板包含用戶設定之排斥風格 `avoidStyles`，直接剔除)*
`S_preference = 0.50 × S_style + 0.35 × S_color + 0.15 × S_inspiration`

- **風格標籤相似度 (`S_style`，權重 50%)**：計算用戶偏好風格集與模板風格集之 Jaccard 相似度 `|A ∩ B| / |A ∪ B|`
- **色系偏好相似度 (`S_color`，權重 35%)**：計算用戶偏好彩妝色系（如大地色系、肉桂烏龍）與模板色系之 Jaccard 相似度
- **妝容靈感相似度 (`S_inspiration`，權重 15%)**：計算局部妝容重點（如乾淨底妝、原生感毛流）之 Jaccard 相似度

#### 第三階段：MMR 多樣性重排與信心度判定
- **MMR 重排公式**：在選出 Top-1 後，後續候選模板依 `MMR = λ × Score - (1 - λ) × max(Sim_selected)` 進行重排（設定 `λ = 0.86`），確保推薦清單兼顧高契合度與視覺多樣性
- **三級信心度輸出**：當 Top-1 分數 `≥ 0.70`、與 Top-2 分差 `≥ 0.05` 且核心生理/風格條件命中 `≥ 3` 項時，系統輸出「**高信心度**」並自動生成最多 4 條白話推薦理由

---

## 🏗️ 系統總體架構 (System Architecture)

<p align="center">
  <img src="https://github.com/user-attachments/assets/09c82460-1549-4629-98a1-0c64ae9c4d55" width="85%" alt="BlushNote 系統總體架構圖">
</p>

### 架構亮點
1. **端點 AI 運算 (Client Side)**：於 iPadOS 本地端透過 AVFoundation 擷取實時視訊，呼叫 Apple Vision 進行幾何臉型與 HSV 膚色色調辨識，不經雲端 API。
2. **微服務與資安防護 (Supabase Backend)**：串接 Supabase GoTrue 進行 JWT 鑑權，PostgreSQL 開啟 Row-Level Security (RLS) 列級安全政策，確保用戶隱私與權限隔離。

---

## 🛠️ 技術棧與工具列表 (Tech Stack)

* **演算法原型與數值驗證**：Python, Jupyter Notebook, NumPy, Pandas (MCDM 模型與特徵分析)
* **後端與雲端架構**：Supabase, PostgreSQL, PostgREST, GoTrue Auth
* **資安與存取控制**：Row-Level Security (RLS) 權限隔離策略, JWT Authentication
* **iOS / iPadOS App 協作**：Swift, SwiftUI, Apple Vision Framework, ARKit
* **專案管理與協作**：Git, GitHub, Xcode
