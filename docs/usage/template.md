# テンプレートからプロジェクトを作成する

`yarn create` や `npm create` と同様に、PDM もテンプレートからプロジェクトを初期化または作成することをサポートしています。
テンプレートは `pdm init` の位置引数として次の形式のいずれかで指定されます:

- `pdm init django` - テンプレート `https://github.com/pdm-project/template-django` からプロジェクトを初期化します
- `pdm init https://github.com/frostming/pdm-template-django` - Git URL からプロジェクトを初期化します。HTTPS および SSH URL の両方が許可されます。
- `pdm init django@v2` - 特定のブランチまたはタグをチェックアウトします。完全な Git URL もサポートされています。
- `pdm init /path/to/template` - ローカルファイルシステム上のテンプレートディレクトリからプロジェクトを初期化します。
- `pdm init minimal` - 組み込みの "minimal" テンプレートで初期化し、`pyproject.toml` のみを生成します。

そして `pdm init` はデフォルトの組み込みテンプレートを使用します。

プロジェクトは現在のディレクトリに初期化され、同じ名前の既存のファイルは上書きされます。新しいパスでプロジェクトを作成するには、`-p <path>` オプションを使用することもできます。

## テンプレートに貢献する

テンプレート引数の最初の形式に従って、`pdm init <name>` は `https://github.com/pdm-project/template-<name>` にあるテンプレートリポジトリを参照します。テンプレートに貢献するには、テンプレートリポジトリを作成し、所有権を `pdm-project` 組織に移管するリクエストを確立します（リポジトリ設定ページの下部にあります）。組織の管理者がリクエストをレビューし、後続の手順を完了します。移管が受け入れられた場合、リポジトリのメンテナーとして追加されます。

## テンプレートの要件

テンプレートリポジトリは、PEP-621 準拠のメタデータを含む `pyproject.toml` ファイルを持つ pyproject ベースのプロジェクトである必要があります。
他の特別な構成ファイルは必要ありません。

## プロジェクト名の置換

初期化時に、テンプレート内のプロジェクト名は新しいプロジェクトの名前に置き換えられます。これは再帰的な全文検索と置換によって行われます。プロジェクト名からすべての英数字以外の文字をアンダースコアに置き換え、小文字に変換したインポート名も同様に置き換えられます。

たとえば、テンプレート内のプロジェクト名が `foo-project` で、新しいプロジェクト名を `bar-project` にしたい場合、次の置換が行われます:

- すべての `.md` ファイルおよび `.rst` ファイル内の `foo-project` -> `bar-project`
- すべての `.py` ファイル内の `foo_project` -> `bar_project`
- ディレクトリ名内の `foo_project` -> `bar_project`
- ファイル名内の `foo_project.py` -> `bar_project.py`

したがって、インポート名がプロジェクト名から派生していない場合、名前の置換はサポートされません。

## 他のプロジェクトジェネレーターを使用する

より強力なプロジェクトジェネレーターを探している場合は、`--cookiecutter` オプションを使用して [cookiecutter](https://github.com/cookiecutter/cookiecutter) を、`--copier` オプションを使用して [copier](https://github.com/copier-org/copier) を使用できます。

それぞれを使用するには `cookiecutter` および `copier` をインストールする必要があります。これを行うには、`pdm self add <package>` を実行します。
使用するには:

```bash
pdm init --cookiecutter gh:cjolowicz/cookiecutter-hypermodern-python
# または
pdm init --copier gh:pawamoy/copier-pdm --UNSAFE
```
