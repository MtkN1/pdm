# プロジェクトを構成する

PDM の `config` コマンドは `git config` と同様に機能しますが、構成を表示するために `--list` は必要ありません。

現在の構成を表示します:

```bash
pdm config
```

単一の構成を取得します:

```bash
pdm config pypi.url
```

構成値を変更し、ホーム構成に保存します:

```bash
pdm config pypi.url "https://test.pypi.org/simple"
```

デフォルトでは、構成はグローバルに変更されますが、このプロジェクトでのみ構成を表示する場合は、`--local` フラグを追加します:

```bash
pdm config --local pypi.url "https://test.pypi.org/simple"
```

ローカル構成はプロジェクトルートディレクトリの `pdm.toml` に保存されます。

## 構成ファイル

構成ファイルは次の順序で検索されます:

1. `<PROJECT_ROOT>/pdm.toml` - プロジェクト構成
2. `<CONFIG_ROOT>/config.toml` - ホーム構成
3. `<SITE_CONFIG_ROOT>/config.toml` - サイト構成

`<CONFIG_ROOT>` は次の場所です:

- Linux では [XDG Base Directory Specification](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html) で定義されているように `$XDG_CONFIG_HOME/pdm` (`ほとんどの場合 ~/.config/pdm`)
- macOS では [Apple File System Basics](https://developer.apple.com/library/archive/documentation/FileManagement/Conceptual/FileSystemProgrammingGuide/FileSystemOverview/FileSystemOverview.html) で定義されているように `~/Library/Application Support/pdm`
- Windows では [Known folders](https://docs.microsoft.com/en-us/windows/win32/shell/known-folders) で定義されているように `%USERPROFILE%\AppData\Local\pdm`

`<SITE_CONFIG_ROOT>` は次の場所です:

- Linux では [XDG Base Directory Specification](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html) で定義されているように `$XDG_CONFIG_DIRS/pdm` (`ほとんどの場合 /etc/xdg/pdm`)
- macOS では [Apple File System Basics](https://developer.apple.com/library/archive/documentation/FileManagement/Conceptual/FileSystemProgrammingGuide/FileSystemOverview/FileSystemOverview.html) で定義されているように `/Library/Application Support/pdm`
- Windows では [Known folders](https://docs.microsoft.com/en-us/windows/win32/shell/known-folders) で定義されているように `C:\ProgramData\pdm\pdm`

`-g/--global` オプションが使用される場合、最初の項目は `<CONFIG_ROOT>/global-project/pdm.toml` に置き換えられます。

利用可能なすべての構成項目は [構成ページ](../reference/configuration.md) にあります。

## Python ファインダーを構成する

デフォルトでは、PDM は次のソースで Python インタープリターを検索します:

- `venv`: PDM 仮想環境の場所
- `path`: `PATH` 環境変数
- `pyenv`: [pyenv](https://github.com/pyenv/pyenv) インストールルート
- `rye`: [rye](https://rye-up.com/) ツールチェーンインストールルート
- `asdf`: [asdf](https://asdf-vm.com/) python インストールルート
- `winreg`: Windows レジストリ

いくつかのソースを選択解除したり、順序を変更したりするには、`python.providers` 構成キーを設定します:

```bash
pdm config python.providers rye   # Rye ソースのみ
pdm config python.providers pyenv,asdf  # pyenv と asdf
```

## 解決結果にプレリリースを許可する

デフォルトでは、`pdm` の依存関係解決ツールは、依存関係の指定されたバージョン範囲に安定版がない場合を除き、プレリリースを無視します。この動作は `[tool.pdm.resolution]` テーブルで `allow-prereleases` を `true` に設定することで変更できます:

```toml
[tool.pdm.resolution]
allow-prereleases = true
```

## パッケージインデックスを構成する

PDM にパッケージを見つける場所を指定するには、`pyproject.toml` にソースを追加するか、`pypi.*` 構成を介して行います。

`pyproject.toml` にソースを追加します:

```toml
[[tool.pdm.source]]
name = "private"
url = "https://private.pypi.org/simple"
verify_ssl = true
```

`pdm config` を介してデフォルトのインデックスを変更します:

```bash
pdm config pypi.url "https://test.pypi.org/simple"
```

`pdm config` を介して追加のインデックスを追加します:

```bash
pdm config pypi.extra.url "https://extra.pypi.org/simple"
```

利用可能な構成オプションは次のとおりです:

- `url`: インデックスの URL
- `verify_ssl`: (オプション) SSL 証明書を検証するかどうか、デフォルトは true
- `username`: (オプション) インデックスのユーザー名
- `password`: (オプション) インデックスのパスワード
- `type`: (オプション) index または find_links、デフォルトは index

??? note "ソースタイプについて"
    デフォルトでは、すべてのソースは pip の `--index-url` および `--extra-index-url` のような [PEP 503](https://www.python.org/dev/peps/pep-0503/) スタイルの "インデックス" と見なされますが、タイプを `find_links` に設定することで、ファイルやリンクを直接検索することができます。2 つのタイプの違いについては [この回答](https://stackoverflow.com/a/46651848) を参照してください。

    たとえば、ローカルディレクトリをソースとして使用するには:

    ```toml
    [[tool.pdm.source]]
    name = "local"
    url = "file:///${PROJECT_ROOT}/packages"
    type = "find_links"
    ```

これらの構成は次の順序で読み取られ、最終的なソースリストが構築されます:

- `pypi.url`、`pyproject.toml` の `name` フィールドに `pypi` が表示されない場合
- `pyproject.toml` のソース
- PDM 構成の `pypi.<name>.url`

`pypi.ignore_stored_index` を `true` に設定して、PDM 構成からのすべての追加インデックスを無効にし、`pyproject.toml` に指定されたもののみを使用することができます。

!!! TIP "デフォルトの PyPI インデックスを無効にする"
    デフォルトの PyPI インデックスを省略したい場合は、ソース名を `pypi` に設定するだけで、そのソースが **置き換え** られます。

    ```toml
    [[tool.pdm.source]]
    url = "https://private.pypi.org/simple"
    verify_ssl = true
    name = "pypi"
    ```

??? note "pyproject.toml または構成のインデックス"
    プロジェクトを使用する他の人とインデックスを共有したい場合は、`pyproject.toml` に追加する必要があります。
    たとえば、いくつかのパッケージはプライベートインデックスにのみ存在し、インデックスを構成しないとインストールできません。
    それ以外の場合は、他の人には表示されないローカル構成に保存します。

### ソースの順序を尊重する

デフォルトでは、すべてのソースは同等と見なされ、パッケージはバージョンとホイールタグでソートされ、最も一致するものが選択されます。

場合によっては、優先ソースからパッケージを返し、他のソースから見つからない場合に検索することを望むかもしれません。PDM は `respect-source-order` 構成を読み取ることでこれをサポートします。たとえば:

```toml
[tool.pdm.resolution]
respect-source-order = true

[[tool.pdm.source]]
name = "private"
url = "https://private.pypi.org/simple"

[[tool.pdm.source]]
name = "pypi"
url = "https://pypi.org/simple"
```

パッケージは最初に `private` インデックスから検索され、一致するバージョンが見つからない場合にのみ、`pypi` インデックスから検索されます。

### 個々のパッケージのインデックスを指定する

`include_packages` および `exclude_packages` 構成を使用して、パッケージを特定のソースにバインドできます。

```toml
[[tool.pdm.source]]
name = "private"
url = "https://private.pypi.org/simple"
include_packages = ["foo", "foo-*"]
exclude_packages = ["bar-*"]
```

上記の構成では、`foo` または `foo-*` に一致するパッケージは `private` インデックスからのみ検索され、`bar-*` に一致するパッケージは `private` を除くすべてのインデックスから検索されます。

`include_packages` および `exclude_packages` はどちらもオプションであり、グロブパターンのリストを受け入れ、パターンが一致する場合に `include_packages` が排他的に適用されます。

### インデックスとともに資格情報を保存する

`${ENV_VAR}` 変数展開を使用して URL に資格情報を指定でき、これらの変数は環境変数から読み取られます:

```toml
[[tool.pdm.source]]
name = "private"
url = "https://${PRIVATE_PYPI_USERNAME}:${PRIVATE_PYPI_PASSWORD}@private.pypi.org/simple"
```

### HTTPS 証明書を構成する

HTTPS リクエスト用にカスタム CA バンドルまたはクライアント証明書を使用できます。これは、インデックス（パッケージのダウンロード用）とリポジトリ（アップロード用）の両方に構成できます:

```bash
pdm config pypi.ca_certs /path/to/ca_bundle.pem
pdm config repository.pypi.ca_certs /path/to/ca_bundle.pem
```

さらに、標準の certifi 証明書の代わりにシステムの信頼ストアを使用して HTTPS 証明書を検証することも可能です。このアプローチは、追加の構成なしで企業プロキシ証明書をサポートすることが一般的です。

`truststore` を使用するには、Python 3.10 以降が必要であり、`truststore` を PDM と同じ環境にインストールする必要があります:

```bash
pdm self add truststore
```

さらに、`REQUESTS_CA_BUNDLE` および `CURL_CA_BUNDLE` 環境変数で指定された CA 証明書も設定されている場合に使用されます。

### インデックス構成のマージ

インデックス構成は `[[tool.pdm.source]]` テーブルの `name` フィールドまたは構成ファイルの `pypi.<name>` キーでマージされます。
これにより、URL と資格情報を別々に保存して、ソース管理で秘密が公開されないようにすることができます。
たとえば、次の構成がある場合:

```toml
[[tool.pdm.source]]
name = "private"
url = "https://private.pypi.org/simple"
```

構成ファイルに資格情報を保存できます:

```bash
pdm config pypi.private.username "foo"
pdm config pypi.private.password "bar"
```

PDM は `private` インデックスの構成を両方の場所から取得できます。

インデックスにユーザー名とパスワードが必要ですが、環境変数や構成ファイルから見つからない場合、PDM は入力を求めます。また、`keyring` がインストールされている場合は、資格情報ストアとして使用されます。PDM はインストールされたパッケージまたは CLI からの `keyring` を使用できます。

## 中央インストールキャッシュ

システム上の多くのプロジェクトでパッケージが必要な場合、各プロジェクトは独自のコピーを保持する必要があります。これは、特にデータサイエンスや機械学習プロジェクトでは、ディスクスペースの無駄になる可能性があります。

PDM は、中央パッケージリポジトリにインストールして、異なるプロジェクトでそのインストールにリンクすることにより、同じホイールのインストールをキャッシュすることをサポートします。有効にするには、次のコマンドを実行します:

```bash
pdm config install.cache on
```

コマンドに `--local` オプションを追加することで、プロジェクトごとに有効にすることができます。

キャッシュは `$(pdm config cache_dir)/packages` にあります。`pdm cache info` を使用してキャッシュの使用状況を表示できます。キャッシュされたインストールは自動的に管理されるため、プロジェクトにリンクされていない場合は削除されることに注意してください。ディスクからキャッシュを手動で削除すると、システム上の一部のプロジェクトが壊れる可能性があります。

さらに、いくつかの異なるリンク方法がサポートされています:

- `symlink`（デフォルト）、パッケージファイルへのシンボリックリンクを作成します。
- `hardlink`、キャッシュエントリのパッケージファイルへのハードリンクを作成します。

次のコマンドを実行して、リンク方法を切り替えることができます: `pdm config [--local] install.cache_method <method>`。

!!! note
    パッケージソースのいずれかからインストールされたパッケージのみがキャッシュされます。

## アップロード用のリポジトリを構成する

[`pdm publish`](../reference/cli.md#publish) コマンドを使用する場合、リポジトリの秘密は **グローバル** 構成ファイル (`<CONFIG_ROOT>/config.toml`) から読み取られます。構成ファイルの内容は次のとおりです:

```toml
[repository.pypi]
username = "frostming"
password = "<secret>"

[repository.company]
url = "https://pypi.company.org/legacy/"
username = "frostming"
password = "<secret>"
ca_certs = "/path/to/custom-cacerts.pem"
```

または、これらの資格情報を環境変数で提供できます:

```bash
export PDM_PUBLISH_REPO=...
export PDM_PUBLISH_USERNAME=...
export PDM_PUBLISH_PASSWORD=...
export PDM_PUBLISH_CA_CERTS=...
```

PEM エンコードされた証明書認証局バンドル (`ca_certs`) は、サーバー証明書が標準の [certifi](https://github.com/certifi/python-certifi/blob/master/certifi/cacert.pem) CA バンドルによって署名されていないローカル/カスタム PyPI リポジトリに使用できます。

!!! NOTE
    リポジトリは前のセクションのインデックスとは異なります。リポジトリは公開用であり、インデックスはロックおよび解決用です。これらは構成を共有しません。

!!! TIP
    `pypi` および `testpypi` リポジトリの `url` を構成する必要はありません。デフォルト値で埋められます。
    ユーザー名、パスワード、および証明書認証局バンドルは、それぞれ `--username`、`--password`、および `--ca-certs` を介して `pdm publish` のコマンドラインから渡すことができます。

コマンドラインからリポジトリ構成を変更するには、[`pdm config`](../reference/cli.md#config) コマンドを使用します:

```bash
pdm config repository.pypi.username "__token__"
pdm config repository.pypi.password "my-pypi-token"

pdm config repository.company.url "https://pypi.company.org/legacy/"
pdm config repository.company.ca_certs "/path/to/custom-cacerts.pem"
```

## keyring を使用したパスワード管理

keyring が利用可能でサポートされている場合、パスワードは構成ファイルに書き込む代わりに keyring に保存および取得されます。これは、インデックスとアップロードリポジトリの両方をサポートします。サービス名は、インデックスの場合は `pdm-pypi-<name>`、リポジトリの場合は `pdm-repository-<name>` になります。

keyring を有効にするには、PDM と同じ環境に `keyring` をインストールするか、グローバルにインストールします。PDM 環境に keyring を追加するには:

```bash
pdm self add keyring
```

または、keyring をグローバルにインストールしている場合は、CLI が `PATH` 環境変数に公開されていることを確認して、PDM によって検出可能にします:

```bash
export PATH=$PATH:path/to/keyring/bin
```

### Azure Artifacts 用の keyring を使用したパスワード管理

Azure Artifacts に対して認証を試みる場合、AD グループを使用して認証することができます: `pdm self add keyring artifacts-keyring` を実行して、認証に artifacts-keyring が使用されることを確認します。

次に、アーティファクトの URL を `pyproject.toml` に追加します

```toml
[[tool.pdm.source]]
name = "NameOfFeed"
url = "https://pkgs.dev.azure.com/[org name]/_packaging/[feed name]/pypi/simple/"
```

## ロックファイルから特定のパッケージとその依存関係を除外する

+++ 2.12.0

コードが使用しないことが確実な特定のパッケージをロックファイルに含めたくない場合があります。この場合、依存関係の解決中にそれらを完全にスキップできます:

```toml
[tool.pdm.resolution]
excludes = ["requests"]
```

この構成を使用すると、`requests` はロックファイルにロックされず、他のパッケージによって依存されていない限り、その依存関係（`urllib3` や `idna` など）も解決結果に表示されません。インストーラーはそれらをピックアップできません。

## すべての pdm 呼び出しに定数引数を渡す

+++ 2.7.0

個々の pdm コマンドに渡される追加オプションを `tool.pdm.options` 構成で追加できます:

```toml
[tool.pdm.options]
add = ["--no-isolation", "--no-self"]
install = ["--no-self"]
lock = ["--no-cross-platform"]
```

これらのオプションはコマンド名の直後に追加されます。たとえば、上記の構成に基づいて、
`pdm add requests` は `pdm add --no-isolation --no-self requests` と同等です。

## パッケージ警告を無視する

+++ 2.10.0

依存関係を解決するときに次のような警告が表示されることがあります:

```bash
PackageWarning: Skipping scipy@1.10.0 because it requires Python
<3.12,>=3.8 but the project claims to work with Python>=3.9.
Narrow down the `requires-python` range to include this version. For example, ">=3.9,<3.12" should work.
  warnings.warn(record.message, PackageWarning, stacklevel=1)
Use `-q/--quiet` to suppress these warnings, or ignore them per-package with `ignore_package_warnings` config in [tool.pdm] table.
```

これは、パッケージのサポートされている Python バージョンの範囲が `pyproject.toml` に指定された `requires-python` 値をカバーしていないためです。
これらの警告をパッケージごとに無視するには、次の構成を追加します:

```toml
[tool.pdm]
ignore_package_warnings = ["scipy", "tensorflow-*"]
```

各項目はパッケージ名に一致する大文字小文字を区別しないグロブパターンです。
