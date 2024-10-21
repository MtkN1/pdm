# 高度な使用法

## 自動テスト

### Tox をランナーとして使用する

[Tox](https://tox.readthedocs.io/en/latest/) は、複数の Python バージョンや依存関係セットに対してテストを行うための優れたツールです。
次のように `tox.ini` を構成して、PDM と統合することができます。

```ini
[tox]
env_list = py{36,37,38},lint

[testenv]
setenv =
    PDM_IGNORE_SAVED_PYTHON="1"
deps = pdm
commands =
    pdm install --dev
    pytest tests

[testenv:lint]
deps = pdm
commands =
    pdm install -G lint
    flake8 src/
```

Tox によって作成された仮想環境を使用するには、`pdm config python.use_venv true` を設定していることを確認する必要があります。PDM はその後、[`pdm lock`](../reference/cli.md#lock) からの依存関係を仮想環境にインストールします。専用の venv では、`pytest tests/` のように `pdm run pytest tests/` の代わりにツールを直接実行できます。

テストコマンドで `pdm add/pdm remove/pdm update/pdm lock` を実行しないようにする必要があります。そうしないと、[`pdm lock`](../reference/cli.md#lock) ファイルが予期せず変更されます。追加の依存関係は `deps` 設定で指定できます。さらに、`isolated_build` と `passenv` 設定を上記の例のように設定して、PDM が正しく動作するようにする必要があります。

これらの制約を取り除くために、使用を容易にする Tox プラグイン [tox-pdm](https://github.com/pdm-project/tox-pdm) があります。次のコマンドでインストールできます。

```bash
pip install tox-pdm
```

または、

```bash
pdm add --dev tox-pdm
```

そして、`tox.ini` を次のように整理できます。

```ini
[tox]
env_list = py{36,37,38},lint

[testenv]
groups = dev
commands =
    pytest tests

[testenv:lint]
groups = lint
commands =
    flake8 src/
```

詳細なガイダンスについては、[プロジェクトの README](https://github.com/pdm-project/tox-pdm) を参照してください。

### Nox をランナーとして使用する

[Nox](https://nox.thea.codes/) は、もう一つの優れた自動テストツールです。Tox とは異なり、Nox は設定に標準の Python ファイルを使用します。

Nox で PDM を使用するのは非常に簡単です。以下は `noxfile.py` の例です。

```python hl_lines="4"
import os
import nox

os.environ.update({"PDM_IGNORE_SAVED_PYTHON": "1"})

@nox.session
def tests(session):
    session.run_always('pdm', 'install', '-G', 'test', external=True)
    session.run('pytest')

@nox.session
def lint(session):
    session.run_always('pdm', 'install', '-G', 'lint', external=True)
    session.run('flake8', '--import-order-style', 'google')
```

`PDM_IGNORE_SAVED_PYTHON` を設定して、PDM が仮想環境内の Python を正しく認識できるようにする必要があります。また、`pdm` が `PATH` に存在することを確認してください。
Nox を実行する前に、`python.use_venv` 設定項目が true に設定されていることを確認して、venv の再利用を有効にしてください。

### PEP 582 `__pypackages__` ディレクトリについて

デフォルトでは、[`pdm run`](../reference/cli.md#run) を使用してツールを実行すると、`__pypackages__` がプログラムおよびそれによって作成されたすべてのサブプロセスによって認識されます。これは、これらのツールによって作成された仮想環境も `__pypackages__` 内のパッケージを認識することを意味し、いくつかのケースで予期しない動作を引き起こします。
`nox` では、`noxfile.py` に次の行を追加することでこれを回避できます。

```python
os.environ.pop("PYTHONPATH", None)
```

`tox` では、`PYTHONPATH` はテストセッションに渡されないため、これは問題になりません。さらに、`nox` と `tox` をそれぞれの pipx 環境に配置して、すべてのプロジェクトにインストールする必要がないようにすることをお勧めします。この場合、PEP 582 パッケージも問題になりません。

## 継続的インテグレーションで PDM を使用する

覚えておくべきことは一つだけです -- PDM は Python < 3.7 ではインストールできないため、これらの Python バージョンでプロジェクトをテストする場合は、特定のジョブ/タスクが実行されるターゲット Python バージョンとは異なる Python バージョンで PDM がインストールされていることを確認する必要があります。

幸いなことに、GitHub Action を使用している場合、これを簡単にするための [pdm-project/setup-pdm](https://github.com/marketplace/actions/setup-pdm) があります。
以下は GitHub Actions のワークフローの例ですが、他の CI プラットフォームに適応させることができます。

```yaml
Testing:
  runs-on: ${{ matrix.os }}
  strategy:
    matrix:
      python-version: [3.7, 3.8, 3.9, '3.10', '3.11']
      os: [ubuntu-latest, macOS-latest, windows-latest]

  steps:
    - uses: actions/checkout@v4
    - name: Set up PDM
      uses: pdm-project/setup-pdm@v4
      with:
        python-version: ${{ matrix.python-version }}

    - name: Install dependencies
      run: |
        pdm sync -d -G testing
    - name: Run Tests
      run: |
        pdm run -v pytest tests
```

!!! important "TIPS"
    GitHub Action ユーザー向けに、Ubuntu 仮想環境での[既知の互換性の問題](https://github.com/actions/virtual-environments/issues/2803)があります。
    PDM の並列インストールがそのマシンで失敗した場合、`parallel_install` を `false` に設定するか、`LD_PRELOAD=/lib/x86_64-linux-gnu/libgcc_s.so.1` 環境変数を設定する必要があります。
    これは `pdm-project/setup-pdm` アクションによって既に処理されています。

!!! note
    CI スクリプトが適切なユーザー設定なしで実行される場合、PDM がキャッシュディレクトリを作成しようとするときに権限エラーが発生する可能性があります。
    これを回避するには、書き込み可能なディレクトリに HOME 環境変数を設定できます。例えば：

    ```bash
    export HOME=/tmp/home
    ```

## マルチステージ Dockerfile で PDM を使用する

PDM をマルチステージ Dockerfile で使用して、最初にプロジェクトと依存関係を `__pypackages__` にインストールし、次にこのフォルダーを最終ステージにコピーし、`PYTHONPATH` に追加することができます。

```dockerfile
ARG PYTHON_BASE=3.10-slim
# build stage
FROM python:$PYTHON_BASE AS builder

# install PDM
RUN pip install -U pdm
# disable update check
ENV PDM_CHECK_UPDATE=false
# copy files
COPY pyproject.toml pdm.lock README.md /project/
COPY src/ /project/src

# install dependencies and project into the local packages directory
WORKDIR /project
RUN pdm install --check --prod --no-editable

# run stage
FROM python:$PYTHON_BASE

# retrieve packages from build stage
COPY --from=builder /project/.venv/ /project/.venv
ENV PATH="/project/.venv/bin:$PATH"
# set command/entrypoint, adapt to fit your needs
COPY src /project/src
CMD ["python", "src/__main__.py"]
```

## モノレポを管理するために PDM を使用する

PDM を使用すると、単一のプロジェクト内に複数のサブパッケージを持つことができ、それぞれに独自の `pyproject.toml` ファイルを持つことができます。そして、すべての依存関係をロックするために 1 つの `pdm.lock` ファイルを作成するだけです。サブパッケージはお互いを依存関係として持つことができます。これを実現するための手順は次のとおりです。

`project/pyproject.toml`:

```toml
[tool.pdm.dev-dependencies]
dev = [
    "-e file:///${PROJECT_ROOT}/packages/foo-core",
    "-e file:///${PROJECT_ROOT}/packages/foo-cli",
    "-e file:///${PROJECT_ROOT}/packages/foo-app",
]
```

`packages/foo-cli/pyproject.toml`:

```toml
[project]
dependencies = ["foo-core"]
```

`packages/foo-app/pyproject.toml`:

```toml
[project]
dependencies = ["foo-core"]
```

次に、プロジェクトのルートで `pdm install` を実行すると、すべての依存関係がロックされた `pdm.lock` が作成されます。すべてのサブパッケージは編集可能モードでインストールされます。

詳細については、[🚀 例のリポジトリ](https://github.com/pdm-project/pdm-example-monorepo) を参照してください。

## `pre-commit` のフック

[`pre-commit`](https://pre-commit.com/) は、git フックを集中管理するための強力なフレームワークです。PDM はすでに内部 QA チェックのために `pre-commit` [フック](https://github.com/pdm-project/pdm/blob/main/.pre-commit-config.yaml) を使用しています。PDM はローカルまたは CI パイプラインで実行できるいくつかのフックも公開しています。

### `requirements.txt` をエクスポートする

このフックは、`pdm export` コマンドと任意の有効な引数をラップします。これは、[`pdm lock`](../reference/cli.md#lock) の実際の内容を反映する `requirements.txt` をコードベースにチェックインすることを保証するためのフック（例：CI）として使用できます。

```yaml
# export python requirements
- repo: https://github.com/pdm-project/pdm
  rev: 2.x.y # a PDM release exposing the hook
  hooks:
    - id: pdm-export
      # command arguments, e.g.:
      args: ['-o', 'requirements.txt', '--without-hashes']
      files: ^pdm.lock$
```

### `pdm.lock` が pyproject.toml と一致していることを確認する

このフックは、`pdm lock --check` コマンドと任意の有効な引数をラップします。これは、`pyproject.toml` に依存関係が追加/変更/削除された場合に `pdm.lock` も最新であることを確認するためのフック（例：CI）として使用できます。

```yaml
- repo: https://github.com/pdm-project/pdm
  rev: 2.x.y # a PDM release exposing the hook
  hooks:
    - id: pdm-lock-check
```

### 現在の作業セットを `pdm.lock` と同期する

このフックは、任意の有効な引数とともに `pdm sync` コマンドをラップします。これは、ブランチをチェックアウトまたはマージするたびに現在の作業セットが `pdm.lock` と同期されることを保証するためのフックとして使用できます。システムの資格情報ストアを使用する場合は、*keyring* を `additional_dependencies` に追加します。

```yaml
- repo: https://github.com/pdm-project/pdm
  rev: 2.x.y # a PDM release exposing the hook
  hooks:
    - id: pdm-sync
      additional_dependencies:
        - keyring
```
