# 特定のプラットフォームまたは Python バージョン用にロックする

+++ 2.17.0

デフォルトでは、PDM は [`pyproject.toml` の `requires-python`](./project.md#specify-requires-python) で指定された Python バージョン内のすべてのプラットフォームで動作するロックファイルを作成しようとします。これは開発中に非常に便利です。開発環境でロックファイルを生成し、このロックファイルを使用して CI/CD や本番環境で同じ依存関係バージョンを再現できます。

ただし、このアプローチが機能しない場合もあります。たとえば、プロジェクトや依存関係にプラットフォーム固有の依存関係がある場合や、Python バージョンに応じた条件付き依存関係がある場合などです。次のような場合です。

```toml
[project]
name = "myproject"
requires-python = ">=3.8"
dependencies = [
    "numpy<1.25; python_version < '3.9'",
    "numpy>=1.25; python_version >= '3.9'",
    "pywin32; sys_platform == 'win32'",
]
```

この場合、すべてのプラットフォームと Python バージョン（`>=3.8`）で各パッケージの単一の解決を得ることはほぼ不可能です。代わりに、特定のプラットフォームまたは Python バージョン用のロックファイルを作成する必要があります。

## ロックファイルを生成する際のロックターゲットの指定

PDM は、ロックファイルを生成する際に 1 つ以上の環境基準を指定することをサポートしています。これらの基準には次のものが含まれます。

- `--python=<PYTHON_RANGE>`: [PEP 440](https://www.python.org/dev/peps/pep-0440/) 互換の Python バージョン指定子。たとえば、`--python=">=3.8,<3.10"` は Python バージョン `>=3.8` および `<3.10` 用のロックファイルを生成します。便宜上、`--python=3.10` は `--python=">=3.10"` と同等であり、Python 3.10 以上のバージョンに対して解決します。
- `--platform=<PLATFORM>`: プラットフォーム指定子。たとえば、`pdm lock --platform=linux` は Linux x86_64 プラットフォーム用のロックファイルを生成します。利用可能なオプションは次のとおりです。
    * `linux`
    * `windows`
    * `macos`
    * `alpine`
    * `windows_amd64`
    * `windows_x86`
    * `windows_arm64`
    * `macos_arm64`
    * `macos_x86_64`
    * `macos_X_Y_arm64`
    * `macos_X_Y_x86_64`
    * `manylinux_X_Y_x86_64`
    * `manylinux_X_Y_aarch64`
    * `musllinux_X_Y_x86_64`
    * `musllinux_X_Y_aarch64`
- `--implementation=cpython|pypy|pyston`: Python 実装指定子。現在、`cpython`、`pypy`、および `pyston` のみがサポートされています。

いくつかの基準を無視することもできます。たとえば、`--platform=linux` のみを指定すると、生成されたロックファイルは Linux プラットフォームおよびすべての実装に適用されます。

!!! note "`python` 基準と `requires-python`"

    `--python` オプション、またはロックターゲットの `requires-python` 基準は、`pyproject.toml` の `requires-python` によって制限されます。たとえば、`requires-python` が `>=3.8` であり、`--python="<3.11"` を指定した場合、ロックターゲットは `>=3.8,<3.11` になります。

## 別々のロックファイルを作成するか、1 つにマージするか

複数のロックターゲットが必要な場合は、各ターゲットごとに別々のロックファイルを作成するか、1 つのロックファイルにまとめることができます。PDM は両方の方法をサポートしています。

特定のターゲットで別々のロックファイルを作成するには:

```bash
# Linux プラットフォームおよび Python 3.8 用のロックファイルを生成し、結果を py38-linux.lock に書き込みます
pdm lock --platform=linux --python="==3.8.*" --lockfile=py38-linux.lock
```

Linux および Python 3.8 で依存関係をインストールする場合、このロックファイルを使用できます。

```bash
pdm install --lockfile=py38-linux.lock
```

さらに、依存関係グループのサブセットをロックファイルに選択することもできます。詳細については[こちら](./lockfile.md#specify-another-lock-file-to-use)を参照してください。

同じロックファイルを複数のターゲットで使用する場合は、`pdm lock` コマンドに `--append` を追加します。

```bash
# Linux プラットフォームおよび Python 3.8 用のロックファイルを生成し、結果を pdm.lock に追加します
pdm lock --platform=linux --python="==3.8.*" --append
```

単一のロックファイルを使用する利点は、依存関係を更新する際に複数のロックファイルを管理する必要がないことです。ただし、単一のロックファイルでは異なるターゲットに対して異なるロック戦略を指定することはできません。また、ロックの更新にかかる時間も長くなることが予想されます。

さらに、各ロックファイルには 1 つ以上のロックターゲットを含めることができるため、非常に柔軟に使用できます。いくつかのターゲットをロックファイルにマージし、特定のグループとターゲットを別々のロックファイルにロックすることもできます。次のセクションで例を示します。

## 例

以下は `pyproject.toml` の内容です。

```toml
[project]
name = "myproject"
requires-python = ">=3.8"
dependencies = [
    "numpy<1.25; python_version < '3.9'",
    "numpy>=1.25; python_version >= '3.9'",
    "pandas"
]

[project.optional-dependencies]
windows = ["pywin32"]
macos = ["pyobjc"]
```

上記の例では、`numpy` の条件付き依存バージョンと、Windows および MacOS 用のプラットフォーム固有のオプション依存関係があります。Linux、Windows、および MacOS プラットフォーム、および Python 3.8 および 3.9 用のロックファイルを生成したいとします。

```bash
pdm lock --python=">=3.9"
pdm lock --python="<3.9" --append

pdm lock --platform=windows --python=">=3.9" --lockfile=py39-windows.lock --with windows
pdm lock --platform=macos --python=">=3.9" --lockfile=py39-macos.lock --with macos
```
上記のコマンドを順番に実行すると、次の 3 つのロックファイルが生成されます。

- `pdm.lock`: デフォルトのメインロックファイルで、すべてのプラットフォームおよび `>=3.8` の Python バージョンで動作します。プラットフォーム固有の依存関係は含まれていません。このロックファイルには、Python 3.9 以上および以下に適した 2 つのバージョンの `numpy` が含まれています。PDM インストーラーは、Python バージョンに応じて正しいバージョンを選択します。
- `py39-windows.lock`: Windows プラットフォームおよび Python 3.9 以上用のロックファイルで、Windows 用のオプション依存関係が含まれています。
- `py39-macos.lock`: MacOS プラットフォームおよび Python 3.9 以上用のロックファイルで、MacOS 用のオプション依存関係が含まれています。
