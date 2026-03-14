# OpenTofu が配布している Docker イメージの実装状況

## 1. 概要

OpenTofu の公式リポジトリ [opentofu/opentofu](https://github.com/opentofu/opentofu) には、配布用の Docker イメージをビルドするための **2 種類の Dockerfile** が存在する。

| Dockerfile | ベースイメージ | 用途 |
|------------|----------------|------|
| [Dockerfile](https://github.com/opentofu/opentofu/blob/main/Dockerfile) | `alpine:3.20` | フルイメージ（git, bash, openssh 含む） |
| [Dockerfile.minimal](https://github.com/opentofu/opentofu/blob/main/Dockerfile.minimal) | `scratch` | 最小イメージ（tofu バイナリのみ） |

配布は **GitHub Container Registry (GHCR)** のみで行われ、レジストリは `ghcr.io/opentofu/opentofu`。Docker Hub には公式イメージは存在しない。

---

## 2. 各 Dockerfile の内容

### 2.1 Dockerfile（Alpine ベース）

- **ベース**: `FROM alpine:3.20`
- **追加パッケージ**: `git`, `bash`, `openssh`（`apk add --no-cache`）
- **内容**: `/usr/local/bin/tofu` にバイナリをコピー
- **特記事項**:
  - **ONBUILD** で「OpenTofu 1.10 以降、このイメージをベースイメージとして使うことはサポートされていない」旨の警告を表示し、`ONBUILD RUN exit 1` でビルドを失敗させる（[Issue #1931](https://github.com/opentofu/opentofu/issues/1931) 等に基づく方針）
  - 直接 `docker run` で実行する用途では ONBUILD は発火しないが、公式ドキュメントでは「direct usage of the official images is no longer supported」として、自分でイメージをビルドする方法が案内されている

### 2.2 Dockerfile.minimal（scratch ベース）

- **ベース**: `FROM scratch`
- **内容**: `/usr/local/bin/tofu` にバイナリをコピーするだけ。シェル・パッケージマネージャ・OS ツールは含まない。
- **用途**: マルチステージビルドで `tofu` バイナリだけを自前イメージにコピーするための「バイナリ供給用」イメージとして公式ドキュメントで推奨されている。

---

## 3. ビルド・配布の仕組み

- **リリース手段**: GitHub Actions の [release.yml](https://github.com/opentofu/opentofu/blob/main/.github/workflows/release.yml)（`workflow_dispatch`、タグ指定で手動実行）
- **ビルドツール**: [GoReleaser](https://goreleaser.com/)（`.goreleaser.yaml` で Docker ビルド・マニフェスト・プッシュを定義）
- **マルチプラットフォーム**: `linux/amd64`, `linux/arm64`, `linux/arm`, `linux/386` をビルドし、マニフェストで 1 タグにまとめて配布

---

## 4. 各イメージの配布タグ

同一リポジトリ `ghcr.io/opentofu/opentofu` に対して、**フルイメージ用**と**minimal 用**の 2 系統のタグが付与される。

### 4.1 Alpine ベース（Dockerfile）のタグ

| タグの種類 | 例（バージョン 1.9.1 の場合） | 説明 |
|------------|------------------------------|------|
| 完全バージョン | `1.9.1` | そのリリースのフルイメージ |
| Major.Minor | `1.9` | 最新 1.9.x を指す（リリース時に更新） |
| Major | `1` | 最新 1.x を指す（リリース時に更新） |
| 最新 | `latest` | 最新安定版（リリース時に更新） |

- プレリリース版（例: `1.6.1-alpha1`）の場合は、`skip_push: auto` により **Major.Minor / Major / latest にはプッシュされない**（完全バージョンタグのみ）。

### 4.2 scratch ベース（Dockerfile.minimal）のタグ

| タグの種類 | 例（バージョン 1.9.1 の場合） | 説明 |
|------------|------------------------------|------|
| 完全バージョン | `1.9.1-minimal` | そのリリースの最小イメージ |
| Major.Minor | `1.9-minimal` | 最新 1.9.x の minimal |
| Major | `1-minimal` | 最新 1.x の minimal |
| 最新 | `minimal` | 最新安定版の minimal |

- 同様に、プレリリース時は Major.Minor / Major / minimal タグはプッシュされない（完全バージョン-minimal のみ）。

### 4.3 プラットフォーム別タグ（内部用）

GoReleaser は各アーキテクチャごとに中間タグでイメージをプッシュし、その後マニフェストで 1 つのタグにまとめている。

- **フルイメージ**: `{{ .Version }}-amd64`, `-arm64`, `-arm`, `-386`（例: `1.9.1-amd64`）
- **minimal**: `{{ .Version }}-minimal-amd64`, `-minimal-arm64`, `-minimal-arm`, `-minimal-386`（例: `1.9.1-minimal-amd64`）

ユーザーが通常使うのは、上記 4.1 / 4.2 のマニフェストタグ（プラットフォーム非依存）である。

---

## 5. 参照した一次資料

| 内容 | 参照先 |
|------|--------|
| Dockerfile（Alpine） | https://github.com/opentofu/opentofu/blob/main/Dockerfile |
| Dockerfile.minimal（scratch） | https://github.com/opentofu/opentofu/blob/main/Dockerfile.minimal |
| リリースワークフロー | https://github.com/opentofu/opentofu/blob/main/.github/workflows/release.yml |
| Docker ビルド・タグ定義 | https://github.com/opentofu/opentofu/blob/main/.goreleaser.yaml（`dockers` / `docker_manifests` セクション） |
| 公式 Docker 利用ドキュメント | https://opentofu.org/docs/intro/install/docker/ |

---

## 6. まとめ

- **2 種類の Dockerfile**: Alpine 3.20 ベースのフルイメージと、scratch ベースの minimal イメージ。
- **配布先**: `ghcr.io/opentofu/opentofu` のみ。タグは GoReleaser の `docker_manifests` により、バージョン・Major.Minor・Major・latest / minimal の形式で付与される。
- **フルイメージ**: `1.9.1`, `1.9`, `1`, `latest` など。ベースイメージとしての利用は 1.10 以降 ONBUILD で禁止。
- **minimal イメージ**: `1.9.1-minimal`, `1.9-minimal`, `1-minimal`, `minimal` など。マルチステージビルドでバイナリをコピーする用途で公式に案内されている。

*このメモは 2026 年 3 月時点のリポジトリ・ドキュメントに基づく。*
