%% ============================================================
%% T3 Recruit Automation — 全体アーキテクチャ図
%% ============================================================

flowchart TB

    %% ── ユーザー ──
    subgraph USER["👤 ユーザー層"]
        direction LR
        U1["採用担当者\n（クライアント企業）"]
        U2["T3管理者\n（admin）"]
    end

    %% ── フロントエンド ──
    subgraph FE["🖥️ フロントエンド  /  Next.js + TypeScript"]
        direction LR
        FE1["ユーザー画面 6画面\nログイン / TOP / プロンプト設定\n送信時間設定 / ログ閲覧 / 会社設定"]
        FE2["管理画面 4画面\n管理者ログイン / テナント一覧\nテナント作成 / ジョブ監視"]
    end

    %% ── バックエンド ──
    subgraph BE["⚙️ バックエンド  /  Next.js API Routes"]
        direction TB
        BE1["認証 / セッション管理\nNextAuth.js + JWT"]
        BE2["テナント管理 API\nCRUD + 状態制御 + 上限管理"]
        BE3["n8n ゲートウェイ\nワークフロー操作のラッパー"]
        BE4["ログ集約 API\nSheets → DB への取り込み"]
        BE5["APIキー管理\nAES-256 暗号化保存"]
    end

    %% ── DB ──
    subgraph DB["🗄️ DB  /  PostgreSQL（Supabase）"]
        direction LR
        DB1["tenants\nテナント情報・状態・上限"]
        DB2["job_logs\nジョブ実行履歴"]
        DB3["api_keys\n暗号化APIキー"]
        DB4["send_logs\n送信ログ（Sheetsから移行）"]
    end

    %% ── n8n ──
    subgraph N8N["🔄 n8n Cloud  /  ワークフローエンジン"]
        direction TB
        N1["ワークフロー群\n（テナント × 媒体ごとにコピー）\n例: T3_新卒_OfferBox\n    A社_新卒_OfferBox\n    A社_中途_BizReach"]
        N2["Cron Trigger\n送信スケジュール管理"]
        N3["Webhook\nRPA ↔ n8n 通信"]
        N4["n8n Credentials\nAPIキー・アカウント情報"]
    end

    %% ── AI ──
    subgraph AI["🤖 AI層"]
        direction LR
        AI1["Claude API\nレジュメ解析\n候補者分類\nメッセージ生成（統合処理）"]
        AI2["Gemini API\n品質スコアリング\n0–100点"]
    end

    %% ── RPA ──
    subgraph RPA["🤖 RPA層  /  RoboCorp Cloud"]
        direction TB
        R1["ブックマーク取得\n対象候補者リスト"]
        R2["レジュメスクレイピング\n1件ずつ取得"]
        R3["メッセージ自動送信\nIPローテーション込み"]
    end

    %% ── 採用媒体 ──
    subgraph MEDIA["🌐 採用媒体"]
        direction LR
        M1["OfferBox（新卒）"]
        M2["BizReach（中途）\nPhase2"]
        M3["Wantedly / Green\nPhase2"]
    end

    %% ── Google Workspace ──
    subgraph GWS["📊 Google Workspace"]
        direction LR
        G1["Spreadsheets\n承認待ちビュー\n禁止ワード辞書\n送信履歴ログ（Phase1）"]
    end

    %% ── 接続 ──
    U1 -->|操作| FE1
    U2 -->|操作| FE2

    FE1 -->|API呼び出し| BE
    FE2 -->|API呼び出し| BE

    BE1 --> DB1
    BE2 --> DB1
    BE3 -->|n8n REST API| N1
    BE3 -->|activate/deactivate| N2
    BE4 --> DB2
    BE4 --> DB4
    BE5 --> DB3

    N1 -->|HTTP Request| AI1
    N1 -->|HTTP Request| AI2
    N1 <-->|Webhook| R1
    N2 -->|Cron起動| R1
    N1 -->|Sheets API| G1

    R1 --> R2
    R2 -->|レジュメデータ| N3
    N3 --> N1
    N1 -->|生成メッセージ| R3

    R2 -->|スクレイピング| M1
    R3 -->|送信| M1
    R3 -.->|Phase2| M2
    R3 -.->|Phase2| M3

    %% スタイル
    classDef userStyle fill:#EFF6FF,stroke:#3B82F6,stroke-width:2px
    classDef feStyle fill:#F0FDF4,stroke:#22C55E,stroke-width:2px
    classDef beStyle fill:#FFFBEB,stroke:#F59E0B,stroke-width:2px
    classDef dbStyle fill:#FEF2F2,stroke:#EF4444,stroke-width:2px
    classDef n8nStyle fill:#F5F3FF,stroke:#8B5CF6,stroke-width:2px
    classDef aiStyle fill:#ECFDF5,stroke:#10B981,stroke-width:2px
    classDef rpaStyle fill:#FFF7ED,stroke:#F97316,stroke-width:2px

    class USER userStyle
    class FE feStyle
    class BE beStyle
    class DB dbStyle
    class N8N n8nStyle
    class AI aiStyle
    class RPA rpaStyle
