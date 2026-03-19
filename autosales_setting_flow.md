# 営業フィードバックAIエージェント 初期登録フロー

```mermaid
flowchart TD
    classDef userTask fill:#fffbe6,stroke:#d4a017,stroke-width:1px;
    classDef systemTask fill:#f0f7ff,stroke:#0052cc,stroke-width:1px;
    classDef mailTask fill:#f6ffed,stroke:#52c41a,stroke-width:1px;
    classDef section fill:#fafafa,stroke:#999,stroke-width:2px,stroke-dasharray:5 5;
    classDef monthly fill:#fff0f6,stroke:#eb2f96,stroke-width:1px;
    classDef yearly fill:#f9f0ff,stroke:#722ed1,stroke-width:1px;

    %% ===== 起点 =====
    F1(["フロー①\n営業担当が手動 または 代理入力\n参画フォーム送信\n（プランの種類・契約数を回収）"]):::userTask

    F1 --> S1 & S2

    %% ===== 商談フィードバック系 =====
    subgraph S1["サービス① 商談フィードバック系 初期設定"]
        direction TB
        F2(["フロー②\nメールA 送信\n組織図を入力\n全エリア・全店舗を入力依頼\n担当：アカウント担当者"]):::mailTask
        F3(["フロー③\nメールB 送信\nトリガー：日次起動\nエリア/店舗の責任者・アドレス・\nレポート対象を回収\n担当：アカウント担当者"]):::mailTask
        F2 --> F3
    end

    %% ===== 数値フィードバック系 =====
    subgraph S2["サービス② 数値フィードバック系 初期設定"]
        direction TB

        F5(["フロー⑤\nメールA 送信\n個人IDの数量に合わせて\n入力リストを生成"]):::mailTask

        F5 --> F6A & F6B & F6C

        subgraph LG["フロー⑥ リンク生成（メールCで閲覧リンク共有）"]
            direction LR
            F6A["個人リンク生成"]:::systemTask
            F6B["店舗リンク生成"]:::systemTask
            F6C["エリアリンク生成"]:::systemTask
        end

        F6A & F6B & F6C --> INIT

        subgraph INIT["初回のみの動作"]
            direction TB
            F6D(["フロー⑥（初回のみ）\n昨年の年間実績を入力\n6項目 × 従業員人数 × 12ヶ月\n担当：各店舗店長"]):::userTask
            F6E(["フロー⑥\nメールB 送信\n年間の個人目標を入力\n6項目 × 営業人数 × 12ヶ月\n担当：各店舗店長"]):::mailTask
            F6D --> F6E
        end

        F6E --> SI[昨年データをもとに季節指数算出]:::systemTask
        SI --> TG[通期分の目標を反映]:::systemTask
    end

    %% ===== 月次の動作 =====
    TG --> MONTHLY

    subgraph MONTHLY["月次の動作"]
        direction TB
        F7(["フロー⑦\nトリガー：フォーム回答\n月次実績を入力\n6項目 × 営業人数\n担当：各店舗店長"]):::userTask
        F7 --> R1[月次実績データ反映\n目標GAPデータ反映\n達成率データ反映]:::systemTask
        R1 --> R2[全ユーザーの平均数値算出・反映]:::systemTask
        R2 --> F8(["フロー⑧\nトリガー：月次起動\n各社の事業年度をもとに\n月次・四半期・半期・年次を\nメールで通知"]):::mailTask
    end

    %% ===== 年次の動作 =====
    MONTHLY --> YEARLY

    subgraph YEARLY["年次の動作"]
        direction TB
        Y1[通期目標 回収・反映]:::yearly
        Y1 --> Y2[昨期データから季節指数を再算出]:::yearly
        Y2 --> Y3[次年度の月次サイクルへ移行]:::yearly
    end
```

---

## フロー番号サマリー

| フロー | トリガー | 担当 | 内容 |
|---|---|---|---|
| ① | 営業担当（手動） | 営業担当 / アカウント担当 | 参画フォーム送信。プランの種類・契約数を回収 |
| ② | フロー①の完了 | アカウント担当者 | メールAで組織図入力依頼。全エリア・全店舗を登録 |
| ③ | 日次起動 | アカウント担当者 | メールBでエリア/店舗の責任者・アドレス・レポート対象を回収 |
| ⑤ | フロー①の完了 | システム | 個人IDの数量に合わせて入力リストを自動生成 |
| ⑥（初回のみ） | フロー⑤の完了 | 各店舗店長 | 昨年年間実績を入力（6項目×従業員人数×12ヶ月）|
| ⑥（目標入力） | フロー⑤の完了 | 各店舗店長 | メールBで年間個人目標を入力（6項目×営業人数×12ヶ月） |
| ⑥（リンク共有） | フロー⑤の完了 | システム | メールCで個人/店舗/エリアの閲覧リンクを生成・共有 |
| ⑦ | フォーム回答 | 各店舗店長 | 月次実績を入力（6項目×営業人数） |
| ⑧ | 月次起動 | システム | 各社の事業年度をもとに月次・四半期・半期・年次でメール通知 |
