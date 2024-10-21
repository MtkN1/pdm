# ビルドと公開

ライブラリを開発している場合、プロジェクトに依存関係を追加し、コーディングを完了した後、パッケージをビルドして公開する時が来ました。これは、次のコマンド 1 つで簡単に行えます:

```bash
pdm publish
```

これにより、自動的にホイールとソースディストリビューション (sdist) がビルドされ、PyPI インデックスにアップロードされます。

PyPI では、パッケージを公開するために API トークンが必要です。ユーザー名として `__token__` を使用し、パスワードとして API トークンを使用できます。

PyPI 以外のリポジトリを指定するには、`--repository` オプションを使用します。パラメータは、アップロード URL または構成ファイルに保存されたリポジトリの名前のいずれかです。

```bash
pdm publish --repository testpypi
pdm publish --repository https://test.pypi.org/legacy/
```

## 信頼できるパブリッシャーで公開する

リリースワークフローで PyPI トークンを公開する必要がないように、PyPI に信頼できるパブリッシャーを構成できます。これを行うには、[ガイド](https://docs.pypi.org/trusted-publishers/adding-a-publisher/) に従ってパブリッシャーを追加し、以下のように GitHub Actions ワークフローを記述します:

```yaml
on:
  release:
    types: [published]


jobs:
  pypi-publish:
    name: upload release to PyPI
    runs-on: ubuntu-latest
    permissions:
      # This permission is needed for private repositories.
      contents: read
      # IMPORTANT: this permission is mandatory for trusted publishing
      id-token: write
    steps:
      - uses: actions/checkout@v4

      - uses: pdm-project/setup-pdm@v4

      - name: Publish package distributions to PyPI
        run: pdm publish
```

## ビルドと公開を別々に行う

パッケージをビルドしてアップロードするのを 2 つのステップで行うこともできます。これにより、アップロード前にビルドされたアーティファクトを検査することができます。

```bash
pdm build
```

使用されるバックエンドに応じて、ビルドプロセスを制御するための多くのオプションがあります。詳細については、[ビルド構成](../reference/build.md) セクションを参照してください。

アーティファクトは `dist/` に作成され、PyPI にアップロードできます。

```bash
pdm publish --no-build
```
