# 「direct usage of the official images is no longer supported」の経緯・クロスチェック

## プロンプト

コンテキスト: [@README.md](README.md)

https://opentofu.org/docs/intro/install/docker/ に、

> direct usage of the official images is no longer supported

という記述が追加されました。

opentofu の利用の大きな制約になるため、この記述の意図、理由、経緯を把握したいです。
ただ、私が見た限り、この記述についての経緯が分かる資料が見当たりませんでした。

この調査のクロスチェックを行ってください。

関連すると思われる情報は以下です:
* https://github.com/opentofu/opentofu/pull/3656 
* https://github.com/opentofu/opentofu/pull/2942 
* https://github.com/opentofu/opentofu/issues/2740 

このチャットの内容は、記録に残すため、[@01-context-from-public.md](01-context-from-public.md) に記録していってください。

## 1. 結論（クロスチェック要約）

- **意図・理由**は Issue [#1931](https://github.com/opentofu/opentofu/issues/1931) に明記されている：**ベースイメージとしての利用**をやめたい（リソース不足・セキュリティリスク）。
- **経緯**は #1931 → PR [#1993](https://github.com/opentofu/opentofu/pull/1993)（1.9）→ PR [#1994](https://github.com/opentofu/opentofu/pull/1994)（1.10）→ ドキュメント更新 PR [#2942](https://github.com/opentofu/opentofu/pull/2942)（v1.10）→ [#3656](https://github.com/opentofu/opentofu/pull/3656)（main へのポート）で追える。
- **注意点**：#1931 / #1993 / #1994 では「**CLI としての利用（例: `docker run`）は safe**」と書かれており、**ベースイメージ利用の禁止**が主目的だった。一方、現在のドキュメントは「**direct usage** is no longer supported」「`docker run ghcr.io/opentofu/opentofu` を使っていた場合は自分でイメージをビルドすること」と記載しており、**文言としては「直接利用全体」をサポート外**と読める。経緯が分かる一次資料では「ベースイメージ禁止」までが明示されており、「docker run もサポート外」と明言した議論は見当たらない。

---

## 2. 意図・理由（Issue #1931）

- **#1931**「Dockerimage update base image」で、Alpine バージョン更新の要望を受けて、コアチームが方針を返答（2024-09-05）。
- 要点：
  - **ベースイメージとして他者がビルドに使うことを維持するリソースがない。**
  - 安全なベースイメージとするには、少なくとも週次でパッケージ更新を出し続ける必要があり、その負荷を負えない。
  - ベースイメージのまま使うと、通常は深刻でないセキュリティ問題も、その上で Web サービスなどを動かすとクリティカルになり得る。「そのリスクは取りたくない」。
  - そのため「**ベースイメージとしての利用**」をやめ、**自分でイメージをビルドする手順・例をドキュメントに書く**方針。
- 具体的なステップとして以下を表明：
  1. `ghcr.io/opentofu/opentofu:1` タグの修正
  2. OpenTofu 1.9.0 で最新安定 Alpine へ切り替え
  3. **OCI イメージのインストール説明に「ベースイメージとして使わないこと」の警告と、カスタムイメージのビルド例を追加**
  4. 1.9.0 から ONBUILD で「ベースイメージとしての利用」に対する**警告**
  5. **1.10.0 から ONBUILD でベースイメージとして使った場合は `exit 1` でビルド失敗**

→ 経緯が分かる**一次資料では「ベースイメージとしての使用を禁止する」**ことが目的であり、「docker run のみの利用」を禁止すると明言した記述はない。

---

## 3. 実装の経緯（PR #1993, #1994）

- **#1993**（1.9.0）：ベースイメージ利用の**非推奨化**を実装。
  - 理由：セキュリティ。安全なベースイメージを維持するには少なくとも週次リリースが必要で、その余力がない。代わりにドキュメントでカスタムイメージのビルド方法を案内。
  - **注記**：「The OpenTofu image is **safe to use** if you are using it as a **CLI tool**.」（CLI ツールとして使う分には安全に使ってよい）
- **#1994**（1.10.0）：`ghcr.io/opentofu/opentofu` を**ベースイメージとして使うことを禁止**（ONBUILD で `exit 1`）。
  - 同じ注記：「OpenTofu image is **safe to use** if you are using it as a **CLI tool**.」

→ 実装と PR 説明は一貫して「**ベースイメージとしての使用禁止**」と「**CLI としての利用は safe**」であり、「direct usage 全体をサポートしない」とは書いていない。

---

## 4. ドキュメントの変更（PR #2942, #3656）

- **#2942**「Updated what's new, docker install guides, added OTel docs」（v1.10、2025-06 マージ）で、Docker まわりのインストール説明を「古い方法を削除」する形で更新（"Fixed docker installations to remove old methods"）。
- この結果、現在の [Docker インストールページ](https://opentofu.org/docs/intro/install/docker/) には次のようにある：
  - 「Previously, OpenTofu provided official Docker images that could be used **directly**. Starting with OpenTofu 1.10, **direct usage of the official images is no longer supported**.」
  - 「If you were previously using `docker run ghcr.io/opentofu/opentofu`, you will need to **build your own image** following the instructions below.」
- **#3656**（2026-01）：#2942 の変更を v1.10 から **main** にポートし、マイグレーション・インストールガイドを main と v1.10 で揃えた。

→ ドキュメント上は「**direct usage**」がサポートされなくなったこと、および **`docker run` で公式イメージをそのまま使っていたユーザーも「自分でイメージをビルドすること」** と読める表現になっている。  
→ #2740（Migration Guides are out of date）は **マイグレーションガイドの古い記述（#1204 など）の更新**が主で、Docker 方針の議論そのものではない。

---

## 5. 技術的な事実（イメージの挙動）

- 1.10 で導入されたのは「**FROM でこのイメージをベースにしたときに ONBUILD で exit 1 する**」という挙動。
- そのため **`docker run ghcr.io/opentofu/opentofu` のような「直接実行」は、現状のイメージだけ見れば技術的には可能**（ONBUILD はビルド時にしか発火しない）。
- ただし **マルチステージビルドで `FROM ghcr.io/opentofu/opentofu:1.9.1` のように使うと 1.10 ではビルドが失敗する**（Issue [#2728](https://github.com/opentofu/opentofu/issues/2728)）。解決策として **`ghcr.io/opentofu/opentofu:minimal`（および 1.9.1-minimal などのタグ）** でバイナリだけコピーする方法が案内されている。

→ 「direct usage is no longer supported」は、**技術的にイメージが動かなくなる**というより、**ドキュメント・サポート方針として「公式イメージの直接利用は推奨しない／サポートしない」と宣言した**と解釈できる。

---

## 6. 参照リンク一覧

| 資料 | URL | 内容 |
|------|-----|------|
| 方針の根拠 | https://github.com/opentofu/opentofu/issues/1931 | ベースイメージ維持のリソース不足・リスク、1.9/1.10 の ONBUILD 計画 |
| 1.9 実装 | https://github.com/opentofu/opentofu/pull/1993 | ベースイメージ非推奨、「CLI としての利用は safe」の注記 |
| 1.10 実装 | https://github.com/opentofu/opentofu/pull/1994 | ベースイメージ使用時のビルド失敗、「CLI は safe」の注記 |
| ドキュメント更新（v1.10） | https://github.com/opentofu/opentofu/pull/2942 | Docker インストールの書き換え、「direct usage no longer supported」等 |
| main へのポート | https://github.com/opentofu/opentofu/pull/3656 | #2942 の main へのバックポート |
| マイグレーションガイド | https://github.com/opentofu/opentofu/issues/2740 | マイグレーションガイド更新（Docker 方針の直接の議論ではない） |
| マルチステージと minimal | https://github.com/opentofu/opentofu/issues/2728 | マルチステージが 1.10 でブロックされる事象、:minimal の案内 |
| 現行ドキュメント | https://opentofu.org/docs/intro/install/docker/ | 上記文言の掲載箇所 |

---

## 7. 残っているギャップ（クロスチェックで分かったこと）

- **「direct usage」の定義**：一次資料（#1931, #1993, #1994）では「**ベースイメージとしての使用**」の禁止と「**CLI としての利用は safe**」が書かれているが、現行ドキュメントは「direct usage of the official images is no longer supported」および「`docker run` していた人も自分でイメージをビルドすること」としている。
- そのため、
  - **「direct usage」＝「ベースイメージとしての使用」に限定する意図だった**のか、
  - **のちに「docker run での直接利用もサポート外」と方針を広げた**のか、
  を区別する**公開された議論・Issue/PR は見当たらない**。
- プロジェクトに確認するなら、「direct usage」を「ベースイメージ利用のみ」と解釈してよいか、それとも「`docker run` 含むすべての直接利用をサポート外としているか」を問うとよい。

---

*このメモは公開情報（GitHub Issues/PR、opentofu.org ドキュメント）に基づく調査のクロスチェック結果である。*
