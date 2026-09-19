# BlushNote：基於動態臉部辨識之美妝 AR 教學系統

> **指導教授**：劉譯閔 教授  
> **專案小組成員**：盧恩佳、洪語欣、廖冠筑、簡偉玲  
> **個人核心職責**：演算法原型研發與驗證、雲端資料庫架構設計 (Supabase / RLS)、雙來源加權推薦邏輯、專案進度管理

---

## 📌 專案背景與核心價值
針對美妝新手對自身臉型特徵認知不足、妝容選品決策成本高及化妝步驟缺乏即時反饋等痛點，本專案開發 iPad 原生美妝引導應用 **BlushNote**。系統結合裝置端臉部特徵分析演算法、個人化加權推薦模型與 AR 即時步驟疊加引導，打造兼顧個人隱私與互動體驗的美妝學習方案。

---

## 🏗️ 系統總體架構 (System Architecture)

系統採用**「邊緣運算特徵解析 ＋ 雲端微服務解耦」**架構，落實隱私優先原則：

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'primaryColor': '#FDECEF', 'edgeLabelBackground':'#FFF9F9', 'tertiaryColor': '#FAF0F2'}}}%%
flowchart TD
    %% 節點定義
    A["📷 使用者 / iPad 鏡頭串流"] -->|"AVFoundation 實時影像串流"| B["📱 裝置端特徵分析模組"]

    subgraph Client["🌸 裝置端邊緣運算 (Client Device)"]
        B --> C["Face Landmark 幾何特徵捕捉<br/>(五官定位 · 輪廓長寬比例)"]
        B --> D["HSV 色彩空間膚色識別<br/>(冷暖色調 · 膚色明度等級)"]
        C --> E["📐 臉型分類規則引擎<br/>(分析完成度 96% 信心校驗)"]
        D --> E
        E -->|"客觀生理標籤 (權重 x2)"| F["✨ 雙來源標籤加權推薦引擎"]
        CustomTag["🏷️ 使用者自選偏好標籤<br/>(風格 · 場合 · 權重 x1)"] --> F
    end

    subgraph Backend["☁️ 雲端服務與資料庫 (Supabase Backend)"]
        G["🔐 GoTrue 身分驗證服務<br/>(/auth/v1 · JWT 鑑權)"] <-->|"REST API"| H["🌐 iPad URLSession 通訊層"]
        I["⚡ PostgREST 資料存取介面<br/>(/rest/v1 · 動態 CRUD)"] <-->|"REST API"| H
        I <--> J[("🗄️ PostgreSQL 關聯資料庫<br/>(7張核心資料表)")]
        J --- K["🛡️ 3 項 RLS 列級安全性政策<br/>(用戶隱私與權限隔離)"]
    end

    F <-->|"匹配查詢與權重計算"| H
    F --> L["💄 ARKit 虛擬面具動態彩妝疊加教學"]

    %% 美學樣式定義 (BlushNote 專屬溫柔玫瑰色系)
    classDef pinkBox fill:#FCECEF,stroke:#D9828A,stroke-width:1.8px,color:#5A2A32,rx:10px,ry:10px;
    classDef softRose fill:#F9E2E6,stroke:#C46872,stroke-width:1.8px,color:#4A2026,rx:10px,ry:10px;
    classDef wineRed fill:#9D384D,stroke:#7F2538,stroke-width:2px,color:#FFFFFF,rx:12px,ry:12px;
    classDef cloudBox fill:#FFF5F6,stroke:#E2A8B0,stroke-width:1.8px,stroke-dasharray: 4 4,color:#5C3238,rx:10px,ry:10px;
    classDef dataBox fill:#F5D5DC,stroke:#B05362,stroke-width:2px,color:#421C22,rx:12px,ry:12px;

    class A,L wineRed;
    class B,C,D,CustomTag pinkBox;
    class E,F softRose;
    class G,I,H,K cloudBox;
    class J dataBox;
```

---

## 🎬 實機操作動態展示 (Live Application Demos)

系統介面以優雅溫和的視覺體驗為核心，結合流暢的帳號驗證、邊緣端即時人臉分析與專屬瀑布流推薦：

| 01. 帳號註冊與個人設定 | 02. 即時人臉分析與標籤推薦 | 03. 快速登入與專屬推薦主頁 |
| :---: | :---: | :---: |
| <img src="./docs/videos/01_register.gif" width="280" alt="註冊與暱稱頭像設定" /> | <img src="./docs/videos/02_analysis.gif" width="280" alt="人臉特徵與膚色分析" /> | <img src="./docs/videos/03_login_home.gif" width="280" alt="登入與瀑布流首頁" /> |
| **流暢註冊與個人設定**<br>• 電子郵件與密碼安全檢查<br>• 建立使用者暱稱與個人化帳號<br>• 串接 Supabase GoTrue 鑑權 | **智慧檢測與特徵標籤**<br>• 光線與臉部角度即時提示<br>• 幾何特徵點計算臉型 (鵝蛋臉)<br>• HSV 膚色色調判定 (暖色調/健康)<br>• 雙來源特徵標籤即時比對 | **專屬個人化美妝首頁**<br>• 快速登入鑑權與狀態同步<br>• 依照加權匹配分數動態排序<br>• 瀑布流瀏覽專屬推薦妝容 |
