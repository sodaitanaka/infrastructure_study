# LinkedIn追加機能 実装パターン システムアーキテクチャ比較

---

## アーキテクチャ比較表

| 項目 | A | B | C | D | E | F |
|---|---|---|---|---|---|---|
| **〈データソース〉** | | | | | | |
| 外部データ取得 | BD Dataset (CSV) | BD Dataset + Scraper API | BD Dataset | BD Dataset + Scraper API | PB LinkedIn Scraper | PB LinkedIn Scraper |
| 差分更新 | ✗ 手動再購入 | ✓ URL指定取得 (¥0.23/件) | ✓ AUTOHUNT内部 | ✓ URL指定取得 (¥0.23/件) | ✓ 定期スクレイプ | ✓ 定期スクレイプ |
| **〈データ層〉** | | | | | | |
| DB / ストレージ | Google Sheets | Google Sheets | AUTOHUNT内部DB | カスタムDB (Firestore等) | Google Sheets | Google Sheets |
| マルチテナント対応 | ✗ | ✗ | △ AH側 | ✓ 独自設計 | ✗ | ✗ |
| **〈バックエンド〉** | | | | | | |
| カスタムサーバー | なし | なし | なし | ✓ 要 (Cloud Functions等) | なし | なし |
| ワークフロー自動化 | n8n Cloud | n8n Cloud | AUTOHUNT (+ n8n) | n8n Cloud | n8n Cloud | n8n Cloud |
| AI / メッセージ生成 | Claude API | Claude API | AH AI + Claude (opt) | Claude API | Claude API | Claude API |
| **〈UI層〉** | | | | | | |
| 操作画面 | Google Sheets | Google Sheets | AUTOHUNT Web UI | カスタム Web UI (React等) | Sheets + PB UI | Sheets + PB UI |
| カスタムUI開発 | ✗ | ✗ | ✗ | ✓ フルスタック | ✗ | ✗ |
| **〈DM送信〉** | | | | | | |
| 送信方式 | 手動 | 手動 | AUTOHUNT自動 | 手動 (URL補助付) | 手動 | PB自動 ⚠️ |
| LinkedIn API連携 | なし | なし | AH公式連携 | なし | Cookie流用 | Cookie流用 |
| **〈カスタム開発範囲〉** | | | | | | |
| 開発スコープ | n8nフロー | n8nフロー | n8nフロー (最小) | UI + API + n8nフロー | n8nフロー | n8nフロー |
| **〈外部SaaS依存〉** | | | | | | |
| 使用SaaS | BD | BD | BD + AUTOHUNT | BD | Phantombuster | Phantombuster |
| ToSリスク | 低 | 低 | 低〜中 | 低 | 中 (グレー) | 高 (BAN risk) |

---

## パターンA〜Fのシステムアーキテクチャ図

---

## パターンA — BD / Dataset のみ

**一括購入・買い切り（差分更新なし）**

```mermaid
flowchart TB
    subgraph EXT["外部データソース"]
        BD[Bright Data\nDataset]
    end

    subgraph BACKEND["バックエンド / 自動化"]
        N8N[n8n Cloud]
        CLAUDE[Claude API]
        N8N <-->|メッセージ生成| CLAUDE
    end

    subgraph DATA["データ / UI 層"]
        GS[(Google Sheets\nDB 兼 操作UI)]
    end

    subgraph OUTPUT["送信"]
        LI[LinkedIn\n手動DM]
    end

    BD -->|CSV 手動投入| GS
    GS -->|候補者データ| N8N
    N8N -->|生成メッセージ書き戻し| GS
    GS -.->|手動コピペ| LI

    style BD fill:#DBEAFE,stroke:#2563EB
    style N8N fill:#D1FAE5,stroke:#10B981
    style CLAUDE fill:#D1FAE5,stroke:#10B981
    style GS fill:#FEF9C3,stroke:#CA8A04
    style LI fill:#0A66C2,color:#fff,stroke:#0A66C2
```

**カスタム開発スコープ：n8nワークフロー のみ**

---

## パターンB — BD / Dataset + Scraper API

**初回購入 ＋ URL指定差分更新**

