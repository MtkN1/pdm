<div align="center">
<img src="assets/logo_big.png" alt="PDM logo">
</div>

# 紹介

PDM は、最新の PEP 標準をサポートするモダンな Python パッケージおよび依存関係マネージャーとして説明されています。しかし、それは単なるパッケージマネージャーではありません。さまざまな側面で開発ワークフローを強化します。

<script id="asciicast-jnifN30pjfXbO9We2KqOdXEhB" src="https://asciinema.org/a/jnifN30pjfXbO9We2KqOdXEhB.js" async></script>

## 特徴のハイライト

- 大規模なバイナリ配布物向けのシンプルで高速な依存関係解決ツール。
- [PEP 517] ビルドバックエンド。
- [PEP 621] プロジェクトメタデータ。
- 柔軟で強力なプラグインシステム。
- 多用途なユーザースクリプト。
- [indygreg's python-build-standalone](https://github.com/indygreg/python-build-standalone) を使用して Python をインストールします。
- [pnpm] のようなオプトインの集中インストールキャッシュ。

[pep 517]: https://www.python.org/dev/peps/pep-0517
[pep 621]: https://www.python.org/dev/peps/pep-0621
[pnpm]: https://pnpm.io/motivation#saving-disk-space-and-boosting-installation-speed

## インストール

PDM をインストールするには Python 3.8 以上が必要です。Windows、Linux、macOS を含む複数のプラットフォームで動作します。

!!! note
    プロジェクトをより低い Python バージョンで動作させることもできます。方法については[こちら](usage/project.md#working-with-python-37)を参照してください。

### 推奨インストール方法

PDM をインストールするには Python バージョン 3.8 以上が必要です。

Pip と同様に、PDM は PDM を分離された環境にインストールするインストールスクリプトを提供します。

=== "Linux/Mac"

    ```bash
    curl -sSL https://pdm-project.org/install-pdm.py | python3 -
    ```

=== "Windows"

    ```powershell
    powershell -ExecutionPolicy ByPass -c "irm https://pdm-project.org/install-pdm.py | py -"
    ```

!!! note
    Windows では、オプションの `py` ランチャーがインストールされていない場合（Microsoft ストアから Python をインストールした場合を含む）、`py` を `python` に置き換えてください。

セキュリティ上の理由から、`install-pdm.py` のチェックサムを確認する必要があります。
[install-pdm.py.sha256](https://pdm-project.org/install-pdm.py.sha256) からダウンロードできます。

たとえば、Linux/Mac では次のようにします。

```bash
curl -sSLO https://pdm-project.org/install-pdm.py
curl -sSL https://pdm-project.org/install-pdm.py.sha256 | shasum -a 256 -c -
# インストーラーを実行します
python3 install-pdm.py [options]
```

インストーラーは PDM をユーザーサイトにインストールし、場所はシステムによって異なります。

- Unix の場合は `$HOME/.local/bin`
- MacOS の場合は `$HOME/Library/Python/<version>/bin`
- Windows の場合は `%APPDATA%\Python\Scripts`

スクリプトに追加のオプションを渡して、PDM のインストール方法を制御できます。

```bash
usage: install-pdm.py [-h] [-v VERSION] [--prerelease] [--remove] [-p PATH] [-d DEP]

optional arguments:
  -h, --help            show this help message and exit
  -v VERSION, --version VERSION | envvar: PDM_VERSION
                        Specify the version to be installed, or HEAD to install from the main branch
  --prerelease | envvar: PDM_PRERELEASE    Allow prereleases to be installed
  --remove | envvar: PDM_REMOVE            Remove the PDM installation
  -p PATH, --path PATH | envvar: PDM_HOME  Specify the location to install PDM
  -d DEP, --dep DEP | envvar: PDM_DEPS     Specify additional dependencies, can be given multiple times
```

オプションをスクリプトの後に渡すか、環境変数の値を設定できます。

### その他のインストール方法

=== "Homebrew"

    ```bash
    brew install pdm
    ```

=== "Scoop"

    ```
    scoop bucket add frostming https://github.com/frostming/scoop-frostming.git
    scoop install pdm
    ```

=== "uv"

    ```bash
    uv tool install pdm
    ```

=== "pipx"

    ```bash
    pipx install pdm
    ```

    GitHub リポジトリのヘッドバージョンをインストールします。
    システムに [Git LFS](https://git-lfs.github.com/) がインストールされていることを確認してください。

    ```bash
    pipx install git+https://github.com/pdm-project/pdm.git@main#egg=pdm
    ```

    PDM をすべての機能でインストールするには次のようにします。

    ```bash
    pipx install pdm[all]
    ```

    詳細については、<https://pypa.github.io/pipx/> を参照してください。

=== "pip"

    ```bash
    pip install --user pdm
    ```

=== "asdf"

    [asdf](https://asdf-vm.com/) がインストールされていることを前提としています。
    ```
    asdf plugin add pdm
    asdf local pdm latest
    asdf install pdm
    ```

=== "プロジェクト内"

    [Pyprojectx](https://pyprojectx.github.io/) ラッパースクリプトをプロジェクトにコピーすることで、PDM を (npm スタイルの) 開発依存関係としてプロジェクト内にインストールできます。これにより、異なるプロジェクト/ブランチが異なる PDM バージョンを使用できるようになります。

    [新しいプロジェクトまたは既存のプロジェクトを初期化する](https://pyprojectx.github.io/usage/#initialize-a-new-or-existing-project) には、プロジェクトフォルダーに移動して次のようにします。

    === "Linux/Mac"

        ```
        curl -LO https://github.com/pyprojectx/pyprojectx/releases/latest/download/wrappers.zip && unzip wrappers.zip && rm -f wrappers.zip
        ./pw --add pdm
        ```

    === "Windows"

        ```powershell
        Invoke-WebRequest https://github.com/pyprojectx/pyprojectx/releases/latest/download/wrappers.zip -OutFile wrappers.zip; Expand-Archive -Path wrappers.zip -DestinationPath .; Remove-Item -Path wrappers.zip
        .\pw --add pdm
        ```

    この方法で pdm をインストールする場合、すべての `pdm` コマンドを `pw` ラッパーを通じて実行する必要があります。

    === "Linux/Mac/Windows"

        ```
        ./pw pdm install
        ```

### PDM バージョンの更新

```bash
pdm self update
```

### アンインストール

システムから PDM を削除する必要がある場合は、次のスクリプトを使用できます。

=== "Linux/Mac"

    ```bash
    curl -sSL https://pdm-project.org/install-pdm.py | python3 - --remove
    ```

=== "Windows"

    ```powershell
    powershell -ExecutionPolicy ByPass -c "irm https://pdm-project.org/install-pdm.py | py - --remove"
    ```

Homebrew などのサードパーティのパッケージ管理ツールを使用して PDM をインストールした場合は、ツールのアンインストール方法を使用して PDM をアンインストールすることもできます。たとえば、`brew uninstall pdm` などです。

## パッケージングステータス

[![Packaging status](https://repology.org/badge/vertical-allrepos/pdm.svg)](https://repology.org/project/pdm/versions)

## シェル補完

PDM は Bash、Zsh、Fish、または Powershell 用の補完スクリプトの生成をサポートしています。各シェルの一般的な場所は次のとおりです。

=== "Bash"

    ```bash
    pdm completion bash > /etc/bash_completion.d/pdm.bash-completion # ルート (sudo) が必要です。代替手段については次を参照してください。
    pdm completion bash > ~/.bash_completion # ルート (sudo) は不要です。ユーザーアカウントにのみインストールされます。
    ```

=== "Zsh"

    ```bash
    # ~/.zfunc が fpath に追加されていることを確認し、compinit の前に追加します。
    pdm completion zsh > ~/.zfunc/_pdm
    ```

    Oh-My-Zsh:

    ```bash
    mkdir $ZSH_CUSTOM/plugins/pdm
    pdm completion zsh > $ZSH_CUSTOM/plugins/pdm/_pdm
    ```

    次に、~/.zshrc で pdm プラグインが有効になっていることを確認します。

=== "Fish"

    ```bash
    pdm completion fish > ~/.config/fish/completions/pdm.fish
    ```

=== "Powershell"

    ```ps1
    # 補完スクリプトを保存するディレクトリを作成します。
    mkdir $PROFILE\..\Completions
    echo @'
    Get-ChildItem "$PROFILE\..\Completions\" | ForEach-Object {
        . $_.FullName
    }
    '@ | Out-File -Append -Encoding utf8 $PROFILE
    # スクリプトを生成します。
    Set-ExecutionPolicy Unrestricted -Scope CurrentUser
    pdm completion powershell | Out-File -Encoding utf8 $PROFILE\..\Completions\pdm_completion.ps1
    ```

## Virtualenv と PEP 582

PDM は、仮想環境管理に加えて、オプトイン機能として [PEP 582](https://www.python.org/dev/peps/pep-0582/) の実験的サポートを提供しています。Python ステアリング カウンシルは [PEP 582 を拒否しました][rejected] が、PDM を使用してテストすることはできます。

2 つのモードの詳細については、[仮想環境の使用](usage/venv.md) および [PEP 582 の使用](usage/pep582.md) に関する関連章を参照してください。

[rejected]: https://discuss.python.org/t/pep-582-python-local-packages-directory/963/430

## PDM エコシステム

[Awesome PDM](https://github.com/pdm-project/awesome-pdm) は、素晴らしい PDM プラグインとリソースのキュレーションされたリストです。

## スポンサー

<p align="center">
    <a href="https://cdn.jsdelivr.net/gh/pdm-project/sponsors/sponsors.svg">
        <img src="https://cdn.jsdelivr.net/gh/pdm-project/sponsors/sponsors.svg"/>
    </a>
</p>
