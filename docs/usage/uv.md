# uv を使用する (実験的)

+++ 2.19.0

PDM は、リゾルバおよびインストーラとして [uv](https://github.com/astral-sh/uv) を実験的にサポートしています。有効にするには:

```
pdm config use_uv true
```

PDM はシステム上の `uv` バイナリを自動的に検出します。最初に `uv` をインストールする必要があります。詳細については、[uv のインストールガイド](https://docs.astral.sh/uv/getting-started/installation/) を参照してください。

## uv の Python インストールを再利用する

uv は Python インタープリターのインストールもサポートしています。オーバーヘッドを避けるために、PDM を構成して uv の Python インストールを再利用することができます:

```
pdm config python.install_root $(uv python dir)
```

## 制限事項

uv による大幅なパフォーマンス向上にもかかわらず、次の制限事項に注意することが重要です:

- キャッシュファイルは uv 独自のキャッシュディレクトリに保存され、管理するには `uv` コマンドを使用する必要があります。
- PEP 582 ローカルパッケージレイアウトはサポートされていません。
- `inherit_metadata` ロック戦略は uv ではサポートされていません。ロックファイルに書き込む際に無視されます。
- `all` および `reuse` 以外の更新戦略はサポートされていません。
- 編集可能な要件はローカルパスでなければなりません。`-e git+<git_url>` のような要件はサポートされていません。
- `[tool.pdm.resolution]` の下の `overrides` および `excludes` 設定はサポートされていません。
