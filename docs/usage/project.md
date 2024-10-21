# 新しいプロジェクト

まず、[`pdm init`](../reference/cli.md#init) を使用して新しいプロジェクトを作成します:

```bash
mkdir my-project && cd my-project
pdm init
```

いくつかの質問に答える必要があります。これにより、PDM が `pyproject.toml` ファイルを作成するのに役立ちます。
`pdm init` の詳細な使用方法については、[テンプレートからプロジェクトを作成する](./template.md) を参照してください。

## Python インタープリターを選択する

まず、マシンにインストールされている Python バージョンのリストから Python インタープリターを選択する必要があります。インタープリターのパスは `.pdm-python` に保存され、後続のコマンドで使用されます。後で [`pdm use`](../reference/cli.md#use) を使用して変更することもできます。

また、`PDM_PYTHON` 環境変数を介して Python インタープリターのパスを指定することもできます。設定されている場合、`.pdm-python` に保存されたパスは無視されます。

!!! warning "既存の環境を使用する"
既存の環境を使用することを選択した場合、たとえば `conda` によって作成された環境を再利用する場合、PDM は `pdm sync --clean` または `pdm remove` を実行するときに `pyproject.toml` または `pdm.lock` に記載されていない依存関係を削除することに注意してください。これにより、破壊的な結果が生じる可能性があります。したがって、複数のプロジェクト間で環境を共有しないようにしてください。

### PDM で Python インタープリターをインストールする

+++ 2.13.0

PDM は、`pdm python install` コマンドを使用して [@indygreg's python-build-standalone](https://github.com/indygreg/python-build-standalone) から追加の Python インタープリターをインストールすることをサポートしています。たとえば、CPython 3.9.8 をインストールするには:

```bash
pdm python install 3.9.8
```

`pdm python install --list` を使用して、利用可能なすべての Python バージョンを表示できます。

これにより、Python インタープリターが `python.install_root` 構成で指定された場所にインストールされます。

現在インストールされている Python インタープリターを一覧表示します:

```bash
pdm python list
```

インストールされた Python インタープリターを削除します:

```bash
pdm python remove 3.9.8
```

スレッドセーフな Python インタープリターをインストールします:

```bash
pdm python install 3.13t
```

!!! TIP "Rye とインストールを共有する"

    PDM は [Rye](https://rye-up.com) と同じソースを使用して Python インタープリターをインストールします。同時に Rye を使用している場合、`python.install_root` を Rye と同じディレクトリに設定して Python インタープリターを共有できます:

    ```bash
    pdm config python.install_root ~/.rye/py
    ```

    その後、`rye toolchain` または `pdm python` を使用してインストールを管理できます。

### `requires-python` に基づくインストール戦略

+++ 2.16.0

Python `version` が指定されていない場合、PDM は `pyproject.toml` の `requires-python` に基づいて現在のプラットフォーム/アーキテクチャの組み合わせに最適なものをインストールしようとします（pyproject.toml または requires-python 属性が利用できない場合、すべてのインストール可能な Python インタープリターが考慮されます）。

デフォルトの戦略は `maximum` であり、最高の cPython インタープリター バージョンがインストールされます。

`minimum` が好まれる場合は、`--min` オプションを使用し、`version` を空のままにします。

```bash
pdm python install --min
```

同じ原則が [`pdm use`](../reference/cli.md#use) にも適用されます（自動インストール機能を含む）。これにより、CI/CD や「既存の pyproject.toml での新しいスタート」のユースケースに適した無人セットアップ コマンドになります。

### 仮想環境を使用するかどうか

Python インタープリターを選択した後、PDM はプロジェクトの仮想環境を作成するかどうかを尋ねます。
**はい** を選択すると、PDM はプロジェクトのルートディレクトリに仮想環境を作成し、それをプロジェクトの Python インタープリターとして使用します。

選択した Python インタープリターが仮想環境にある場合、PDM はそれをプロジェクト環境として使用し、依存関係をそこにインストールします。
それ以外の場合、プロジェクトのルートに `__pypackages__` が作成され、依存関係がそこにインストールされます。

これら 2 つのアプローチの違いについては、ドキュメントの対応するセクションを参照してください:

- [仮想環境](./venv.md)
- [`__pypackages__`(PEP 582)](./pep582.md)

## ライブラリまたはアプリケーション

ライブラリとアプリケーションは多くの点で異なります。簡単に言えば、ライブラリは他のプロジェクトによってインストールおよび使用されることを意図したパッケージです。ほとんどの場合、PyPI にアップロードする必要もあります。一方、アプリケーションはエンドユーザーに直接向けられており、いくつかの本番環境にデプロイする必要がある場合があります。

PDM では、ライブラリを作成することを選択した場合、PDM は `pyproject.toml` ファイルに `name`、`version` フィールド、および [ビルドバックエンド](../reference/build.md) 用の `[build-system]` テーブルを追加します。これは、プロジェクトをビルドおよび配布する必要がある場合にのみ役立ちます。したがって、プロジェクトをアプリケーションからライブラリに変更する場合は、これらのフィールドを手動で `pyproject.toml` に追加する必要があります。また、ライブラリ プロジェクトは、`pdm install` または `pdm sync` を実行すると環境にインストールされます。ただし、`--no-self` が指定されていない限りです。

`pyproject.toml` には、`[tool.pdm]` テーブルの下に `distribution` フィールドがあります。これが true に設定されている場合、PDM はプロジェクトをライブラリとして扱います。

## `requires-python` を指定する

プロジェクトに適切な `requires-python` 値を設定する必要があります。これは、依存関係の解決方法に影響を与える重要なプロパティです。基本的に、各パッケージの `requires-python` はプロジェクトの `requires-python` 範囲をカバーする必要があります。たとえば、次のセットアップを考えてみましょう:

- プロジェクト: `requires-python = ">=3.9"`
- パッケージ `foo`: `requires-python = ">=3.7,<3.11"`

依存関係を解決すると、`ResolutionImpossible` が発生します:

```bash
Unable to find a resolution because the following dependencies don't work
on all Python versions defined by the project's `requires-python`
```

依存関係の `requires-python` が `>=3.7,<3.11` であるため、プロジェクトの `requires-python` 範囲 `>=3.9` をカバーしていません。言い換えれば、プロジェクトは Python 3.9、3.10、3.11（およびそれ以降）で動作することを約束していますが、依存関係は Python 3.11（またはそれ以上）をサポートしていません。PDM は `requires-python` 範囲内のすべての Python バージョンで動作するクロスプラットフォームのロックファイルを作成するため、有効な解決策を見つけることができません。
これを修正するには、`requires-python` に最大バージョンを追加する必要があります。たとえば、`>=3.9,<3.11` のようにします。

`requires-python` の値は [PEP 440 で定義されているバージョン指定子](https://peps.python.org/pep-0440/#version-specifiers) です。以下にいくつかの例を示します:

| `requires-python`       | 意味                                  |
| ----------------------- | ---------------------------------------- |
| `>=3.7`                 | Python 3.7 以上                     |
| `>=3.7,<3.11`           | Python 3.7、3.8、3.9、および 3.10            |
| `>=3.6,!=3.8.*,!=3.9.*` | Python 3.6 以上、ただし 3.8 および 3.9 を除く |

## 古い Python バージョンで作業する

--- 2.19.0

    PDM は現在、プロジェクトの Python バージョンとして 3.8 以上をサポートしています。

PDM は Python 3.8 以上で動作しますが、**作業プロジェクト** の Python バージョンを低くすることもできます。ただし、プロジェクトがライブラリであり、ビルド、公開、またはインストールする必要がある場合は、使用している PEP 517 ビルドバックエンドが必要な最低 Python バージョンをサポートしていることを確認してください。たとえば、デフォルトのバックエンド `pdm-backend` は Python 3.7+ でのみ動作するため、Python 3.6 でプロジェクトを実行して [`pdm build`](../reference/cli.md#build) を実行するとエラーが発生します。ほとんどのモダンなビルドバックエンドは Python 3.6 以下のサポートを終了しているため、Python バージョンを 3.7+ にアップグレードすることを強くお勧めします。以下は、一般的に使用されるビルドバックエンドのサポートされている Python 範囲です。PEP 621 をサポートしていないものはリストしていません。PDM はそれらと連携できないためです。

| バックエンド               | サポートされている Python | PEP 621 のサポート |
| --------------------- | ---------------- | --------------- |
| `pdm-backend`         | `>=3.7`          | はい             |
| `setuptools>=60`      | `>=3.7`          | 実験的    |
| `hatchling`           | `>=3.7`          | はい             |
| `flit-core>=3.4`      | `>=3.6`          | はい             |
| `flit-core>=3.2,<3.4` | `>=3.4`          | はい             |

プロジェクトがアプリケーション（つまり、`name` メタデータがない）である場合、上記のバックエンドの制限は適用されません。したがって、ビルドバックエンドが必要ない場合は、任意の Python バージョン `>=2.7` を使用できます。

## 他のパッケージマネージャーからプロジェクトをインポートする

すでに Pipenv や Poetry などの他のパッケージマネージャーツールを使用している場合、PDM に移行するのは簡単です。
PDM は `import` コマンドを提供しているため、プロジェクトを手動で初期化する必要はありません。現在サポートされているのは次のとおりです:

1. Pipenv の `Pipfile`
2. Poetry の `pyproject.toml` 内のセクション
3. Flit の `pyproject.toml` 内のセクション
4. pip が使用する `requirements.txt` 形式
5. setuptools の `setup.py`（プロジェクト環境に `setuptools` がインストールされている必要があります。これを行うには、venv の場合は `venv.with_pip` を true に設定し、`__pypackages__` の場合は `pdm add setuptools` を設定します）

また、[`pdm init`](../reference/cli.md#init) または [`pdm install`](../reference/cli.md#install) を実行しているときに、PDM プロジェクトがまだ初期化されていない場合、PDM はインポートする可能性のあるファイルを自動検出できます。

!!! info
`setup.py` を変換するには、プロジェクトインタープリターでファイルを実行します。インタープリターに `setuptools` がインストールされており、`setup.py` が信頼できることを確認してください。

## バージョン管理と連携する

`pyproject.toml` ファイルをコミットする必要があります。`pdm.lock` および `pdm.toml` ファイルをコミットする必要があります。`.pdm-python` ファイルをコミットしないでください。

`pyproject.toml` ファイルは、PDM に必要なプロジェクトのビルドメタデータと依存関係が含まれているため、コミットする必要があります。
また、他の Python ツールによって構成のために一般的に使用されます。`pyproject.toml` ファイルの詳細については、[Pip ドキュメント](https://pip.pypa.io/en/stable/reference/build-system/pyproject-toml/) を参照してください。

`pdm.lock` ファイルをコミットする必要があります。これにより、すべてのインストーラーが同じバージョンの依存関係を使用することが保証されます。
依存関係を更新する方法については、[既存の依存関係を更新する](./dependency.md#update-existing-dependencies) を参照してください。

`pdm.toml` にはいくつかのプロジェクト全体の構成が含まれており、共有するためにコミットすることが役立つ場合があります。

`.pdm-python` は、**現在の** プロジェクトで使用されている **Python パス** を保存し、共有する必要はありません。

## 現在の Python 環境を表示する

```bash
$ pdm info
PDM version:
  2.0.0
Python Interpreter:
  /opt/homebrew/opt/python@3.9/bin/python3.9 (3.9)
Project Root:
  /Users/fming/wkspace/github/test-pdm
Project Packages:
  /Users/fming/wkspace/github/test-pdm/__pypackages__/3.9

# 環境情報を表示する
$ pdm info --env
{
  "implementation_name": "cpython",
  "implementation_version": "3.8.0",
  "os_name": "nt",
  "platform_machine": "AMD64",
  "platform_release": "10",
  "platform_system": "Windows",
  "platform_version": "10.0.18362",
  "python_full_version": "3.8.0",
  "platform_python_implementation": "CPython",
  "python_version": "3.8",
  "sys_platform": "win32"
}
```

[このコマンド](../reference/cli.md#info) は、プロジェクトで使用されているモードを確認するのに役立ちます:

- **Project Packages** が `None` の場合、[virtualenv モード](./venv.md) が有効です。
- それ以外の場合、[PEP 582 モード](./pep582.md) が有効です。

これで、新しい PDM プロジェクトが設定され、`pyproject.toml` ファイルが作成されました。`pyproject.toml` を適切に記述する方法については、[メタデータセクション](../reference/pep621.md) を参照してください。