```mermaid
flowchart TB
    subgraph EXT["外部データソース"]
        BDD[Bright Data\nDataset]
        BDS[Bright Data\nScraper API]
        BDD -->|URLリスト供給| BDS
    end

    subgraph BACKEND["バックエンド / 自動化"]
        N8N[n8n Cloud]
        CLAUDE[Claude API]
        N8N <-->|メッセージ生成| CLAUDE
    end

    subgraph DATA["データ / UI 層"]
        GS[(Google Sheets\nDB 兼 操作UI)]
    end

    subgraph OUTPUT["送信"]
        LI[LinkedIn\n手動DM]
    end

    BDS -->|最新プロフィール| GS
    GS -->|候補者データ| N8N
    N8N -->|生成メッセージ書き戻し| GS
    GS -.->|手動コピペ| LI

    style BDD fill:#DBEAFE,stroke:#2563EB
    style BDS fill:#DBEAFE,stroke:#2563EB
    style N8N fill:#D1FAE5,stroke:#10B981
    style CLAUDE fill:#D1FAE5,stroke:#10B981
    style GS fill:#FEF9C3,stroke:#CA8A04
    style LI fill:#0A66C2,color:#fff,stroke:#0A66C2
```

**カスタム開発スコープ：n8nワークフロー のみ**

---

## パターンC — BD + AUTOHUNT 連携

**AUTOHUNT SaaSによるスコアリング + 自動DM**

```mermaid
flowchart TB
    subgraph EXT["外部データソース"]
        BD[Bright Data\nDataset]
    end

    subgraph SAAS["AUTOHUNT SaaS"]
        AH_DB[(AUTOHUNT\n内部DB)]
        AH_AI[転職兆候スコアリング\n特許AI]
        AH_UI[AUTOHUNT\nWeb UI]
        AH_SEND[LinkedIn\n自動DM送信]
        AH_DB --> AH_AI --> AH_UI
        AH_UI --> AH_SEND
    end

    subgraph BACKEND["バックエンド / 自動化（オプション）"]
        N8N[n8n Cloud]
        CLAUDE[Claude API]
        N8N <-->|カスタムメッセージ生成| CLAUDE
    end

    subgraph OUTPUT["送信"]
        LI[LinkedIn\nAPI 公式連携]
    end

    BD -->|データインポート| AH_DB
    N8N -.->|カスタムメッセージ投入\n※オプション| AH_SEND
    AH_SEND -->|一括自動送信| LI

    style BD fill:#DBEAFE,stroke:#2563EB
    style AH_DB fill:#FEF3C7,stroke:#F59E0B
    style AH_AI fill:#FEF3C7,stroke:#F59E0B
    style AH_UI fill:#FEF3C7,stroke:#F59E0B
    style AH_SEND fill:#FEF3C7,stroke:#F59E0B
    style N8N fill:#D1FAE5,stroke:#10B981
    style CLAUDE fill:#D1FAE5,stroke:#10B981
    style LI fill:#0A66C2,color:#fff,stroke:#0A66C2
```

**カスタム開発スコープ：AHインポート設定 + n8nワークフロー（オプション）**

---

## パターンD — BD / Dataset + Scraper API + カスタムフロント

**差分更新あり ＋ 専用Web UI（マルチテナント対応）**

```mermaid
flowchart TB
    subgraph EXT["外部データソース"]
        BDD[Bright Data\nDataset]
        BDS[Bright Data\nScraper API]
        BDD -->|URLリスト供給| BDS
    end

    subgraph BACKEND["バックエンド / 自動化"]
        API[API Server\nCloud Functions等]
        N8N[n8n Cloud]
        CLAUDE[Claude API]
        N8N <-->|メッセージ生成| CLAUDE
        API <-->|ワークフロー連携| N8N
    end

    subgraph DATA["データ層"]
        DB[(カスタムDB\nFirestore / PostgreSQL等\nマルチテナント設計)]
    end

    subgraph UI["UI 層"]
        WEB[カスタム Web UI\nReact / Vue\n候補者一覧・フィルタ・承認]
    end

    subgraph OUTPUT["送信"]
        LI[LinkedIn\n手動DM\nURLクリック補助]
    end

    BDS -->|最新プロフィール| API
    API -->|保存| DB
    DB -->|データ供給| WEB
    WEB -->|操作・承認| API
    API --> N8N
    N8N -->|生成メッセージ| DB
    WEB -.->|URLクリック補助| LI

    style BDD fill:#DBEAFE,stroke:#2563EB
    style BDS fill:#DBEAFE,stroke:#2563EB
    style API fill:#EDE9FE,stroke:#7C3AED
    style DB fill:#EDE9FE,stroke:#7C3AED
    style WEB fill:#EDE9FE,stroke:#7C3AED
    style N8N fill:#D1FAE5,stroke:#10B981
    style CLAUDE fill:#D1FAE5,stroke:#10B981
    style LI fill:#0A66C2,color:#fff,stroke:#0A66C2
```

**カスタム開発スコープ：Web UI + API Server + DB設計 + n8nワークフロー（フルスタック）**

---

