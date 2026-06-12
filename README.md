# ShiftMaster 一鍵排班系統

> 專為台灣醫療機構設計的智能排班 Web App

## 功能亮點

- **一鍵自動排班** — Google OR-Tools CP-SAT 演算法，秒出整月班表
- **勞基法合規** — 支援一般制 & 四週變形工時，即時違規警示
- **擲骰子機制** — 解決大夜班沒人要上的世紀難題 🎲
- **LINE 通知** — 班表發布自動推播給所有員工
- **Google 行事曆** — 一鍵同步每位員工的個人班次
- **多格式匯出** — Excel / PDF / iCal

## 技術棧

| 層級 | 技術 |
|------|------|
| 前端/後端 | Next.js 16 + TypeScript + Tailwind CSS |
| 排班引擎 | Python FastAPI + Google OR-Tools |
| 資料庫 | PostgreSQL + Prisma 7 |
| 認證 | NextAuth.js |
| 通知 | LINE Messaging API |
| 行事曆 | Google Calendar API |

## 文件

- [PRD 產品需求文件](./docs/PRD.md)

## 開發狀態

- [x] PRD 完成
- [ ] 初始化專案架構
- [ ] 資料庫 Schema
- [ ] 排班引擎（OR-Tools）
- [ ] Web App MVP

## 作者

masalu0105-gif（陳勝宏）  
KB Medical 南區業務 × masalu.lab 個人品牌
