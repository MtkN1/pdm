# ロックファイル

PDM は、既存のロックファイル `pdm.lock` からのみパッケージをインストールします。このファイルは、依存関係をインストールするための唯一の信頼できるソースとして機能します。ロックファイルには、次のような重要な情報が含まれています。

- すべてのパッケージとそのバージョン
- パッケージのファイル名とハッシュ
- オプションで、パッケージをダウンロードするための元の URL（参照：[静的 URL](#static-urls)）
- 各パッケージの依存関係とマーカー（参照：[親からメタデータを継承する](#inherit-the-metadata-from-parents)）

ロックファイルを作成または上書きするには、[`pdm lock`](../reference/cli.md#lock) を実行します。これは、[`pdm add`](../reference/cli.md#add) と同じ [更新戦略](./dependency.md#about-update-strategy) をサポートしています。さらに、[`pdm install`](../reference/cli.md#install) および [`pdm add`](../reference/cli.md#add) コマンドも自動的に `pdm.lock` ファイルを作成します。

??? NOTE "`pdm.lock` をバージョン管理に追加するべきですか？"

    それは状況によります。CI がローカル開発と同じ依存関係バージョンを使用し、予期しない失敗を回避したい場合は、`pdm.lock` ファイルをバージョン管理に追加するべきです。そうでない場合、プロジェクトがライブラリであり、CI がユーザーサイトでのインストールを模倣して、現在のバージョンが PyPI で何も壊さないことを確認したい場合は、`pdm.lock` ファイルを提出しないでください。

## ロックファイルに固定されたパッケージをインストールする

これを行うためのいくつかの類似したコマンドがありますが、わずかな違いがあります。

- [`pdm sync`](../reference/cli.md#sync) はロックファイルからパッケージをインストールします。
- [`pdm update`](../reference/cli.md#update) はロックファイルを更新し、その後 `pdm sync` を実行します。
- [`pdm install`](../reference/cli.md#install) はプロジェクトファイルの変更を確認し、必要に応じてロックファイルを更新し、その後 `pdm sync` を実行します。

`pdm sync` には、インストールされたパッケージを管理するためのいくつかのオプションもあります。

- `--clean`: ロックファイルに含まれていないパッケージを削除します。
- `--clean-unselected`（または `--only-keep`）: `--clean` のより徹底的なバージョンで、`-G`、`-d`、および `--prod` オプションで指定されたグループに含まれていないパッケージも削除します。
注意: デフォルトでは、`pdm sync` はロックファイルからすべてのグループを選択するため、`-G`、`-d`、および `--prod` が使用されない限り、`--clean-unselected` は `--clean` と同じです。

## ロックファイルのハッシュ

デフォルトでは、`pdm install` はロックファイルが `pyproject.toml` の内容と一致するかどうかを確認します。これは、ロックファイルに `pyproject.toml` のコンテンツハッシュを保存することで行われます。

ロックファイルのハッシュが最新かどうかを確認するには:

```bash
pdm lock --check
```

依存関係を変更せずにロックファイルを更新したい場合は、`--refresh` オプションを使用できます。

```bash
pdm lock --refresh
```

このコマンドは、ロックファイルに記録されたすべてのファイルハッシュも更新します。

## 別のロックファイルを指定して使用する

デフォルトでは、PDM は現在のディレクトリにある `pdm.lock` を使用します。`-L/--lockfile` オプションまたは `PDM_LOCKFILE` 環境変数を使用して別のロックファイルを指定できます。

```bash
pdm install --lockfile my-lockfile.lock
```

このコマンドは、`pdm.lock` の代わりに `my-lockfile.lock` からパッケージをインストールします。

代替ロックファイルは、異なる環境に対して競合する依存関係が存在する場合に役立ちます。この場合、それらを全体としてロックすると、PDM はエラーを発生させます。そのため、[依存関係グループのサブセットを選択](./dependency.md#select-a-subset-of-dependency-groups-to-install)して別々にロックする必要があります。

現実的な例として、プロジェクトが `werkzeug` のリリースバージョンに依存しており、開発中にローカルの開発中のコピーと一緒に作業したい場合があります。次のように `pyproject.toml` に追加できます。

```toml
[project]
requires-python = ">=3.7"
dependencies = ["werkzeug"]

[tool.pdm.dev-dependencies]
dev = ["werkzeug @ file:///${PROJECT_ROOT}/dev/werkzeug"]
```

次に、異なる目的のために異なるオプションで `pdm lock` を実行してロックファイルを生成します。

```bash
# デフォルト + 開発をロックし、pdm.lock に書き込みます。
# ローカルコピーの werkzeug を固定します。
pdm lock
# デフォルトをロックし、pdm.prod.lock に書き込みます。
# リリースバージョンの werkzeug を固定します。
pdm lock --prod -L pdm.prod.lock
```

ロックファイルの `metadata.groups` フィールドを確認して、どのグループが含まれているかを確認します。

## ロックファイルを書き込まないオプション

依存関係を追加または更新する際にロックファイルを更新したくない場合や、`pdm.lock` を生成したくない場合は、`--frozen-lockfile` オプションを使用できます。

```bash
pdm add --frozen-lockfile flask
```

この場合、ロックファイルが存在する場合は読み取り専用になり、書き込み操作は行われません。
ただし、必要に応じて依存関係の解決ステップは実行されます。

## ロック戦略

現在、ロックの動作を制御するための 3 つのフラグをサポートしています。`cross_platform`、`static_urls`、および `direct_minimal_versions` で、それぞれの意味は次のとおりです。
これらのフラグを `--strategy/-S` オプションで `pdm lock` に渡すことで、1 つ以上のフラグを指定できます。カンマ区切りのリストを指定するか、オプションを複数回渡すことで指定できます。
これらのコマンドは同じ方法で機能します。

```bash
pdm lock -S cross_platform,static_urls
pdm lock -S cross_platform -S static_urls
```

フラグはロックファイルにエンコードされ、次回 `pdm lock` を実行するときに読み取られます。ただし、フラグを無効にするには、フラグ名の前に `no_` を付けます。

```bash
pdm lock -S no_cross_platform
```

このコマンドは、ロックファイルをクロスプラットフォームにしません。

### クロスプラットフォーム

+++ 2.6.0

!!! warning "2.17.0 で非推奨"
    新しい動作については、[特定のプラットフォームまたは Python バージョン用にロックする](./lock-targets.md) を参照してください。

デフォルトでは、生成されたロックファイルは **クロスプラットフォーム** です。つまり、依存関係を解決する際に現在のプラットフォームは考慮されません。結果のロックファイルには、すべての可能なプラットフォームおよび Python バージョンのホイールと依存関係が含まれます。
ただし、リリースにすべてのホイールが含まれていない場合、これにより誤ったロックファイルが生成されることがあります。
これを回避するには、PDM に **このプラットフォーム** のみで機能するロックファイルを作成し、現在のプラットフォームに関連しないホイールをトリミングするように指示できます。
これは、`pdm lock` に `--strategy no_cross_platform` オプションを渡すことで行えます。

```bash
pdm lock --strategy no_cross_platform
```

### 静的 URL

+++ 2.8.0

デフォルトでは、PDM はロックファイルにパッケージのファイル名のみを保存します。これにより、異なるパッケージインデックス間での再利用が容易になります。
ただし、ロックファイルにパッケージの静的 URL を保存したい場合は、`pdm lock` に `--strategy static_urls` オプションを渡すことができます。

```bash
pdm lock --strategy static_urls
```

設定は保存され、同じロックファイルに対して記憶されます。`--strategy no_static_urls` を渡して無効にすることもできます。

### 直接最小バージョン

+++ 2.10.0

`--strategy direct_minimal_versions` を渡すことで有効にすると、`pyproject.toml` に指定された依存関係は、最新バージョンではなく、利用可能な最小バージョンに解決されます。これは、依存関係のバージョン範囲内でプロジェクトの互換性をテストしたい場合に役立ちます。

たとえば、`pyproject.toml` に `flask>=2.0` を指定した場合、他に互換性の問題がなければ、`flask` はバージョン `2.0.0` に解決されます。

!!! NOTE
    パッケージ依存関係のバージョン制約は将来にわたって互換性があるわけではありません。依存関係を最小バージョンに解決すると、後方互換性の問題が発生する可能性があります。
    たとえば、`flask==2.0.0` は `werkzeug>=2.0` を要求しますが、実際には `Werkzeug 3.0.0` とは互換性がありません。これは、リリースから 2 年後にリリースされました。

### 親からメタデータを継承する

+++ 2.11.0

以前は、`pdm lock` コマンドはパッケージメタデータをそのまま記録していました。インストール時には、PDM はトップレベルの要件から開始し、依存関係ツリーのリーフノードまで下に向かってトラバースします。その後、現在の環境に対してマーカーを評価します。マーカーが満たされない場合、パッケージは破棄されます。言い換えれば、インストール時に追加の「解決」ステップが必要です。

`inherit_metadata` 戦略が有効になっている場合、PDM はパッケージの祖先から環境マーカーを継承およびマージします。これらのマーカーはロック時にロックファイルにエンコードされ、インストールが高速化されます。これはバージョン `2.11.0` からデフォルトで有効になっており、設定でこの戦略を無効にするには、`pdm config strategy.inherit_metadata false` を使用します。

### 特定の日付以降のパッケージを除外する

+++ 2.13.0

`pdm lock` に `--exclude-newer` オプションを渡すことで、特定の日付以降のパッケージを除外できます。これは、ビルドの再現性を確保するために依存関係を特定の日付にロックしたい場合に役立ちます。

日付は RFC 3339 タイムスタンプ（例：`2006-12-02T02:07:43Z`）または同じ形式の UTC 日付（例：`2006-12-02`）として指定できます。

```bash
pdm lock --exclude-newer 2024-01-01
```

!!! note
    パッケージインデックスは、[PEP 700] で指定されているように `upload-time` フィールドをサポートしている必要があります。特定のディストリビューションにフィールドが存在しない場合、そのディストリビューションは利用できないものとして扱われます。

[PEP 700]: https://peps.python.org/pep-0700/

## ロックまたはインストールのための許容フォーマットを設定する

パッケージのフォーマット（バイナリ/ソースディストリビューション）を制御したい場合は、環境変数 `PDM_NO_BINARY`、`PDM_ONLY_BINARY`、および `PDM_PREFER_BINARY` を設定できます。

各環境変数はパッケージ名のカンマ区切りリストです。すべてのパッケージに適用するには `:all:` を設定できます。たとえば：

```toml
# werkzeug のバイナリはロックされず、インストールにも使用されません
PDM_NO_BINARY=werkzeug pdm add flask
# バイナリのみがロックファイルにロックされます
PDM_ONLY_BINARY=:all: pdm lock
# インストールにはバイナリは使用されません
PDM_NO_BINARY=:all: pdm install
# バイナリディストリビューションを優先し、より高いバージョンの sdist が利用可能な場合でも
PDM_PREFER_BINARY=flask pdm install
```

これらの値をプロジェクトの `pyproject.toml` に `tool.pdm.resolution` セクションの `no-binary`、`only-binary`、および `prefer-binary` キーで定義することもできます。
これらは環境変数と同じ形式を受け入れ、リストもサポートしています。

```toml
[tool.pdm.resolution]
# werkzeug と flask のバイナリはロックされず、インストールにも使用されません
no-binary = "werkzeug,flask"
# 同等
no-binary = ["werkzeug", "flask"]
# バイナリのみがロックファイルにロックされます
only-binary = ":all:"
# バイナリディストリビューションを優先し、より高いバージョンの sdist が利用可能な場合でも
prefer-binary = "flask"
```

!!! note
    各環境変数は `pyproject.toml` の代替よりも優先されます。

## プレリリースバージョンのインストールを許可する

`pyproject.toml` に次の設定を含めて有効にします。

```toml
[tool.pdm.resolution]
allow-prereleases = true
```

## ロックの失敗を解決する

PDM が要件を満たす解決策を見つけられない場合、エラーを発生させます。たとえば、

```bash
pdm django==3.1.4 "asgiref<3"
...
🔒 ロックに失敗しました
asgiref の解決策を見つけることができませんでした。次の競合が原因です。
    asgiref<3 (プロジェクトから)
    asgiref<4,>=3.2.10 (候補 django 3.1.4 から https://pypi.org/simple/django/)
これを修正するには、pyproject.toml の依存関係バージョン制約を緩和することができます。それが不可能な場合は、`[tool.pdm.resolution.overrides]` テーブルで解決されたバージョンを上書きすることもできます。
```

`django` のバージョンを下げるか、`asgiref` の上限を削除することができます。ただし、プロジェクトに適していない場合は、`pyproject.toml` で [解決されたパッケージバージョンを上書き](./config.md#override-the-resolved-package-versions) するか、特定のパッケージを [ロックしない](./config.md#exclude-specific-packages-and-their-dependencies-from-the-lock-file) ことも試すことができます。

## ロックされたパッケージを別の形式にエクスポートする

`pdm.lock` ファイルを他の形式にエクスポートできます。これにより、CI フローやイメージビルドプロセスが簡素化されます。現在、`requirements.txt` 形式のみがサポートされています。

```bash
pdm export -o requirements.txt
```

!!! TIP
    [`.pre-commit` フック](./advanced.md#hooks-for-pre-commit) で `pdm export` を実行することもできます。
