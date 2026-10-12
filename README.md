# BlushNote：基於動態臉部辨識與端點 AI 之美妝 AR 教學系統

> **指導教授**：劉譯閎 教授  
> **競賽經歷**：2026 大專校院資訊應用服務創新競賽產學合作組（現在進行式）
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

作為專案的演算法研發與後端資料庫核心負責人，本人主要主導以下模組之設計與實作：

### 1. 臉部特徵分析原型與加權推薦演算法研發 (Python & MCDM Algorithm)
* **特徵分析演算法原型**：在 Jupyter Notebook 中建立人臉特徵分析與膚色色調判斷演算法原型，驗證非結構化影像特徵轉化為結構化標籤的可行性。
* **雙來源加權推薦邏輯**：設計結合 MCDM（多標準決策分析）與 Jaccard 相似度的推薦架構，進行邊界條件測試、權重靈敏度校正與骨骼相容矩陣設計，確保推薦結果兼具精準度與可解釋性。

### 2. Supabase 雲端資料庫建置與資安政策 (PostgreSQL & RLS)
* **資料庫 Schema 規劃**：於 Supabase (PostgreSQL) 建立並正規化 7 張核心資料表（涵蓋 Profiles、Templates、Steps、收藏夾與社群互動）。
* **生物特徵防護與 RLS 列級安全**：落實 Privacy by Design 原則，並於 PostgreSQL 撰寫 3 項嚴謹的 Row-Level Security (RLS) 安全政策，配合 JWT 鑑權實作企業級用戶權限隔離。

### 3. 前後端 API 串接文檔與開發協作 (API & Developer Documentation)
* **跨端串接指南整理**：負責撰寫與整理 Supabase RESTful API 通訊規範、PostgREST 查詢範例及資料結構規格書，提供給負責 iOS 環境開發的同學進行端對端串接與整合。

### 4. 專案進度管理與資服競賽籌備 (Project Management)
* **進度控制與里程碑交付**：統籌團隊 4 人之開發時程與任務指派，控管資服競賽產學合作組系統概述文件、技術展示文檔與展演 Prototype 之準時交付。

---

## 🧠 核心推薦演算法與數學模型 (Recommendation Engine)

推薦引擎採用**多標準決策分析 (MCDM)** 與 **Jaccard 相似度** 結合的加權評分架構，將感性的彩妝經驗轉化為可解釋的量化指標，計算總契合度得分（Score 介於 0 至 100 分）：

**推薦總分公式**：  
`Score = 100 × (0.50 × S_skin + 0.35 × S_face + 0.15 × S_style)`

---

### 權重分配與指標拆解

* **膚色調性相似度 (S_skin) — 權重 50%**  
  彩妝之首在於底妝調性。
  * 完全匹配（如：暖色 ↔ 暖色）：給予 `1.0`
  * 中性色調過渡（如：中性 ↔ 暖色/冷色）：給予 `0.5`
  * 衝突色調：給予 `0.0`

* **臉型骨骼相容性 (S_face) — 權重 35%**  
  考慮彩妝技法之跨臉型轉移性。
  * 完全相同：給予 `1.0`
  * 相容結構對（如：`["圓臉", "鵝蛋臉"]`、`["長臉", "心型臉"]`、`["方臉", "長臉"]`）：給予 `0.5`
  * 其餘非相容組合：給予 `0.0`

* **風格標籤相似度 (S_style) — 權重 15%**  
  採用 **Jaccard 相似度** 運算用戶自選標籤集 A 與模板標籤集 B 之重合度：  
  `S_style = |A ∩ B| / |A ∪ B|`  
  *(若使用者未勾選偏好標籤，預設 `S_style = 1.0` 不扣分)*

---

## 🏗️ 系統總體架構 (System Architecture)

<p align="center">
  <img src="https://github.com/user-attachments/assets/09c82460-1549-4629-98a1-0c64ae9c4d55" width="85%" alt="BlushNote 系統總體架構圖">
</p>

### 架構亮點
1. **端點 AI 運算 (Client Side)**：於 iPadOS 本地端透過 AVFoundation 擷取實時視訊，呼叫 Apple Vision 進行幾何臉型與 HSV 膚色色調辨識，完全不經雲端 API。
2. **微服務與資安防護 (Supabase Backend)**：串接 Supabase GoTrue 進行 JWT 鑑權，PostgreSQL 開啟 Row-Level Security (RLS) 列級安全政策，確保用戶隱私與權限隔離。

---

## 🛠️ 技術棧與工具列表 (Tech Stack)

* **演算法原型與數值驗證**：Python, Jupyter Notebook, NumPy, Pandas (MCDM 模型與特徵分析)
* **後端與雲端架構**：Supabase, PostgreSQL, PostgREST, GoTrue Auth
* **資安與存取控制**：Row-Level Security (RLS) 權限隔離策略, JWT Authentication
* **iOS / iPadOS App 協作**：Swift, SwiftUI, Apple Vision Framework, ARKit
* **專案管理與協作**：Git, GitHub, Xcode
