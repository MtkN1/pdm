# PDM スクリプト

PDM を使用すると、`npm run` のように、ローカルパッケージが読み込まれた状態で任意のスクリプトやコマンドを実行できます。

## 任意のスクリプト

```bash
pdm run flask run -p 54321
```

これにより、プロジェクト環境内のパッケージを認識した環境で `flask run -p 54321` が実行されます。

## 単一ファイルスクリプト

+++ 2.16.0

PDM は、[インラインスクリプトメタデータ](https://peps.python.org/pep-0723/) を使用して単一ファイルスクリプトを実行できます。

以下は、埋め込みメタデータを含むスクリプトの例です:

```python
# test_script.py
# /// script
# requires-python = ">=3.11"
# dependencies = [
#   "requests<3",
#   "rich",
# ]
# ///

import requests
from rich.pretty import pprint

resp = requests.get("https://peps.python.org/api/peps.json")
data = resp.json()
pprint([(k, v["title"]) for k, v in data.items()][:10])
```

これを `pdm run test_script.py` で実行すると、指定された依存関係がインストールされた一時環境が作成され、スクリプトが実行されます:

```python
[
│   ('1', 'PEP Purpose and Guidelines'),
│   ('2', 'Procedure for Adding New Modules'),
│   ('3', 'Guidelines for Handling Bug Reports'),
│   ('4', 'Deprecation of Standard Modules'),
│   ('5', 'Guidelines for Language Evolution'),
│   ('6', 'Bug Fix Releases'),
│   ('7', 'Style Guide for C Code'),
│   ('8', 'Style Guide for Python Code'),
│   ('9', 'Sample Plaintext PEP Template'),
│   ('10', 'Voting Guidelines')
]
```
前回作成された環境を再利用したい場合は、`--reuse-env` オプションを追加します。
また、スクリプトメタデータに `[tool.pdm]` セクションを追加して PDM を構成することもできます。例えば:

```python
# test_script.py
# /// script
# requires-python = ">=3.11"
# dependencies = [
#   "requests<3",
#   "rich",
# ]
#
# [[tool.pdm.source]]  # カスタムインデックスを使用する
# url = "https://mypypi.org/simple"
# name = "pypi"
# ///
```

詳細については、[仕様](https://packaging.python.org/en/latest/specifications/inline-script-metadata/#inline-script-metadata) を参照してください。

## ユーザースクリプト

PDM は、`pyproject.toml` のオプションの `[tool.pdm.scripts]` セクションでカスタムスクリプトショートカットもサポートしています。

!!! NOTE "`[project.scripts]` と混同しないでください"
    `pyproject.toml` には別のフィールド `[project.scripts]` があり、スクリプトは `pdm run` で実行できます。これは、パッケージと一緒にインストールされるコンソールスクリプトエントリポイントを定義するために使用されます。したがって、プロジェクト自体が環境にインストールされた後にのみ実行可能です。つまり、`distribution = true` である必要があります。

    対照的に、`[tool.pdm.scripts]` はプロジェクトで実行するタスクを定義します。これは、`distribution` が `true` か `false` かに関係なくプロジェクトに対して機能します。タスクは主に開発およびテスト目的のためのものであり、後で示すように、より多くのタイプと設定をサポートします。`Makefile` の代替として考えることができます。プロジェクトをインストールする必要はありませんが、`pyproject.toml` ファイルが存在する必要があります。

    `[project.scripts]` についての詳細な説明は[こちら](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/#creating-executable-scripts) を参照してください。

次に、`pdm run <script_name>` を実行して、PDM プロジェクトのコンテキストでスクリプトを呼び出すことができます。例えば:

```toml
[tool.pdm.scripts]
start = "flask run -p 54321"
```

そして、ターミナルで:

```bash
$ pdm run start
Flask サーバーが http://127.0.0.1:54321 で開始されました
```

後続の引数はコマンドに追加されます:

```bash
$ pdm run start -h 0.0.0.0
Flask サーバーが http://0.0.0.0:54321 で開始されました
```

!!! note "Yarn のようなスクリプトショートカット"
    スクリプトが組み込みまたはプラグイン提供のコマンドと競合しない限り、すべてのスクリプトをルートコマンドとして利用できる組み込みショートカットがあります。
    つまり、`start` スクリプトがある場合、`pdm run start` と `pdm start` の両方を実行できます。
    ただし、`install` スクリプトがある場合、`pdm run install` のみがそれを実行し、`pdm install` は組み込みの `install` コマンドを実行します。

PDM は 4 種類のスクリプトをサポートしています:

### `cmd`

プレーンテキストスクリプトは通常のコマンドとして扱われますが、明示的に指定することもできます:

```toml
[tool.pdm.scripts]
start = {cmd = "flask run -p 54321"}
```

場合によっては、パラメータ間にコメントを追加したい場合など、コマンドを配列として指定する方が便利です:

```toml
[tool.pdm.scripts]
start = {cmd = [
    "flask",
    "run",
    # ここに常にポート 54321 を使用することに関する重要なコメント
    "-p", "54321"
]}
```

### `shell`

シェルスクリプトを使用して、パイプラインや出力リダイレクトなどのシェル固有のタスクを実行できます。
これは基本的に `subprocess.Popen()` を `shell=True` で実行します:

```toml
[tool.pdm.scripts]
filter_error = {shell = "cat error.log|grep CRITICAL > critical.log"}
```

### `call`

スクリプトは `<module_name>:<func_name>` の形式で Python 関数を呼び出すように定義することもできます:

```toml
[tool.pdm.scripts]
foobar = {call = "foo_package.bar_module:main"}
```

関数にリテラル引数を渡すこともできます:

```toml
[tool.pdm.scripts]
foobar = {call = "foo_package.bar_module:main('dev')"}
```

### `composite`

このスクリプトは他の定義されたスクリプトを実行できます:

```toml
[tool.pdm.scripts]
lint = "flake8"
test = "pytest"
all = {composite = ["lint", "test"]}
```

`pdm run all` を実行すると、最初に `lint` が実行され、次に `lint` が成功した場合に `test` が実行されます。

+++ 2.13.0

デフォルトの動作を上書きして、失敗後に残りのスクリプトの実行を続行するには、`keep_going` オプションを `true` に設定します:

```toml
[tool.pdm.scripts]
lint = "flake8"
test = "pytest"
all.composite = ["lint", "test"]
all.keep_going = true
```

`keep_going` が `true` に設定されている場合、複合スクリプトの戻りコードはすべて成功した場合は '0'、または最後に失敗した個々のスクリプトのコードになります。

呼び出されたスクリプトに引数を提供することもできます:

```toml
[tool.pdm.scripts]
lint = "flake8"
test = "pytest"
all = {composite = ["lint mypackage/", "test -v tests/"]}
```

!!! note
    コマンドラインで渡された引数は、呼び出された各タスクに渡されます。

複数のコマンドを組み合わせるために `composite` スクリプトを使用することもできます:

```toml
[tool.pdm.scripts]
mytask.composite = [
    "echo 'Hello'",
    "echo 'World'"
]
```

## スクリプトオプション

### `env`

現在のシェルで設定されたすべての環境変数は `pdm run` によって参照され、実行時に展開されます。
さらに、`pyproject.toml` にいくつかの固定環境変数を定義することもできます:

```toml
[tool.pdm.scripts]
start.cmd = "flask run -p 54321"
start.env = {FOO = "bar", FLASK_DEBUG = "1"}
```

複合辞書を定義するために [TOML の構文](https://github.com/toml-lang/toml) を使用する方法に注意してください。

!!! note "環境変数の置換について"
    スクリプト仕様の変数はすべてのスクリプトタイプで置換できます。`cmd` スクリプトでは、すべてのプラットフォームで `${VAR}` 構文のみがサポートされますが、`shell` スクリプトでは構文はプラットフォーム依存です。たとえば、Windows の cmd は `%VAR%` を使用し、bash は `$VAR` を使用します。

!!! note
    複合タスクレベルで指定された環境変数は、呼び出されたタスクによって定義されたものを上書きします。

### `env_file`

すべての環境変数を dotenv ファイルに保存し、PDM に読み取らせることもできます:

```toml
[tool.pdm.scripts]
start.cmd = "flask run -p 54321"
start.env_file = ".env"
```

dotenv ファイル内の変数は既存の環境変数を上書きしません。
既存の環境変数を上書きするために dotenv ファイルを使用する場合は、次のようにします:

```toml
[tool.pdm.scripts]
start.cmd = "flask run -p 54321"
start.env_file.override = ".env"
```

!!! note "環境変数の読み込み順序"
    異なるソースから読み込まれる環境変数は次の順序で読み込まれ、最終的なソースリストが構築されます:

    1. OS 環境変数
    2. プロジェクト環境変数（`PDM_PROJECT_ROOT`、`PATH`、`VIRTUAL_ENV` など）
    3. `env_file` で指定された dotenv ファイル
    4. `env` で指定された環境変数マッピング

    後のソースからの環境変数は、前のソースからのものを上書きします。
    複合タスクレベルで指定された dotenv ファイルは、呼び出されたタスクによって定義されたものを上書きします。

    環境変数は、前のソースから読み込まれた別の環境変数を参照することができます。たとえば:

    ```
    VAR=42
    FOO=hello-${VAR}
    ```
    は `FOO=hello-42` になります。参照には `${VAR:-default}` 構文を使用してデフォルト値を含めることもできます。

### `working_dir`

+++ 2.13.0

スクリプトの現在の作業ディレクトリを設定できます:

```toml
[tool.pdm.scripts]
start.cmd = "flask run -p 54321"
start.working_dir = "subdir"
```

相対パスはプロジェクトルートに対して解決されます。

### `site_packages`

実行環境が外部の Python インタープリターから適切に分離されていることを確認するために、
選択されたインタープリターの site-packages は `sys.path` に読み込まれません。ただし、次のいずれかの条件が満たされる場合を除きます:

1. 実行可能ファイルが `PATH` から取得され、`__pypackages__` フォルダー内にない場合。
2. `-s/--site-packages` フラグが `pdm run` に続く場合。
3. スクリプトテーブルまたはグローバル設定キー `_` に `site_packages = true` がある場合。

PEP 582 が有効になっている場合（`pdm run` プレフィックスなしで実行される場合）、site-packages は常に読み込まれます。

### 共有オプション

`pdm run` で実行されるすべてのタスクにオプションを共有したい場合は、
`[tool.pdm.scripts]` テーブルの特別なキー `_` の下に書くことができます:

```toml
[tool.pdm.scripts]
_.env_file = ".env"
start = "flask run -p 54321"
migrate_db = "flask db upgrade"
```

さらに、タスク内では、`PDM_PROJECT_ROOT` 環境変数がプロジェクトルートに設定されます。

### 引数プレースホルダー

デフォルトでは、すべてのユーザー提供の追加引数はコマンドに単純に追加されます（または `composite` タスクの場合はすべてのコマンドに追加されます）。

ユーザー提供の追加引数をより詳細に制御したい場合は、`{args}` プレースホルダーを使用できます。
これはすべてのスクリプトタイプで利用可能であり、それぞれに適切に補間されます:

```toml
[tool.pdm.scripts]
cmd = "echo '--before {args} --after'"
shell = {shell = "echo '--before {args} --after'"}
composite = {composite = ["cmd --something", "shell {args}"]}
```

次の補間が生成されます（これらは実際のスクリプトではなく、補間を説明するためのものです）:

```shell
$ pdm run cmd --user --provided
--before --user --provided --after
$ pdm run cmd
--before --after
$ pdm run shell --user --provided
--before --user --provided --after
$ pdm run shell
--before --after
$ pdm run composite --user --provided
cmd --something
shell --before --user --provided --after
$ pdm run composite
cmd --something
shell --before --after
```

ユーザー引数が提供されていない場合に使用されるデフォルト値をオプションで提供できます:

```toml
[tool.pdm.scripts]
test = "echo '--before {args:--default --value} --after'"
```

次のようになります:

```shell
$ pdm run test --user --provided
--before --user --provided --after
$ pdm run test
--before --default --value --after
```

!!! note
    プレースホルダーが検出されると、引数は追加されなくなります。
    これは `composite` スクリプトにとって重要です。なぜなら、サブタスクの 1 つでプレースホルダーが検出されると、引数はサブタスクに追加されなくなるためです。
    引数が必要なすべてのネストされたコマンドにプレースホルダーを明示的に渡す必要があります。

!!! note
    `call` スクリプトは `{args}` プレースホルダーをサポートしていません。これらは `sys.argv` に直接アクセスして複雑なケースを処理できます。

### `{pdm}` プレースホルダー

複数の PDM インストールがある場合や、`pdm` が異なる名前でインストールされている場合があります。
これは、たとえば CI/CD の状況や、異なるリポジトリで異なる PDM バージョンを使用する場合に発生する可能性があります。
スクリプトをより堅牢にするために、スクリプトを実行している PDM エントリポイントを使用するために `{pdm}` を使用できます。
これは `{sys.executable} -m pdm` に展開されます。

```toml
[tool.pdm.scripts]
whoami = { shell = "echo `{pdm} -V` was called as '{pdm} -V'" }
```

次の出力が生成されます:

```shell
$ pdm whoami
PDM, version 0.1.dev2501+g73651b7.d20231115 was called as /usr/bin/python3 -m pdm -V

$ pdm2.8 whoami
PDM, version 2.8.0 was called as <snip>/venvs/pdm2-8/bin/python -m pdm -V
```

!!! note
    上記の例では PDM 2.8 を使用していますが、この機能は 2.10 シリーズで導入され、ショーケースのためにのみバックポートされました。

## スクリプトのリストを表示する

利用可能なスクリプトショートカットのリストを表示するには、`pdm run --list/-l` を使用します:

```bash
$ pdm run --list
╭─────────────┬───────┬───────────────────────────╮
│ Name        │ Type  │ Description               │
├─────────────┼───────┼───────────────────────────┤
│ test_cmd    │ cmd   │ flask db upgrade          │
│ test_script │ call  │ call a python function    │
│ test_shell  │ shell │ shell command             │
╰─────────────┴───────┴───────────────────────────╯
```

スクリプトの説明を含む `help` オプションを追加すると、上記の出力の `Description` 列に表示されます。

!!! note
    名前がアンダースコア（`_`）で始まるタスクは内部（ヘルパーなど）と見なされ、リストには表示されません。

## プレ & ポストスクリプト

`npm` のように、PDM もプレおよびポストスクリプトによるタスクの構成をサポートしており、プレスクリプトは指定されたタスクの前に実行され、ポストスクリプトは後に実行されます。

```toml
[tool.pdm.scripts]
pre_compress = "{{ `compress` スクリプトの前に実行 }}"
compress = "tar czvf compressed.tar.gz data/"
post_compress = "{{ `compress` スクリプトの後に実行 }}"
```

この例では、`pdm run compress` はこれらの 3 つのスクリプトを順番に実行します。

!!! note "パイプラインは早期に失敗します"
    プレ - 自身 - ポストスクリプトのパイプラインでは、失敗が発生すると後続の実行がキャンセルされます。

## フックスクリプト

特定の状況下で PDM はいくつかの特別なフックスクリプトを実行します:

- `post_init`: `pdm init` の後に実行
- `pre_install`: パッケージのインストール前に実行
- `post_install`: パッケージのインストール後に実行
- `pre_lock`: 依存関係の解決前に実行
- `post_lock`: 依存関係の解決後に実行
- `pre_build`: ディストリビューションのビルド前に実行
- `post_build`: ディストリビューションのビルド後に実行
- `pre_publish`: ディストリビューションの公開前に実行
- `post_publish`: ディストリビューションの公開後に実行
- `pre_script`: 任意のスクリプトの前に実行
- `post_script`: 任意のスクリプトの後に実行
- `pre_run`: スクリプトの呼び出し前に一度実行
- `post_run`: スクリプトの呼び出し後に一度実行

!!! note
    プレ & ポストスクリプトは引数を受け取ることができません。

!!! note "名前の競合を避ける"
    `[tool.pdm.scripts]` テーブルに `install` スクリプトが存在する場合、`pre_install` スクリプトは `pdm install` と `pdm run install` の両方によってトリガーされる可能性があります。したがって、予約された名前を使用しないことをお勧めします。

!!! note
    複合タスクもプレおよびポストスクリプトを持つことができます。
    呼び出されたタスクはそれぞれのプレおよびポストスクリプトを実行します。

## スクリプトのスキップ

スクリプトを実行するが、そのフックやプレおよびポストスクリプトを実行しない場合があるため、
すべてのフック、プレおよびポストを無効にする `--skip=:all` があります。
また、すべての `pre_*` フックをスキップする `--skip=:pre` とすべての `post_*` フックをスキップする `--skip=:post` もあります。

プレスクリプトは必要だがポストスクリプトは不要な場合や、
複合タスクのすべてのタスクを実行するが、1 つのタスクは実行しない場合があります。
これらのユースケースのために、スキップするタスクやフックの名前のリストを受け入れるより詳細な `--skip` パラメータがあります。

```bash
pdm run --skip pre_task1,task2 my-composite
```

このコマンドは `my-composite` タスクを実行し、`pre_task1` フックと `task2` およびそのフックをスキップします。

スキップリストを `PDM_SKIP_HOOKS` 環境変数に提供することもできますが、`--skip` パラメータが指定されると上書きされます。

フックとプレ/ポストスクリプトの動作に関する詳細は[専用のフックページ](hooks.md) を参照してください。