## パターンE — Phantombuster / スクレイピングのみ

**PBでデータ収集 + n8nでメッセージ生成（DM送信は手動）**

```mermaid
flowchart TB
    subgraph EXT["外部データソース"]
        PB[Phantombuster\nLinkedIn Profile Scraper\nSaaS]
        LIN_SRC[LinkedIn\n※スクレイピング対象]
        PB -->|スクレイピング\nCookie流用| LIN_SRC
    end

    subgraph BACKEND["バックエンド / 自動化"]
        N8N[n8n Cloud]
        CLAUDE[Claude API]
        N8N <-->|メッセージ生成| CLAUDE
    end

    subgraph DATA["データ / UI 層"]
        GS[(Google Sheets\nDB 兼 操作UI)]
    end

    subgraph OUTPUT["送信"]
        LI[LinkedIn\n手動DM]
    end

    PB -->|CSV出力| GS
    GS -->|候補者データ| N8N
    N8N -->|生成メッセージ書き戻し| GS
    GS -.->|手動コピペ| LI

    style PB fill:#FEE2E2,stroke:#EF4444
    style LIN_SRC fill:#FEE2E2,stroke:#EF4444
    style N8N fill:#D1FAE5,stroke:#10B981
    style CLAUDE fill:#D1FAE5,stroke:#10B981
    style GS fill:#FEF9C3,stroke:#CA8A04
    style LI fill:#0A66C2,color:#fff,stroke:#0A66C2
```

> ⚠️ **リスク**：LinkedIn ToS グレーゾーン。レート制限遵守で実運用BANリスクは管理可能。

**カスタム開発スコープ：n8nワークフロー のみ**

---

## パターンF — Phantombuster / DM自動送信（PoC）

**PBでスクレイピング〜DM送信まで自動化（BAN・ToSリスク高）**

```mermaid
flowchart TB
    subgraph EXT["外部データソース"]
        PB[Phantombuster\nLinkedIn Profile Scraper]
        LIN_SRC[LinkedIn\n※スクレイピング対象]
        PB -->|スクレイピング\nCookie流用| LIN_SRC
    end

    subgraph BACKEND["バックエンド / 自動化"]
        N8N[n8n Cloud]
        CLAUDE[Claude API]
        N8N <-->|メッセージ生成| CLAUDE
    end

    subgraph DATA["データ / UI 層"]
        GS[(Google Sheets\nDB 兼 操作UI)]
    end

    subgraph AUTOSEND["自動送信 ⚠️ HIGH RISK"]
        PBSEND[Phantombuster\nMessage Sender\nCookie流用・自動DM]
    end

    subgraph OUTPUT["送信"]
        LI[LinkedIn\n自動DM\n⚠️ BAN高リスク]
    end

    PB -->|プロフィールデータ| GS
    GS -->|候補者データ| N8N
    N8N -->|生成メッセージ書き戻し| GS
    GS -->|送信リスト| PBSEND
    PBSEND -->|一括自動送信| LI

    style PB fill:#FEE2E2,stroke:#EF4444
    style LIN_SRC fill:#FEE2E2,stroke:#EF4444
    style PBSEND fill:#FEE2E2,stroke:#EF4444
    style N8N fill:#D1FAE5,stroke:#10B981
    style CLAUDE fill:#D1FAE5,stroke:#10B981
    style GS fill:#FEF9C3,stroke:#CA8A04
    style LI fill:#7F1D1D,color:#fff,stroke:#7F1D1D
```

> 🚨 **HIGH RISK**：LinkedIn ToS 違反。週100件超でアカウント停止。T3メインアカウントでの本番利用不可。PoC・テスト用途限定。

**カスタム開発スコープ：n8nワークフロー のみ**

---

## パターン比較サマリー

| | A | B | C | D | E | F |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| データソース | BD Dataset | BD Dataset + Scraper | BD + AUTOHUNT | BD Dataset + Scraper | Phantombuster | Phantombuster |
| 差分更新 | ✗ | ✓ | ✓ | ✓ | ✓ | ✓ |
| DM送信 | 手動 | 手動 | 自動（AH） | 手動 | 手動 | 自動（PB）⚠️ |
| 専用UI | ✗ | ✗ | AH UI | ✓ Web UI | ✗ | ✗ |
| カスタムDB | ✗ | ✗ | ✗ | ✓ | ✗ | ✗ |
| カスタムサーバー | ✗ | ✗ | ✗ | ✓ | ✗ | ✗ |
| マルチテナント | ✗ | ✗ | △ | ✓ | ✗ | ✗ |
| ToSリスク | 低 | 低 | 低〜中 | 低 | 中 | 高 |
