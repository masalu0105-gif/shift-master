# ShiftMaster — 一鍵排班系統 PRD

> 版本：v0.1  
> 建立日期：2026-04-08  
> 作者：masalu0105-gif（陳勝宏）  
> 狀態：規劃中，待 MVP 驗證

---

## 一、專案背景

KB Medical 南區業務在服務醫療院所客戶（診所、檢驗科、醫院部門）時，發現客戶普遍使用 Excel 或紙本做排班，痛點明顯：

- 排班耗時，護理長/行政人員每月花 2-3 天手動排班
- 調班後勞基法合規性需重算，怕勞檢被罰
- 班表公布後靠 LINE 群組通知，改班就洗版
- 沒有系統記錄員工不可上班時段，靠口頭溝通

**目標：** 打造一個傻瓜式「一鍵排班」Web App，讓醫療機構主管 5 分鐘內完成整月排班，並自動整合 Google 行事曆 + LINE 通知。

---

## 二、目標使用者

### 主要使用者：排班主管（護理長、診所行政、科主任）
- 設定班別、人員、規則
- 一鍵產生班表，手動微調
- 發布班表

### 次要使用者：員工
- 填寫不可上班時段
- 查看自己的班表
- 在 LINE 收到排班通知

---

## 三、核心功能（MVP 範圍）

### 3.1 機構 & 人員管理
- 建立機構（名稱、工時制度類型）
- 新增員工（姓名、職稱、性別、到職日期）
- 設定班別（名稱、開始/結束時間、顏色標籤）— 完全自定義

### 3.2 員工不可上班時段申報
- 員工可在排班前填寫「不可上班」日期/時段
- 主管可查看所有員工的申報狀況
- 申報截止後鎖定，進入排班流程

### 3.3 一鍵自動排班（核心）
- 輸入：員工清單、班別、人力需求（每班幾人）、不可上班申報、勞基法規則
- 演算法：Google OR-Tools CP-SAT（Python 微服務）
- 輸出：整月班表草稿
- 勞基法自動合規檢查，違規項目標紅警示

**支援工時制度：**
- 一般制（週 40hr）
- 四週變形工時（醫療保健業適用）

**硬約束（不可違反）：**
```
- 每日工時 ≤ 12 小時（含加班）
- 每週正常工時 ≤ 40 小時
- 每月加班 ≤ 46 小時
- 每 7 日至少 1 例假 + 1 休息日
- 連續工作天數 ≤ 6 天
- 換班間隔 ≥ 11 小時
- 員工不可上班時段不可排班
```

**軟約束（盡量滿足）：**
```
- 公平分配夜班次數
- 公平分配假日班次數
- 員工偏好班別盡量滿足
```

### 3.4 主管手動微調介面
- 甘特圖式/月曆式班表視圖
- 拖拉換班，即時合規檢查
- 顯示當前班表的違規警示清單

### 3.5 擲骰子機制（遊戲化）🎲
**三種模式，主管自由選擇：**

**模式 A — 搶救大夜班**
當某個班別無人自願時，系統標記為「待決」，主管可啟動骰子，從符合資格員工中隨機選出

**模式 B — 選班順序抽籤**
每月排班前，全員骰子決定「選班優先序號」，按號碼順序讓員工先後選想要的班

**模式 C — 雨露均霑（推薦）**
系統追蹤每位員工累積的「不爽班點數」（大夜班 +3、假日班 +2、連假班 +1）；
點數越高者，下次排班骰子自動加成（等效增加中籤機率），讓長期吃虧的人更容易骰到好班

### 3.6 班表發布 & 通知
- 發布後自動推送 LINE 通知（透過 LINE Notify 或 Messaging API）
- 自動建立/更新 Google 行事曆（每位員工的個人班次）
- 匯出格式：Excel、PDF、iCal

---

## 四、技術架構（方案 A — 獨立新專案）

```
┌─────────────────────────────────────────────────┐
│                   前端 (Next.js)                  │
│  排班表視圖 / 人員管理 / 申報介面 / 骰子動畫       │
└──────────────────────┬──────────────────────────┘
                       │ API Routes
┌──────────────────────▼──────────────────────────┐
│              後端 API (Next.js API Routes)         │
│  人員 CRUD / 排班管理 / 發布流程 / 認證             │
└──────────────────────┬──────────────────────────┘
                       │ HTTP (排班演算法呼叫)
┌──────────────────────▼──────────────────────────┐
│         排班演算法微服務 (Python FastAPI)           │
│         Google OR-Tools CP-SAT                   │
│         POST /solve → 回傳班表草稿                 │
└─────────────────────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────┐
│                  資料庫 (PostgreSQL)               │
│  Users / Shifts / Schedules / Preferences        │
└─────────────────────────────────────────────────┘
              ┌────────┴────────┐
  ┌───────────▼──────┐  ┌──────▼──────────┐
  │  Google Calendar  │  │   LINE Notify   │
  │  API 整合         │  │   推播通知       │
  └──────────────────┘  └─────────────────┘
```

