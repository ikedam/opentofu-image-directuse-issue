Subject: Request to reconsider direct usage of the official Docker images

Hi,

I found the following notice in the OpenTofu documentation (https://opentofu.org/docs/intro/install/docker/):

> Starting with OpenTofu 1.10, direct usage of the official images is no longer supported.

As I understand it, this means that running OpenTofu via `docker run ghcr.io/opentofu/opentofu` is no longer recommended or supported, and that we need to build our own image instead.

I feel that having runnable official images is a considerable advantage for many users and for adoption of open source projects. The current wording can feel restrictive, especially to people who are evaluating or introducing OpenTofu.

The background for this change appears to be described in issue https://github.com/opentofu/opentofu/issues/1931. My understanding of the reasons for no longer supporting direct usage is:

* The project has limited resources to keep the base image (Alpine) updated, so it is not updated frequently.
* This can lead to security concerns.
* This can also lead to outdated CA certificates.

I also note that in the discussions around #1931 and the related PRs (#1993, #1994), the focus was on discouraging use of the image as a base image, and it was stated that using the image as a CLI tool is safe. The current documentation, however, reads as if all direct usage (including `docker run`) is unsupported. I am not sure whether the scope was intentionally broadened or if there is room to align the documentation with that earlier distinction.

I would like to ask whether the maintainers would consider the following, only if they find it feasible:

* Documentation and wording
  * Instead of "not supported," describing this as a limitation: for example, that the base image may not always be up to date; that images for a given OpenTofu version are not updated after release (e.g. no rebuilds with a newer base); that building your own image is recommended especially in security-sensitive environments; and that the base image may change in future versions.
  * If the intent is only to discourage use as a base image, possibly clarifying that in the docs so that simple CLI use (e.g. `docker run`) is not discouraged in the same way.

* Maintenance of the image
  * If the team is open to it, semi-automating base image updates (e.g. via Dependabot for Docker: https://docs.github.com/en/code-security/reference/supply-chain-security/supported-ecosystems-and-repositories#docker), or using a `latest` tag for the base, with a simple check (e.g. `tofu --version`) in CI to ensure the image still runs. I understand this may still add maintenance burden and may not be acceptable.

I am aware that the project has limited resources and that this request may be asking too much of the core team. The decision is entirely yours. I would be grateful if you could consider this as a gentle request for reconsideration, in the hope that it might improve the experience for users who rely on or would like to use the official images directly.

Thank you for your work on OpenTofu.

---

## レビュー・推敲メモ（執筆者向け）

* **英語の修正**: 主語と動詞の一致（"OpenTofu ... supports"）、"are followings" → "is as follows"、冠詞（"a too strong word" → "too strong a word"）、"opensource" → "open source"、"this ask/result" の動詞修正、その他自然な表現に整えました。
* **トーン**: 決定権はメンテナーチームにあることを明示し、「検討をお願いしたい」という依頼形にし、「押し付けがましくない」ように、提案は "only if they find it feasible" や "I would be grateful if you could consider" にしました。
* **事実の整合**: 
  * 「we can no longer run」は、技術的には `docker run` は可能でサポート方針の問題であるため（01, 03）、「is no longer recommended or supported」のように変更しました。
  * 01-context-from-public.md の「#1931/#1993/#1994 ではベースイメージ禁止と CLI 利用は safe とされ、現行ドキュメントは direct usage 全体がサポート外と読める」というギャップを踏まえ、その点に触れつつ「意図的に範囲を広げたのか、文言をベースイメージに限定する余地があるか」を丁寧に尋ねる形を追加しました。
* **提案の整理**: 「Not supported より Limitation の記載に」と「ベースイメージの半自動更新」の2本立ては維持しつつ、順序を「まず文言・ドキュメントの検討」「その上で運用の負荷が許すなら」とし、Dependabot / `latest` は「可能であれば」の提案として弱めました。
* **末尾**: 感謝と「決定はチームに委ねる」旨を短く追加しました。
