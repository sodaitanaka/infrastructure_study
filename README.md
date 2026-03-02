# Infrastructure Study Project (2026.03 - 2026.06)

本リポジトリは、SaaS（n8n/HubSpot）を活用したPM経験を持つ私が、
「インフラ構成をゼロから設計・提案できるITコンサル/PM」へ進化するための学習記録です。

## 1. なぜ「インフラ」を学ぶのか

### 現状の課題
* **抽象的な提案の限界:** n8nやスプレッドシートを用いた構築経験はあるが、大規模化・アプリ化の際の「インフラ構成」を具体化できていない。
* **技術選定の根拠不足:** DBの種類やサーバー構成の選択肢、それぞれのトレードオフを論理的に説明できない。

### 4ヶ月後の理想像
* **AWS SAA取得:** クラウドインフラの標準知識を体系的に習得している。
* **アーキテクチャ設計力:** 顧客の要件に対し、三層アーキテクチャに基づいた構成図（Mermaid等）を用いて一次回答ができる。

---

## 2. インフラ移行の概念イメージ

n8n+スプシの構成から、スケーラブルなWebアプリ構成への進化図です。

```mermaid
graph LR
    subgraph "Current: SaaS-based"
        A[n8n / HubSpot] --> B[(Spreadsheet)]
    end

    subgraph "Future: Cloud Infrastructure"
        C[Frontend: React/Next.js] -- "REST API" --> D[Compute: Lambda/EC2]
        D --> E[(Database: RDS/DynamoDB)]
    end

    B -.->|Migration| E
