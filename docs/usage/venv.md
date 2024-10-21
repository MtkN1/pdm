# 仮想環境を使用する

[`pdm init`](../reference/cli.md#init) コマンドを実行すると、PDM はプロジェクトで使用する [Python インタープリターを選択する](./project.md#choose-a-python-interpreter) ように求めます。これは、依存関係をインストールし、タスクを実行するための基本的なインタープリターです。

[PEP 582](https://www.python.org/dev/peps/pep-0582/) と比較して、仮想環境はより成熟しており、Python エコシステムや IDE でのサポートが優れています。したがって、特に設定されていない限り、仮想環境がデフォルトモードです。

!!! NOTE "pdm を構成して仮想環境または PEP 582 を使用する"
    デフォルトでは、pdm は PEP 582 の代わりに仮想環境を使用するように構成されています。ただし、この動作は `pdm config python.use_venv False` 構成変数で変更できます。

**プロジェクトインタープリター（`.pdm-python` に保存され、`pdm info` で確認できるインタープリター）が仮想環境からのものである場合、仮想環境が使用されます。**

## 仮想環境の自動作成

デフォルトでは、PDM は他のパッケージマネージャーと同様に仮想環境レイアウトを使用することを好みます。PDM がまだ Python インタープリターを決定していない新しい PDM 管理プロジェクトで最初に `pdm install` を実行すると、PDM は `<project_root>/.venv` に仮想環境を作成し、依存関係をそこにインストールします。`pdm init` の対話型セッションでも、PDM は仮想環境を作成するかどうかを尋ねます。

PDM が仮想環境を作成するために使用するバックエンドを選択できます。現在、3 つのバックエンドがサポートされています:

- [`virtualenv`](https://virtualenv.pypa.io/)（デフォルト）
- `venv`
- `conda`

`pdm config venv.backend [virtualenv|venv|conda]` で変更できます。

+++ 2.13.0

    さらに、`python.use_venv` 構成が `true` に設定されている場合、PDM は Python インタープリターを切り替えるために `pdm use` を使用する際に常に仮想環境を作成しようとします。

## 自分で仮想環境を作成する

任意の Python バージョンで複数の仮想環境を作成できます。

```bash
# 3.8 インタープリターに基づいて仮想環境を作成する
pdm venv create 3.8
# バージョン文字列以外の名前を割り当てる
pdm venv create --name for-test 3.8
# venv をバックエンドとして使用して作成する。3 つのバックエンドをサポート: virtualenv（デフォルト）、venv、conda
pdm venv create --with venv 3.9
```

## 仮想環境の場所

`--name` が指定されていない場合、PDM は `<project_root>/.venv` に仮想環境を作成します。それ以外の場合、仮想環境は `venv.location` 構成で指定された場所に作成されます。
名前の衝突を避けるために、`<project_name>-<path_hash>-<name_or_python_version>` という名前が付けられます。
`pdm config venv.in_project false` でプロジェクト内の仮想環境の作成を無効にできます。すべての仮想環境は `venv.location` に作成されます。

## 他の場所で作成した仮想環境を再利用する

前の手順で作成した仮想環境を PDM に使用させることができます。[`pdm use`](../reference/cli.md#use) を使用します:

```bash
pdm use -f /path/to/venv
```

## 仮想環境の自動検出

プロジェクト構成にインタープリターが保存されていない場合、または `PDM_IGNORE_SAVED_PYTHON` 環境変数が設定されている場合、PDM は使用可能な仮想環境を検出しようとします:

- プロジェクトルートの `venv`、`env`、`.venv` ディレクトリ
- 現在アクティブな仮想環境（`PDM_IGNORE_ACTIVE_VENV` が設定されていない場合）

## このプロジェクトで作成されたすべての仮想環境を一覧表示する

```bash
$ pdm venv list
このプロジェクトで作成された仮想環境:

-  3.8.6: C:\Users\Frost Ming\AppData\Local\pdm\pdm\venvs\test-project-8Sgn_62n-3.8.6
-  for-test: C:\Users\Frost Ming\AppData\Local\pdm\pdm\venvs\test-project-8Sgn_62n-for-test
-  3.9.1: C:\Users\Frost Ming\AppData\Local\pdm\pdm\venvs\test-project-8Sgn_62n-3.9.1
```

## 仮想環境のパスまたは Python インタープリターを表示する

```bash
pdm venv --path for-test
pdm venv --python for-test
```

## 仮想環境を削除する

```bash
$ pdm venv remove for-test
このプロジェクトで作成された仮想環境:
削除されます: C:\Users\Frost Ming\AppData\Local\pdm\pdm\venvs\test-project-8Sgn_62n-for-test, 続行しますか？ [y/N]:y
削除されました C:\Users\Frost Ming\AppData\Local\pdm\pdm\venvs\test-project-8Sgn_62n-for-test
```

## 仮想環境をアクティブにする

`pipenv` や `poetry` が行うようにサブシェルを生成する代わりに、`pdm venv` はシェルを作成せず、アクティベートコマンドをコンソールに表示します。この方法では、現在のシェルを離れることはありません。次に、出力を `eval` に渡して仮想環境をアクティブにできます:

=== "bash/csh/zsh"

    ```bash
    $ eval $(pdm venv activate for-test)
    (test-project-for-test) $  # 仮想環境に入りました
    ```

=== "Fish"

    ```bash
    $ eval (pdm venv activate for-test)
    ```

=== "Powershell"

    ```ps1
    PS1> Invoke-Expression (pdm venv activate for-test)
    ```

    さらに、プロジェクトインタープリターが venv Python である場合、アクティベートの後に名前引数を省略できます。

!!! NOTE
    `venv activate` はプロジェクトで使用される Python インタープリターを切り替え **ません**。環境変数に仮想環境パスを注入することでシェルを変更するだけです。前述の目的のためには、`pdm use` コマンドを使用してください。

詳細な CLI の使用方法については、[`pdm venv`](../reference/cli.md#venv) ドキュメントを参照してください。

!!! TIP "`pdm shell` を探していますか？"
    PDM は `shell` コマンドを提供していません。多くの便利なシェル機能がサブシェルで完全に機能しない可能性があり、すべてのコーナーケースをサポートするためのメンテナンス負担が生じるためです。ただし、次の方法でこの機能を利用できます:

    - `pdm run $SHELL` を使用します。これにより、環境変数が適切に設定されたサブシェルが生成されます。**サブシェルは `exit` または `Ctrl+D` で終了できます。**
    - 仮想環境をアクティブにするシェル関数を追加します。以下は BASH 関数の例で、ZSH でも動作します:

      ```bash
      pdm() {
        local command=$1

        if [[ "$command" == "shell" ]]; then
            eval $(pdm venv activate)
        else
            command pdm $@
        fi
      }
      ```

      この関数を `~/.bashrc` ファイルにコピーして貼り付け、シェルを再起動します。

      `fish` シェルの場合、次の内容を `~/fish/config.fish` または `~/.config/fish/config.fish` に追加できます:

      ```fish
        function pdm
            set cmd $argv[1]

            if test "$cmd" = "shell"
                eval (pdm venv activate)
            else
                command pdm $argv
            end
        end
      ```

      これで `pdm shell` を実行して仮想環境をアクティブにできます。
      **仮想環境は通常の `deactivate` コマンドで非アクティブ化できます。**

## プロンプトのカスタマイズ

仮想環境をアクティブにすると、デフォルトでプロンプトに `{project_name}-{python_version}` が表示されます。

たとえば、プロジェクト名が `test-project` の場合:

```bash
$ eval $(pdm venv activate for-test)
(test-project-3.10) $  # {project_name} == test-project および {python_version} == 3.10
```

仮想環境の作成前に [`venv.prompt`](../reference/configuration.md) 構成または `PDM_VENV_PROMPT` 環境変数でフォーマットをカスタマイズできます（`pdm init` または `pdm venv create` の前に）。
利用可能な変数は次のとおりです:

- `project_name`: プロジェクトの名前
- `python_version`: Python のバージョン（仮想環境で使用される）

```bash
$ PDM_VENV_PROMPT='{project_name}-py{python_version}' pdm venv create --name test-prompt
$ eval $(pdm venv activate test-prompt)
(test-project-py3.10) $
```

## 仮想環境をアクティブにせずにコマンドを実行する

```bash
# スクリプトを実行する
pdm run --venv test test
# パッケージをインストールする
pdm sync --venv test
# インストールされているパッケージを一覧表示する
pdm list --venv test
```

`--venv` フラグまたは `PDM_IN_VENV` 環境変数をサポートする他のコマンドもあります。詳細は [CLI リファレンス](../reference/cli.md) を参照してください。この機能を使用する前に、`pdm venv create --name <name>` で仮想環境を作成する必要があります。

## プロジェクト環境として仮想環境に切り替える

デフォルトでは、`pdm use` を使用して非 venv Python を選択すると、プロジェクトは [PEP 582 モード](./pep582.md) に切り替わります。`--venv` フラグを使用して名前付き仮想環境に切り替えることもできます:

```bash
# test という名前の仮想環境に切り替える
$ pdm use --venv test
# $PROJECT_ROOT/.venv にあるプロジェクト内の venv に切り替える
$ pdm use --venv in-project
```

## 仮想環境モードを無効にする

`pdm config python.use_venv false` で仮想環境の自動作成と自動検出を無効にできます。
**venv が無効になっている場合、選択されたインタープリターが仮想環境からのものであっても、PEP 582 モードが常に使用されます。**

## 仮想環境に pip を含める

デフォルトでは、PDM は仮想環境に `pip` を含めません。
これにより、仮想環境にインストールされるのは **依存関係のみ** であることが保証され、分離が強化されます。

一度だけ `pip` をインストールするには（たとえば、CI で任意の依存関係をインストールしたい場合）:

```bash
# 仮想環境に pip をインストールする
pdm run python -m ensurepip
# 任意の依存関係をインストールする
# これらの依存関係はロックファイルの依存関係と競合しないことが確認されていません！
pdm run python -m pip install coverage
```

または、`--with-pip` を使用して仮想環境を作成します:

```bash
pdm venv create --with-pip 3.9
```

詳細については、[ensurepip ドキュメント](https://docs.python.org/3/library/ensurepip.html) を参照してください。

PDM を仮想環境に `pip` を含めるように永続的に構成するには、[`venv.with_pip`](../reference/configuration.md) 構成を使用できます。
