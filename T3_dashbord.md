flowchart TB

    subgraph P0["フェーズ 0　目標データ投入"]
        SS["Google スプレッドシート\n────────────\nデータ: tgt / gpu / tv / tr\n役割: 期初の目標値マスター\n操作者: 管理者が手入力\n技術: Google Sheets API v4"]
    end

    subgraph P1["フェーズ 1　バッチ取込 (n8n)"]
        N8N_B["n8n バッチ ワークフロー\n────────────\nホスト: n8n Cloud または Self-hosted\nトリガー: Cron（期初・定期）\n役割: SS → DB 目標値 取込\n技術: Google Sheets ノード\n      + PostgreSQL ノード"]
        N8N_M["n8n 手動トリガー ワークフロー\n────────────\nトリガー: 管理者が手動実行\n役割: 確定月実績を SS → DB 上書き\n技術: Webhook ノード\n      + PostgreSQL UPDATE ノード"]
        SS -->|"① 期初バッチ実行\nSheetsAPI 読取"| N8N_B
        SS -->|"② 手動トリガー実行\nSheetsAPI 読取"| N8N_M
    end

    subgraph P2["フェーズ 2　データベース層"]
        MT["monthly_targets\n────────────\nDB: PostgreSQL（未定 ⚠️要確認）\nカラム: staff_id / month\n        tgt / gpu / tv★ / tr★\nINDEX: staff_id + month"]
        MA["monthly_actuals\n────────────\nDB: PostgreSQL（未定 ⚠️要確認）\nカラム: staff_id / month\n        act / ag / vis\n        enq★ / est★\n        st: done/active/future"]
        CAL["staff_calendars ⚠️テーブル未定義\n────────────\nDB: PostgreSQL\nカラム: staff_id / month\n        off_days: JSON配列\n        working_days: INT"]
        SIM["staff_sim_settings ⚠️テーブル未定義\n────────────\nDB: PostgreSQL\nカラム: staff_id / month\n        sim_vals: JSONB\n        sim_start_day: DATE"]
        N8N_B -->|"③ INSERT/UPDATE\nmonthly_targets"| MT
        N8N_M -->|"④ UPDATE\nmonthly_actuals"| MA
    end

    subgraph P3["フェーズ 3　API層"]
        API1["GET /api/monthly/:staff_id\n────────────\nランタイム: Next.js API Routes\nホスト: Vercel（未定 ⚠️要確認）\nレスポンス: M配列 12ヶ月分\n           + tv★/tr★/enq★/est★\n認証: JWT / Supabase Auth（未定）"]
        API2["PUT /api/actuals/:staff_id\n────────────\nランタイム: Next.js API Routes\n役割: 実績5項目 保存\nBody: vis/enq/est/cl/ag\nトリガー: onchange → 即時PUT"]
        API3["PUT /api/calendar/:staff_id ⚠️未定義\n────────────\nランタイム: Next.js API Routes\n役割: 休み設定 保存\nトリガー: onchange → 3秒debounce\nBody: off_days JSON配列"]
        API4["PUT /api/sim/:staff_id ⚠️未定義\n────────────\nランタイム: Next.js API Routes\n役割: 努力KPI設定 保存\nトリガー: onchange → 3秒debounce\nBody: simVals JSONB"]
        MT --> API1
        MA --> API1
        CAL --> API3
        SIM --> API4
    end

    subgraph P4["フェーズ 4　個人画面（スタッフ）"]
        S_CAL["📅 カレンダーサブタブ\n────────────\n技術: Next.js + React\nUI: 7×6グリッド カレンダー\n操作: タップで休み⇔出勤切替\n連動: recalcWD() 残り営業日再計算\n保存: onchange → PUT /api/calendar"]
        S_ACT["📊 今日サブタブ\n────────────\n技術: Next.js + React\nUI: ファネル / BN診断 / 目標指標\n操作: 実績5項目 手入力\n保存: onchange → PUT /api/actuals\n計算: 全てクライアントサイドJS"]
        S_SIM["📐 SIMサブタブ\n────────────\n技術: Next.js + React\nUI: SIMカレンダー + KPI入力\n操作: 転換率3値 / 台粗利 入力\n保存: onchange → PUT /api/sim\n計算: 着地SIM / 改善プラン（クライアントJS）"]
        API1 -->|"⑤ 初期データ読込\nJSON レスポンス"| S_CAL
        API1 --> S_ACT
        API1 --> S_SIM
        S_CAL -->|"⑥ 3秒 debounce\n⚠️自動保存フロー未定義"| API3
        S_ACT -->|"⑦ 即時 PUT"| API2
        S_SIM -->|"⑧ 3秒 debounce\n⚠️自動保存フロー未定義"| API4
        API2 --> MA
        API3 --> CAL
        API4 --> SIM
    end

    subgraph P5["フェーズ 5　店長画面"]
        M_CAL["店舗カレンダービュー\n────────────\n技術: Next.js + React\nデータソース: staff_calendars\n表示: 全メンバーの稼働日\n⚠️同期タイミング未定義\n（リアルタイム? ページ読込時?）"]
        M_KPI["メンバー別KPIビュー\n────────────\n技術: Next.js + React\nデータソース: staff_sim_settings\n表示: 個人設定KPI・着地予想\n⚠️同期タイミング未定義"]
        M_DD["ドリルダウン閲覧\n────────────\n技術: Next.js Router\n操作: 店長が個人画面を閲覧専用で表示\n認証: role=manager のみ許可"]
        API1 -->|"⑨ 初期データ読込"| M_CAL
        API1 --> M_KPI
        CAL -->|"⚠️連携フロー未定義"| M_CAL
        SIM -->|"⚠️連携フロー未定義"| M_KPI
        M_DD -.->|"⑩ 閲覧専用\nread-only"| S_ACT
    end

    subgraph P6["フェーズ 6　エリア長・事業部長画面 ⚠️未実装"]
        A_VIEW["エリア長画面\n────────────\n技術: Next.js + React（未実装）\nデータ: 各店長KPI + 店舗着地予想集計\n⚠️店長→エリア長の\n  ロールアップ設計未定義"]
        E_VIEW["事業部長画面\n────────────\n技術: Next.js + React（未実装）\nデータ: 各エリア長KPI + エリア着地予想集計\n⚠️エリア長→事業部長の\n  ロールアップ設計未定義"]
        M_KPI -->|"⚠️連携フロー未定義"| A_VIEW
        A_VIEW -->|"⚠️連携フロー未定義"| E_VIEW
    end

    classDef p0 fill:#0f172a,stroke:#6366f1,color:#c7d2fe
    classDef p1 fill:#0c1a0c,stroke:#16a34a,color:#bbf7d0
    classDef p2 fill:#0c1220,stroke:#2563eb,color:#bfdbfe
    classDef p3 fill:#14290e,stroke:#15803d,color:#bbf7d0
    classDef p4 fill:#1a1a2e,stroke:#7c3aed,color:#ddd6fe
    classDef p5 fill:#0f1f2e,stroke:#0ea5e9,color:#bae6fd
    classDef p6 fill:#1a1a10,stroke:#ca8a04,color:#fef08a
    classDef warn stroke:#ef4444,stroke-width:2px,stroke-dasharray:4 3

    class SS p0
    class N8N_B,N8N_M p1
    class MT,MA p2
    class CAL,SIM warn
    class API1,API2 p3
    class API3,API4 warn
    class S_CAL,S_ACT,S_SIM p4
    class M_CAL,M_KPI,M_DD p5
    class A_VIEW,E_VIEW p6
