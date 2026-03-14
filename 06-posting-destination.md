# 投稿先の検討（Issue vs Discussion）と既存スレッドの確認

opentofu のリポジトリーは issue と discusssion の2つをオープンしているようです。

* これを投稿するのは issue と discussion のどちらが適切でしょうか。
* 投稿先によって、変更が好ましい場所があったら教えてください。
* 既存の issue や discussion で、既にこの話を取り扱っている投稿がないか確認してください。

## 1. Issue と Discussion のどちらが適切か

**結論: Issue を推奨します。**

### 理由

| 観点 | Issue | Discussion |
|------|--------|------------|
| 前例 | 本件の根拠である「Docker イメージ方針」は **#1931** で議論・決定されている。方針の表明やドキュメント変更は Issue 上で行われた。 | 方針の「再検討を願い出る」だけなら Discussion でも可能だが、OpenTofu では同種の話題が Issue で扱われている。 |
| 性質 | ドキュメント／文言の**変更提案**であり、メンテナーが「対応する・しない・後で検討」を判断し、ラベル・マイルストーンで追跡しやすい。 | オープンな意見交換や「みんなどう思う？」に向く。本件は「検討をお願いしたい」という明確な依頼なので、Issue の方が依頼が届きやすい。 |
| 一般的な使い分け | バグ・改善・ドキュメント修正など、**具体的なタスクや結果**を追跡する。 | ブレスト・合意形成・Q&A など、**即座のアクションにしない会話**向け。 |

「direct usage is no longer supported の文言・方針の再検討」は、#1931 の延長線上にある**ドキュメント・方針の提案**なので、**同じリポジトリの Issue として出すのが一貫しており、メンテナーも対応しやすい**と考えられます。

Discussion は「まずコミュニティの意見を広く集めたい」場合に選ぶ選択肢です。本件は「メンテナーに検討を願い出る」形なので、まずは Issue が適切です。

---

## 2. 投稿先による変更が好ましい点

### Issue に投稿する場合（推奨）

- **現状の 05-reviewed-by-agent.md の本文をそのまま使って問題ありません。**
- 次のどちらかを検討すると、既存の文脈が伝わりやすくなります。
  - **タイトル（Subject）の直下**に、短い前置きを 1 行足す:
    - 例: `This request relates to the policy decided in #1931. I would like to ask the maintainers to reconsider how it is documented and applied (including the scope of "direct usage").`
  - または、本文の「The background for this change appears to be described in issue https://...」の部分で、**#1931 へのリンクを明示**したまま（現状どおり）にしておくだけでも十分です。
- Issue の**タイトル**は、現在の Subject のまま  
  `Request to reconsider direct usage of the official Docker images`  
  でよいです。必要なら `(documentation wording)` などを末尾に足してもよいですが、必須ではありません。

### Discussion に投稿する場合

- **General** または **Ideas** カテゴリが向いています。
- 次のようにすると「議論を促す」形になります。
  - **冒頭**: 短く状況と目的を書く。  
    例: `The documentation states that direct usage of the official Docker images is no longer supported (since 1.10). I would like to ask whether the maintainers would consider revisiting how this is communicated, and to hear other users’ experiences.`
  - **末尾**: 必要なら  
    `If the maintainers prefer to track this as an issue, I am happy to open one.`  
    と書くと、Issue に移す判断をしやすくなります。
- 本文の提案（limitation 表記や文言の整理など）は、Discussion でもそのまま使えます。Discussion は「意見を集める」場なので、提案を少し「問いかけ」に寄せてもよいです（例: "Would it be possible to describe this as a limitation rather than 'not supported'?"）。

---

## 3. 既存の Issue / Discussion の確認結果

### 本件を「直接」扱っている投稿

- **ありません。**  
  「公式イメージの direct usage を no longer supported としている方針・文言を、再検討してほしい」という趣旨の既存 Issue も Discussion も見つかっていません。

### 関連する既存スレッド（参照・言及用）

| 番号 | 種類 | 内容 | 本件との関係 |
|------|------|------|----------------|
| [#1931](https://github.com/opentofu/opentofu/issues/1931) | Issue (closed) | Docker イメージのベースイメージ更新要望 → 方針決定「ベースイメージとしての利用をやめ、ONBUILD で禁止。CLI としての利用は safe」 | **根拠となる議論**。本文で必ず触れ、リンクする。本件は「#1931 の範囲（ベースイメージ）と、現行ドキュメントの "direct usage is no longer supported" のギャップの再考」として新規投稿でよい。 |
| [#2728](https://github.com/opentofu/opentofu/issues/2728) | Issue (closed) | マルチステージビルドで ONBUILD が発火してビルドが失敗する → 解決策は `:minimal` を利用 | **別トピック**（マルチステージの回避策）。文言・方針の再検討は扱っていない。 |
| [#1993](https://github.com/opentofu/opentofu/pull/1993), [#1994](https://github.com/opentofu/opentofu/pull/1994) | PR | #1931 の実装（ONBUILD 警告・禁止） | 経緯の参照用。必要に応じて言及。 |

### Discussion の検索結果

- 「direct usage」「no longer supported」「Docker image policy」で、本件と同一趣旨の Discussion は**見当たりませんでした**。  
  （OpenTofu の Discussion は `github.com/orgs/opentofu/discussions` で、General / Ideas / Q&A / Show and tell などがあります。）

---

## 4. 投稿時の注意（共通）

- 投稿本文は **05-reviewed-by-agent.md の「---」より前**（Subject から Thank you まで）を使用し、ファイル末尾の「レビュー・推敲メモ」は**投稿しない**でください。
- Issue を選ぶ場合、**#1931** を本文中で参照しておくと、メンテナーが経緯を追いやすくなります（現状の 05 の記載で参照済みです）。
