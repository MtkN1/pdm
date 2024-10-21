# 依存関係を管理する

PDM は、プロジェクトと依存関係を管理するための便利なコマンドを提供します。
以下の例は Ubuntu 18.04 で実行されており、Windows を使用している場合は、いくつかの変更が必要です。

## 依存関係を追加する

[`pdm add`](../reference/cli.md#add) は、1 つまたは複数の依存関係を指定することができ、依存関係の指定は [PEP 508](https://www.python.org/dev/peps/pep-0508/) に記載されています。

例:

```bash
pdm add requests   # requests を追加
pdm add requests==2.25.1   # バージョン制約付きで requests を追加
pdm add requests[socks]   # 追加の依存関係付きで requests を追加
pdm add "flask>=1.0" flask-sqlalchemy   # 異なる指定子を持つ複数の依存関係を追加
```

PDM は、`-G/--group <name>` オプションを提供して、追加の依存関係グループを許可し、それらの依存関係はそれぞれプロジェクトファイルの `[project.optional-dependencies.<name>]` テーブルに移動します。

他のオプショナルグループを `optional-dependencies` で参照することができます。パッケージがアップロードされる前でも可能です:

```toml
[project]
name = "foo"
version = "0.1.0"

[project.optional-dependencies]
socks = ["pysocks"]
jwt = ["pyjwt"]
all = ["foo[socks,jwt]"]
```

その後、依存関係とサブ依存関係が適切に解決され、インストールされます。解決されたすべての依存関係の結果を確認するには、`pdm.lock` を参照してください。

### ローカル依存関係

ローカルパッケージは、そのパスを指定して追加できます。パスはファイルまたはディレクトリである必要があります:

```bash
pdm add ./sub-package
pdm add ./first-1.0.0-py2.py3-none-any.whl
```

パスは `.` で始まる必要があります。そうでない場合は、通常の名前付き要件として認識されます。ローカル依存関係は、URL 形式で `pyproject.toml` ファイルに書き込まれます:

```toml
[project]
dependencies = [
    "sub-package @ file:///${PROJECT_ROOT}/sub-package",
    "first @ file:///${PROJECT_ROOT}/first-1.0.0-py2.py3-none-any.whl",
]
```

??? note "他のビルドバックエンドを使用する"
    `hatchling` を pdm バックエンドの代わりに使用している場合、URL は次のようになります:

    ```
    sub-package @ {root:uri}/sub-package
    first @ {root:uri}/first-1.0.0-py2.py3-none-any.whl
    ```
    他のバックエンドは、URL 内の相対パスのエンコードをサポートしておらず、代わりに絶対パスを書き込みます。

### URL 依存関係

PDM は、Web アドレスから直接パッケージをダウンロードしてインストールすることもサポートしています。

例:

```bash
# プレーン URL から gzipped パッケージをインストール
pdm add "https://github.com/numpy/numpy/releases/download/v1.20.0/numpy-1.20.0.tar.gz"
# プレーン URL からホイールをインストール
pdm add "https://github.com/explosion/spacy-models/releases/download/en_core_web_trf-3.5.0/en_core_web_trf-3.5.0-py3-none-any.whl"
```

### VCS 依存関係

Git リポジトリ URL や他のバージョン管理システムからもインストールできます。以下がサポートされています:

- Git: `git`
- Mercurial: `hg`
- Subversion: `svn`
- Bazaar: `bzr`

URL は次のようになります: `{vcs}+{url}@{rev}`

例:

```bash
# タグ `22.0` の pip リポジトリをインストール
pdm add "git+https://github.com/pypa/pip.git@22.0"
# URL に資格情報を提供
pdm add "git+https://username:password@github.com/username/private-repo.git@master"
# 依存関係に名前を付ける
pdm add "pip @ git+https://github.com/pypa/pip.git@22.0"
# または #egg フラグメントを使用
pdm add "git+https://github.com/pypa/pip.git@22.0#egg=pip"
# サブディレクトリからインストール
pdm add "git+https://github.com/owner/repo.git@master#egg=pkg&subdirectory=subpackage"
```

git に対して ssh スキームを使用するには、`https://` を `ssh://git@` に置き換えます。

例:

```bash
pdm add "wheel @ git+ssh://git@github.com/pypa/wheel.git@main"
```

### URL 内の資格情報を隠す

`${ENV_VAR}` 変数構文を使用して、URL 内の資格情報を隠すことができます:

```toml
[project]
dependencies = [
  "mypackage @ git+http://${VCS_USER}:${VCS_PASSWD}@test.git.com/test/mypackage.git@master"
]
```

これらの変数は、プロジェクトをインストールする際に環境変数から読み取られます。

### 開発専用の依存関係を追加する

+++ 1.5.0

PDM は、テスト用やリント用など、開発に役立つ依存関係のグループを定義することもサポートしています。これらの依存関係は、配布物のメタデータに表示されることはありませんので、`optional-dependencies` を使用するのはおそらく良いアイデアではありません。開発依存関係として定義できます:

```bash
pdm add -dG test pytest
```

これにより、次のような `pyproject.toml` が生成されます:

```toml
[tool.pdm.dev-dependencies]
test = ["pytest"]
```

開発専用の依存関係グループをいくつか持つことができます。`optional-dependencies` とは異なり、これらは `PKG-INFO` や `METADATA` などのパッケージ配布メタデータには表示されません。
パッケージインデックスはこれらの依存関係を認識しません。スキーマは `optional-dependencies` と似ていますが、`tool.pdm` テーブルにあります。

```toml
[tool.pdm.dev-dependencies]
lint = [
    "flake8",
    "black"
]
test = ["pytest", "pytest-cov"]
doc = ["mkdocs"]
```

後方互換性のために、`-d` または `--dev` のみが指定された場合、依存関係はデフォルトで `[tool.pdm.dev-dependencies]` の `dev` グループに移動します。

!!! NOTE
    同じグループ名は、`[tool.pdm.dev-dependencies]` と `[project.optional-dependencies]` の両方に表示されてはなりません。

### 編集可能な依存関係

**ローカルディレクトリ** および **VCS 依存関係** は、[編集可能モード](https://pip.pypa.io/en/stable/cli/pip_install/#editable-installs) でインストールできます。`pip install -e <package>` のように動作します。**編集可能なパッケージは開発依存関係でのみ許可されます**:

!!! NOTE
    編集可能なインストールは、`dev` 依存関係グループでのみ許可されます。他のグループ（デフォルトを含む）は `[PdmUsageError]` で失敗します。

```bash
# ディレクトリへの相対パス
pdm add -e ./sub-package --dev
# ローカルディレクトリへのファイル URL
pdm add -e file:///path/to/sub-package --dev
# VCS URL
pdm add -e git+https://github.com/pallets/click.git@main#egg=click --dev
```

### バージョン指定子を保存する

パッケージがバージョン指定子なしで指定された場合（例: `pdm add requests`）。
PDM は、依存関係に対して保存されるバージョン指定子の異なる動作を 3 つ提供します。
`--save-<strategy>` オプションで指定します（依存関係の最新バージョンが `2.21.0` であると仮定します）:

- `minimum`: 最小バージョン指定子を保存します: `>=2.21.0`（デフォルト）。
- `compatible`: 互換性のあるバージョン指定子を保存します: `>=2.21.0,<3.0.0`。
- `exact`: 正確なバージョン指定子を保存します: `==2.21.0`。
- `wildcard`: バージョンを制約せず、指定子をワイルドカードにします: `*`。

### プレリリースを追加する

[`pdm add`](../reference/cli.md#add) に `--pre/--prerelease` オプションを指定すると、指定されたパッケージに対してプレリリースが許可されます。

## 既存の依存関係を更新する

ロックファイル内のすべての依存関係を更新するには:

```bash
pdm update
```

指定されたパッケージを更新するには:

```bash
pdm update requests
```

複数の依存関係グループを更新するには:

```bash
pdm update -G security -G http
```

または、カンマ区切りのリストを使用します:

```bash
pdm update -G "security,http"
```

指定されたグループ内の特定のパッケージを更新するには:

```bash
pdm update -G security cryptography
```

グループが指定されていない場合、PDM はデフォルトの依存関係セットで要件を検索し、見つからない場合はエラーを発生させます。

開発依存関係のパッケージを更新するには:

```bash
# すべてのデフォルト + 開発依存関係を更新
pdm update -d
# 指定されたグループの開発依存関係のパッケージを更新
pdm update -dG test pytest
```

### 更新戦略について

同様に、PDM は依存関係とサブ依存関係の更新に対して 3 つの異なる動作を提供します。
`--update-<strategy>` オプションで指定します:

- `reuse`: コマンドラインで指定されたものを除いて、すべてのロックされた依存関係を保持します（デフォルト）。
- `reuse-installed`: 作業セットにインストールされているバージョンを再利用しようとします。**これにより、コマンドラインで要求されたパッケージにも影響します**。
- `eager`: コマンドラインで指定されたパッケージとその再帰的なサブ依存関係の新しいバージョンをロックしようとし、他の依存関係はそのままにします。
- `all`: すべての依存関係とサブ依存関係を更新します。

### バージョン指定子を破るバージョンにパッケージを更新する

`-u/--unconstrained` を指定すると、`pyproject.toml` のバージョン指定子を無視するように PDM に指示できます。
これは `yarn upgrade -L/--latest` コマンドと同様に動作します。さらに、
[`pdm update`](../reference/cli.md#update_2) は `--pre/--prerelease` オプションもサポートしています。

## 既存の依存関係を削除する

プロジェクトファイルとライブラリディレクトリから既存の依存関係を削除するには:

```bash
# デフォルトの依存関係から requests を削除
pdm remove requests
# オプショナル依存関係の 'web' グループから h11 を削除
pdm remove -G web h11
# `test` グループの開発依存関係から pytest-cov を削除
pdm remove -dG test pytest-cov
```

## 古いパッケージと最新バージョンを一覧表示する

+++ 2.13.0

古いパッケージと最新バージョンを一覧表示するには:

```bash
pdm outdated
```

表示するパッケージをフィルタリングするためにグロブパターンを渡すことができます:

```bash
pdm outdated requests* flask*
```

## インストールする依存関係グループのサブセットを選択する

次の依存関係を持つプロジェクトがあるとします:

```toml
[project]  # これはプロダクション依存関係です
dependencies = ["requests"]

[project.optional-dependencies]  # これはオプショナル依存関係です
extra1 = ["flask"]
extra2 = ["django"]

[tool.pdm.dev-dependencies]  # これは開発依存関係です
dev1 = ["pytest"]
dev2 = ["mkdocs"]
```

| コマンド                         | その動作                                                         | コメント                  |
| ------------------------------- | -------------------------------------------------------------------- | ------------------------- |
| `pdm install`                   | ロックファイルにロックされたすべてのグループをインストール                            |                           |
| `pdm install -G extra1`         | プロダクション依存関係、開発依存関係、および "extra1" オプショナルグループをインストール             |                           |
| `pdm install -G dev1`           | プロダクション依存関係と "dev1" 開発グループのみをインストール                          |                           |
| `pdm install -G:all`            | プロダクション依存関係、開発依存関係、および "extra1"、"extra2" オプショナルグループをインストール   |                           |
| `pdm install -G extra1 -G dev1` | プロダクション依存関係、"extra1" オプショナルグループ、および "dev1" 開発グループのみをインストール |                           |
| `pdm install --prod`            | プロダクション依存関係のみをインストール                                                    |                           |
| `pdm install --prod -G extra1`  | プロダクション依存関係と "extra1" オプショナルグループをインストール                              |                           |
| `pdm install --prod -G dev1`    | 失敗、`--prod` は開発依存関係と一緒に指定できません                  | `--prod` オプションを省略します |

**すべての** 開発依存関係は、`--prod` が指定されていない限り、`-G` が開発グループを指定しない限り含まれます。

さらに、ルートプロジェクトをインストールしたくない場合は、`--no-self` オプションを追加し、すべてのパッケージを非編集可能バージョンでインストールしたい場合は `--no-editable` を使用できます。

これらのオプションを使用して、pdm lock コマンドを使用して、指定されたグループのみをロックすることもできます。これらはロックファイルの `[metadata]` テーブルに記録されます。`--group/--prod/--dev/--no-default` オプションが指定されていない場合、`pdm sync` および `pdm update` はロックファイル内のグループを使用して操作を行います。ただし、ロックファイルに含まれていないグループがコマンドの引数として指定された場合、PDM はエラーを発生させます。

## 依存関係のオーバーライド

特定のパッケージのバージョンがすべての制約を満たさない場合、解決は失敗します。この場合、依存関係のオーバーライドを使用して、特定のバージョンのパッケージを使用するようにリゾルバに指示できます。

オーバーライドは、ユーザーが依存関係がパッケージが宣言するよりも新しいバージョンと互換性があることを知っているが、パッケージがその互換性を宣言するためにまだ更新されていない場合に役立つ最後の手段です。

たとえば、トランジティブ依存関係が `pydantic>=1.0,<2.0` を宣言しているが、ユーザーがパッケージが `pydantic>=2.0` と互換性があることを知っている場合、ユーザーは `pydantic>=2.0,<3` で宣言された依存関係をオーバーライドして、リゾルバが続行できるようにします。

PDM では、オーバーライドを指定する方法が 2 つあります:

### プロジェクトファイル内

+++ 1.12.0

`pyproject.toml` ファイルの `[tool.pdm.resolution.overrides]` テーブルの下にオーバーライドを指定できます:

```toml
[tool.pdm.resolution.overrides]
asgiref = "3.2.10"  # 正確なバージョン
urllib3 = ">=1.26.2"  # バージョン範囲
pytz = "https://mypypi.org/packages/pytz-2020.9-py3-none-any.whl"  # 絶対 URL
```

テーブル内の各エントリは、パッケージ名とバージョン指定子です。バージョン指定子は、バージョン範囲、正確なバージョン、または絶対 URL である場合があります。

### CLI オプションを介して

+++ 2.17.0

PDM は、依存関係のオーバーライドを要件ファイルから読み取ることもサポートしています。このファイルは、pip の制約ファイル（`--constraint constraints.txt`）と同様に機能し、構文は要件ファイルと同じです:

```
requests==2.20.0
django==1.11.8
certifi==2018.11.17
chardet==3.0.4
idna==2.7
pytz==2019.3
urllib3==1.23
```

オーバーライドファイルは、組織内の複数のプロジェクトで共有できる集中管理された場所に依存関係を保存するための便利な方法です。

依存関係の解決を行うさまざまな PDM コマンドに制約ファイルを渡すことができます。たとえば、[`pdm install`](../reference/cli.md#install)、[`pdm lock`](../reference/cli.md#lock)、[`pdm add`](../reference/cli.md#add) などです。

```bash
pdm lock --override constraints.txt
```

このオプションは複数回指定できます。

オーバーライドファイルは、URL を介しても提供できます。たとえば、`--override http://example.com/constraints.txt` のように、組織がリモートサーバーでそれらを保存および提供できるようにします。

## インストールされているパッケージを表示する

`pip list` と同様に、パッケージディレクトリにインストールされているすべてのパッケージを一覧表示できます:

```bash
pdm list
```

### グループを含めたり除外したりする

デフォルトでは、作業セットにインストールされているすべてのパッケージが一覧表示されます。表示するグループを `--include/--exclude` オプションで指定できます。
`include` は `exclude` よりも優先されます。

```bash
pdm list --include dev
pdm list --exclude test
```

特別なグループ `:sub` があり、含まれている場合、すべてのトランジティブ依存関係も表示されます。デフォルトで含まれています。

`pdm list` に `--resolve` を渡すと、作業セットにインストールされているパッケージではなく、`pdm.lock` に解決されたパッケージが表示されます。

### 出力フィールドと形式を変更する

デフォルトでは、名前、バージョン、および場所がリスト出力に表示されます。`--fields` オプションを使用して、表示するフィールドを指定したり、フィールドの順序を指定したりできます:

```bash
pdm list --fields name,licenses,version
```

サポートされているすべてのフィールドについては、[CLI リファレンス](../reference/cli.md#list_1) を参照してください。

また、デフォルトのテーブル出力以外の出力形式を指定することもできます。サポートされている形式とオプションは `--csv`、`--json`、`--markdown`、および `--freeze` です。

### 依存関係ツリーを表示する

または、依存関係ツリーを表示するには:

```bash
$ pdm list --tree
tempenv 0.0.0
└── click 7.0 [ required: <7.0.0,>=6.7 ]
black 19.10b0
├── appdirs 1.4.3 [ required: Any ]
├── attrs 19.3.0 [ required: >=18.1.0 ]
├── click 7.0 [ required: >=6.5 ]
├── pathspec 0.7.0 [ required: <1,>=0.6 ]
├── regex 2020.2.20 [ required: Any ]
├── toml 0.10.0 [ required: >=0.9.4 ]
└── typed-ast 1.4.1 [ required: >=1.4.0 ]
bump2version 1.0.0
```

`--tree` モードでは、一致するパッケージのサブツリーのみが表示されます。これは、特定のパッケージが必要な理由を示すために `pnpm why` と同じ目的を達成するために使用できます。

### パターンでパッケージをフィルタリングする

また、`pdm list` にパターンを渡すことで、表示するパッケージを制限することもできます:

```bash
pdm list flask-* requests-*
```

??? warning "シェル展開に注意してください"
    ほとんどのシェルでは、現在のディレクトリに一致するファイルがある場合、ワイルドカード `*` が展開されます。
    予期しない結果を避けるために、パターンをシングルクォートで囲むことができます: `pdm list 'flask-*' 'requests-*'`。

`--tree` モードでは、一致するパッケージのサブツリーのみが表示されます。これは、特定のパッケージが必要な理由を示すために `pnpm why` と同じ目的を達成するために使用できます。

```bash
$ pdm list --tree --reverse certifi
certifi 2023.7.22
└── requests 2.31.0 [ requires: >=2017.4.17 ]
    └── cachecontrol[filecache] 0.13.1 [ requires: >=2.16.0 ]
```

## グローバルプロジェクトを管理する

ユーザーがグローバル Python インタープリターの依存関係を追跡したい場合があります。
PDM を使用すると、`-g/--global` オプションを使用して簡単に行うことができます。このオプションはほとんどのサブコマンドでサポートされています。

このオプションが指定されると、`<CONFIG_ROOT>/global-project` がプロジェクトディレクトリとして使用されます。
これは、プロジェクトファイルが自動的に作成されることを除いて、通常のプロジェクトとほぼ同じです。
ビルド機能はサポートされていません。このアイデアは Haskell の [stack](https://docs.haskellstack.org) から取られています。

ただし、`stack` とは異なり、デフォルトでは、ローカルプロジェクトが見つからない場合に PDM がグローバルプロジェクトを自動的に使用することはありません。
ユーザーは `-g/--global` を明示的に指定してアクティブにする必要があります。パッケージが間違った場所に移動するのはあまり嬉しくないためです。
ただし、PDM はこの決定をユーザーに委ねています。`global_project.fallback` を `true` に設定するだけです。

デフォルトでは、PDM が暗黙的にグローバルプロジェクトを使用する場合、次のメッセージが表示されます: `Project is not found, fallback to the global project`。
このメッセージを無効にするには、`global_project.fallback_verbose` を `false` に設定します。

グローバルプロジェクトが `<CONFIG_ROOT>/global-project` 以外のプロジェクトファイルを追跡するようにしたい場合は、
`-p/--project <path>` オプションを使用してプロジェクトパスを指定できます。
特に、`--global --project .` を指定すると、
PDM は現在のプロジェクトの依存関係をグローバル Python にインストールします。

!!! warning
    グローバルプロジェクトが使用されている場合、`remove` および `sync --clean/--pure` コマンドに注意してください。これにより、システム Python にインストールされたパッケージが削除される可能性があります。
