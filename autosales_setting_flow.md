# 営業フィードバックAIエージェント 初期登録フロー

```mermaid
flowchart TD
    classDef userTask fill:#fffbe6,stroke:#d4a017,stroke-width:1px;
    classDef systemTask fill:#f0f7ff,stroke:#0052cc,stroke-width:1px;
    classDef mailA fill:#e6fffb,stroke:#13c2c2,stroke-width:2px;
    classDef mailB fill:#f6ffed,stroke:#52c41a,stroke-width:2px;
    classDef mailC fill:#fff0f6,stroke:#eb2f96,stroke-width:2px;

    %% ===== フロー① 起点 =====
    F1(["フロー①\n営業担当が手動 または 代理入力\n参画フォーム送信\nプランの種類・契約数を回収"]):::userTask

    F1 --> MA

    %% ===== メールA =====
    subgraph MA["メールA　送信タイミング：参画フォーム回答時　To アカウント担当者"]
        direction LR
        MA1["商談フィードバック系\n・対象者入力リスト\n・応酬話法URL"]:::mailA
        MA2["数値レポート系\n・組織図入力\n・レポート対象者リスト入力"]:::mailA
    end

    MA --> SYS1 & SYS2

    %% ===== 商談フィードバック系 =====
    subgraph SYS1["サービス① 商談フィードバック系"]
        direction TB
        G1[アカウント担当者が\n対象者・応酬話法URLを入力]:::userTask
        G1 --> G2[商談フィードバックフォームを生成]:::systemTask
        G2 --> MB

        MB(["メールB\n商談フィードバックURL作成完了通知\nTo アカウント担当者\nタイミング：日次・URL作成完了時"]):::mailB

        MB --> MG[アカウント担当者が\nURLを対象者へ共有]:::userTask
        MG --> CF["商談フォームへの\n回答（随時）"]:::userTask
        CF --> MB2

        MB2(["メールB-2\n商談フィードバックレポート配信\nTo 本人・上長\nタイミング：商談フォーム回答後"]):::mailB
    end

    %% ===== 数値レポート系 =====
    subgraph SYS2["サービス② 数値レポート系"]
        direction TB
        H1[アカウント担当者が\n組織図・対象者リストを入力]:::userTask
        H1 --> MC

        MC(["メールC\n年間個人目標・昨年個人数値の回収依頼\nTo アカウント担当者　CC T3\nタイミング：要確認"]):::mailC

        MC --> H2[アカウント担当者が\n年間目標・昨年実績を入力]:::userTask
        H2 --> H3[季節指数算出\n通期目標の反映]:::systemTask

        H3 --> MC2

        MC2(["メールC-2\n月次個人実績の入力依頼\nTo 店舗責任者　CC T3\nタイミング：毎月1日"]):::mailC

        MC2 --> H4[店舗責任者が\n月次実績を入力]:::userTask
        H4 --> H5[実績・GAPデータ・達成率を反映\n全ユーザー平均値算出]:::systemTask
        H5 --> MC3

        MC3(["メールC-3\n月次・四半期・半期・年次レポート配信\nTo 各種責任者　CC T3\nタイミング：毎月10日"]):::mailC

        H5 --> MC4

        MC4(["メールC-4\n年間目標入力依頼\nTo アカウント担当者　CC T3\nタイミング：期初1日"]):::mailC

        MC4 --> H2
    end
```

---

## メール一覧

| メール名 | メールで通知する内容 | 宛先 | タイミング |
|---|---|---|---|
| メールA | ①商談FB系：対象者入力リスト・応酬話法URL<br>②数値レポート系：組織図入力・レポート対象者リスト入力 | アカウント担当者 | 参画フォーム回答時 |
| メールB | 商談フィードバックURL作成完了通知 | アカウント担当者 | 日次起動・URL作成完了時 |
| メールB-2 | 商談フィードバックレポート | 本人 / 上長 | 商談フォーム回答後・フィードバック生成完了時 |
| メールC | 年間個人目標の回収依頼、昨年個人数値の回収依頼 | アカウント担当者（CC: T3） | 要確認 |
| メールC-2 | 月次個人実績の入力依頼 | 店舗責任者（CC: T3） | 毎月1日 |
| メールC-3 | 月次・四半期・半期・年次レポート完成通知 | 各種責任者（CC: T3） | 毎月10日 |
| メールC-4 | 年間目標入力依頼 | アカウント担当者（CC: T3） | 期初1日 |