### 技術棧

| 層級 | 技術 |
|------|------|
| 前端 | Next.js 16 + React 19 + Tailwind CSS |
| 後端 | Next.js API Routes (TypeScript) |
| 排班引擎 | Python FastAPI + Google OR-Tools (`pip install ortools`) |
| 資料庫 | PostgreSQL + Prisma 7 |
| 認證 | NextAuth.js（角色：admin / manager / staff） |
| 前端 UI 元件 | react-scheduler（甘特圖/月曆） |
| 通知 | LINE Messaging API |
| 行事曆 | Google Calendar API |
| 匯出 | xlsx + jsPDF |
| 部署 | Vercel（Next.js） + Railway/Render（Python 微服務） |

---

## 五、資料模型（概要）

```prisma
model Organization {
  id          String   @id
  name        String
  workType    String   // "standard" | "4week"
  employees   Employee[]
  shiftTypes  ShiftType[]
  schedules   Schedule[]
}

model Employee {
  id          String   @id
  name        String
  role        String   // "manager" | "staff"
  gender      String
  hireDate    DateTime
  orgId       String
  preferences Preference[]
  assignments ShiftAssignment[]
  diceScore   Int      @default(0)  // 雨露均霑點數
}

model ShiftType {
  id        String   @id
  name      String   // e.g. "早班", "大夜班"
  startTime String   // "07:00"
  endTime   String   // "15:00"
  color     String   // hex color
  orgId     String
}

model Preference {
  id         String   @id
  employeeId String
  date       DateTime
  type       String   // "unavailable" | "preferred"
  note       String?
}

model Schedule {
  id          String   @id
  orgId       String
  month       String   // "2026-05"
  status      String   // "draft" | "published"
  assignments ShiftAssignment[]
  createdAt   DateTime
}

model ShiftAssignment {
  id          String   @id
  scheduleId  String
  employeeId  String
  shiftTypeId String
  date        DateTime
  isOvertime  Boolean  @default(false)
  dicePoints  Int      @default(0) // 此次排班累積的點數
}
```

---

## 六、頁面結構（MVP）

```
/                       → 登入頁
/dashboard              → 主管總覽（本月班表概況、待辦事項）
/schedule               → 排班表主頁（月曆視圖 + 甘特圖切換）
/schedule/new           → 新建排班流程（設定 → 員工確認 → 一鍵排班 → 微調 → 發布）
/employees              → 員工管理
/shifts                 → 班別設定
/preferences            → 員工申報頁（員工登入後填不可上班時段）
/dice                   → 骰子頁（排班衝突決策、抽籤選班順序）
/settings               → 機構設定（工時制度、LINE 設定、Google Cal 設定）
```

---

## 七、MVP 一週開發計畫（概要）

| 天 | 工作 |
|----|------|
| Day 1 | 初始化專案 + 資料庫 schema + 認證 |
| Day 2 | 人員管理 + 班別設定 + 員工申報 |
| Day 3 | Python OR-Tools 排班引擎 API |
| Day 4 | 一鍵排班 + 班表視圖 + 手動微調 |
| Day 5 | 勞基法合規檢查 + 骰子機制 |
| Day 6 | LINE 通知 + Google Calendar 整合 |
| Day 7 | Excel/PDF 匯出 + 部署 + 測試 |

---

## 八、未來規劃（v2+）

- 多機構 SaaS 多租戶架構
- 員工手機版 PWA
- 自動統計加班費試算
- 護病比合規警示（醫療專屬）
- AI 排班學習員工偏好（歷史資料訓練）
- 收費模式：$100-150/人/月（甜蜜點：比 Aibou $168 便宜，比 104 功能強）

---

## 九、競品參考

| 產品 | 一鍵排班 | 勞基法 | 價格 | 我們的差異 |
|------|:---:|:---:|------|------|
| NUEIP | 有 | 有 | ~$1,365/月 | 更懂醫療場域 |
| Aibou Crew | 有 | 有 | $168/人/月 | 擲骰子 + 醫療合規 |
| 104 企業大師 | 無 | 有 | $40-70/人/月 | 有自動排班 |

**核心差異化：醫療場域 domain knowledge + 擲骰子遊戲化 + LINE/Google Cal 完整整合**

---

## 十、研究報告參考

- 勞基法規則：本機研究報告《一鍵排班表_研究報告》（未納入 repo）
- GitHub 參考專案：OR-Tools, j3soon/nurse-scheduling, Vhivi/ScheduleOptimization
- 競品分析：NUEIP, MAYOHR, FREONE, Aibou Crew
