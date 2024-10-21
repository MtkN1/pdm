# ライフサイクルとフック

すべての Python デリバラブルと同様に、プロジェクトは Python プロジェクトのライフサイクルのさまざまなフェーズを経て進行し、PDM はこれらのフェーズの期待されるタスクを実行するためのコマンドを提供します。

また、これらのステップにフックを提供し、次のことができます。

- プラグインが同じ名前の[シグナル][pdm.signals]をリッスンする。
- 開発者が同じ名前のカスタムスクリプトを定義する。

さらに、`pre_invoke` シグナルは、任意のコマンドが呼び出される前に発行され、プラグインがプロジェクトやオプションを事前に変更できるようにします。

組み込みコマンドは現在、次の 3 つのグループに分かれています。

- [初期化フェーズ](#initialization)
- [依存関係管理](#dependencies-management)
- [公開フェーズ](#publication)

インストールフェーズと公開フェーズの間にいくつかの定期的なタスク（ハウスキーピング、リント、テストなど）を実行する必要がある場合があります。これが、PDM が[ユーザースクリプト](#user-scripts)を使用して独自のタスク/フェーズを定義できるようにする理由です。

完全な柔軟性を提供するために、PDM は必要に応じて[いくつかのフックとタスクをスキップ](#skipping)することを許可します。

## 初期化

初期化フェーズは、既存のプロジェクトを初期化するために [`pdm init`](../reference/cli.md#init) コマンドを実行して、プロジェクトのライフタイム中に一度だけ発生する必要があります（`pyproject.toml` ファイルを埋めるためのプロンプト）。

これらは次のフックをトリガーします。

- [`post_init`][pdm.signals.post_init]

```mermaid
flowchart LR
  subgraph pdm-init [pdm init]
    direction LR
    post-init{{Emit post_init}}
    init --> post-init
  end
```

## 依存関係管理

依存関係管理は、開発者が作業を行い、次のことを実行するために必要です。

- `lock`: `pyproject.toml` の要件からロックファイルを計算します。
- `sync`: ロックファイルから PEP582 パッケージを同期（追加/削除/更新）し、現在のプロジェクトを編集可能としてインストールします。
- `add`: 依存関係を追加します。
- `remove`: 依存関係を削除します。

これらのステップは、次のコマンドで直接利用できます。

- [`pdm lock`](../reference/cli.md#lock): `lock` タスクを実行します。
- [`pdm sync`](../reference/cli.md#sync): `sync` タスクを実行します。
- [`pdm install`](../reference/cli.md#install): 必要に応じて `lock` を先行して `sync` タスクを実行します。
- [`pdm add`](../reference/cli.md#add): 依存関係の要件を追加し、再ロックしてから同期します。
- [`pdm remove`](../reference/cli.md#remove): 依存関係の要件を削除し、再ロックしてから同期します。
- [`pdm update`](../reference/cli.md#update): 最新バージョンから依存関係を再ロックし、同期します。

これらは次のフックをトリガーします。

- [`pre_install`][pdm.signals.pre_install]
- [`post_install`][pdm.signals.post_install]
- [`pre_lock`][pdm.signals.pre_lock]
- [`post_lock`][pdm.signals.post_lock]

```mermaid
flowchart LR
  subgraph pdm-install [pdm install]
    direction LR

    subgraph pdm-lock [pdm lock]
      direction TB
      pre-lock{{Emit pre_lock}}
      post-lock{{Emit post_lock}}
      pre-lock --> lock --> post-lock
    end

    subgraph pdm-sync [pdm sync]
      direction TB
      pre-install{{Emit pre_install}}
      post-install{{Emit post_install}}
      pre-install --> sync --> post-install
    end

    pdm-lock --> pdm-sync
  end
```

### Python バージョンの切り替え

これは依存関係管理の特別なケースです。[`pdm use`](../reference/cli.md#use) を使用して現在の Python バージョンを切り替えることができ、新しい Python インタープリターで [`post_use`][pdm.signals.post_use] シグナルを発行します。

```mermaid
flowchart LR
  subgraph pdm-use [pdm use]
    direction LR
    post-use{{Emit post_use}}
    use --> post-use
  end
```

## 公開

パッケージ/ライブラリを公開する準備が整ったら、次の公開タスクが必要になります。

- `build`: アセットをビルド/コンパイルし、Python パッケージ（sdist、wheel）にすべてをパッケージ化します。
- `upload`: パッケージをリモート PyPI インデックスにアップロード/公開します。

これらのステップは、次のコマンドで利用できます。

- [`pdm build`](../reference/cli.md#build)
- [`pdm publish`](../reference/cli.md#publish)

これらは次のフックをトリガーします。

- [`pre_publish`][pdm.signals.pre_publish]
- [`post_publish`][pdm.signals.post_publish]
- [`pre_build`][pdm.signals.pre_build]
- [`post_build`][pdm.signals.post_build]

```mermaid
flowchart LR
  subgraph pdm-publish [pdm publish]
    direction LR
    pre-publish{{Emit pre_publish}}
    post-publish{{Emit post_publish}}

    subgraph pdm-build [pdm build]
      pre-build{{Emit pre_build}}
      post-build{{Emit post_build}}
      pre-build --> build --> post-build
    end

    %% subgraph pdm-upload [pdm upload]
    %%   pre-upload{{Emit pre_upload}}
    %%   post-upload{{Emit post_upload}}
    %%   pre-upload --> upload --> post-upload
    %% end

    pre-publish --> pdm-build --> upload --> post-publish
  end
```

実行は最初の失敗で停止し、フックも含まれます。

## ユーザースクリプト

[ユーザースクリプトは独自のセクションで詳しく説明されています](scripts.md)が、次のことを知っておく必要があります。

- 各ユーザースクリプトは、`pre_*` および `post_*` スクリプトを定義でき、複合スクリプトも含まれます。
- 各 `run` 実行は、[`pre_run`][pdm.signals.pre_run] および [`post_run`][pdm.signals.post_run] フックをトリガーします。
- 各スクリプト実行は、[`pre_script`][pdm.signals.pre_script] および [`post_script`][pdm.signals.post_script] フックをトリガーします。

次の `scripts` 定義を考えます。

```toml
[tool.pdm.scripts]
pre_script = ""
post_script = ""
pre_test = ""
post_test = ""
test = ""
pre_composite = ""
post_composite = ""
composite = {composite = ["test"]}
```

`pdm run test` は次のライフサイクルを持ちます。

```mermaid
flowchart LR
  subgraph pdm-run-test [pdm run test]
    direction LR
    pre-run{{Emit pre_run}}
    post-run{{Emit post_run}}
    subgraph run-test [test task]
      direction TB
      pre-script{{Emit pre_script}}
      post-script{{Emit post_script}}
      pre-test[Execute pre_test]
      post-test[Execute post_test]
      test[Execute test]

      pre-script --> pre-test --> test --> post-test --> post-script
    end

    pre-run --> run-test --> post-run
  end
```

一方、`pdm run composite` は次のようになります。

```mermaid
flowchart LR
  subgraph pdm-run-composite [pdm run composite]
    direction LR
    pre-run{{Emit pre_run}}
    post-run{{Emit post_run}}

    subgraph run-composite [composite task]
      direction TB
      pre-script-composite{{Emit pre_script}}
      post-script-composite{{Emit post_script}}
      pre-composite[Execute pre_composite]
      post-composite[Execute post_composite]

      subgraph run-test [test task]
        direction TB
        pre-script-test{{Emit pre_script}}
        post-script-test{{Emit post_script}}
        pre-test[Execute pre_test]
        post-test[Execute post_test]

        pre-script-test --> pre-test --> test --> post-test --> post-script-test
      end

      pre-script-composite --> pre-composite --> run-test --> post-composite --> post-script-composite
    end

     pre-run --> run-composite --> post-run
  end
```

## スキップ

組み込みコマンドやカスタムユーザースクリプトのタスクとフックの実行を制御するために、`--skip` オプションを使用できます。

これは、スキップするフック/タスク名のカンマ区切りリストと、すべてのフック、すべての `pre_*` フック、およびすべての `post_*` フックをそれぞれスキップするための事前定義された `:all`、`:pre`、および `:post` ショートカットを受け入れます。スキップリストを `PDM_SKIP_HOOKS` 環境変数に提供することもできますが、`--skip` パラメータが指定されると上書きされます。

前のスクリプトブロックを考えると、`:pre,post_test composite` をスキップする `pdm run --skip` を実行すると、次のようにライフサイクルが短縮されます。

```mermaid
flowchart LR
  subgraph pdm-run-composite [pdm run composite]
    direction LR
    post-run{{Emit post_run}}

    subgraph run-composite [composite task]
      direction TB
      post-script-composite{{Emit post_script}}
      post-composite[Execute post_composite]

      subgraph run-test [test task]
        direction TB
        post-script-test{{Emit post_script}}

        test --> post-script-test
      end

      run-test --> post-composite --> post-script-composite
    end

     run-composite --> post-run
  end
```
