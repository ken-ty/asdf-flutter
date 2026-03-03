# ローカルブランチの asdf-flutter をテストする手順

## 前提

- `asdf` がインストール済み
- `~/.asdfrc` に `legacy_version_file = yes` が設定済み（`true` では**動かない**）

```
legacy_version_file = yes
```

## 手順

### 1. ブランチをチェックアウト

PR のブランチを手元に取得する。

```bash
# 例: PR #56
gh pr checkout 56 --repo asdf-community/asdf-flutter
```

または手動で：

```bash
git fetch upstream pull/56/head:support-fvm-legacy-files-with-precedence
git checkout support-fvm-legacy-files-with-precedence
```

### 2. プラグインを再インストール

`asdf plugin add` でローカルパスを指定しても、スクリプトは `~/.asdf/plugins/flutter/` に**コピー**される。
ブランチを切り替えても自動では反映されないため、毎回再インストールが必要。

```bash
asdf plugin remove flutter
asdf plugin add flutter /path/to/asdf-flutter
```

例：

```bash
asdf plugin remove flutter
asdf plugin add flutter /Users/apple/ghq/github.com/ken-ty/asdf-flutter
```

反映確認：

```bash
cat ~/.asdf/plugins/flutter/bin/list-legacy-filenames
```

### 3. テスト実行

テスト用プロジェクトディレクトリで `asdf current flutter` を実行する。

```bash
cd /path/to/your/flutter/project
asdf current flutter
```

#### ケース別テスト

**C-1: `.fvmrc` のみ**

```bash
echo '{"flutter": "3.19.1"}' > .fvmrc
asdf current flutter
# Source が .fvmrc になることを確認
```

**C-2: `.fvm/fvm_config.json` のみ**

```bash
mkdir -p .fvm
echo '{"flutterSdkVersion": "3.19.1"}' > .fvm/fvm_config.json
asdf current flutter
# Source が .fvm/fvm_config.json になることを確認
```

**C-3: 両方あり（`.fvmrc` が優先されることを確認）**

```bash
echo '{"flutter": "3.29.0-stable"}' > .fvmrc
echo '{"flutterSdkVersion": "3.19.1"}' > .fvm/fvm_config.json
asdf current flutter
# Version: 3.29.0-stable, Source: .fvmrc になることを確認
```

**C-4: `.tool-versions` との共存（`.tool-versions` が優先）**

```bash
echo 'flutter 3.32.5-stable' > .tool-versions
echo '{"flutter": "3.19.1"}' > .fvmrc
asdf current flutter
# Source が .tool-versions になることを確認
rm .tool-versions
```

**C-5: ファイルなし**

```bash
asdf current flutter
# No version set になることを確認
```

## トラブルシューティング

### `legacy_version_file` が効かない

`~/.asdfrc` の値を確認する。`true` ではなく `yes` が正しい。

```bash
cat ~/.asdfrc
# legacy_version_file = yes  ← これが正しい
```

### ブランチを変えたのにスクリプトが古い

プラグインを再インストールする（手順 2）。asdf はスクリプトを `~/.asdf/plugins/flutter/` にコピーするため、ブランチ変更は自動反映されない。

```bash
# 現在のスクリプトを確認
cat ~/.asdf/plugins/flutter/bin/list-legacy-filenames
```
