---
marp: true
theme: gaia
class: invert
paginate: true
footer: "© 2024 QIQB"
---

# 頻繁に利用するコマンドを Taskfile で管理してみた。ついでに自己文書化とインタラクティブ実行にも対応してみた

---

## はじめに

宮永と申します。大阪大学で研究員兼ソフトウェアエンジニアをしています。

---

## イントロ

皆さんは業務の中で頻繁に利用するコマンドとかないでしょうか？

毎回入力するのは面倒だけど、そのために専用のスクリプトを書くのも面倒だし、みたいなやつです。

---

## サンプル

最近ですと私の場合はプライベートネットワークに置かれた RDS にアクセスするためのポートフォワードコマンドとかをよく使います。

```bash
@aws ssm start-session \
--target i-0xxxxxxxxxx \
--document-name AWS-StartPortForwardingSessionToRemoteHost \
--parameters '{"host":["rds-foo-bar.foo-bar.ap-northeast-1.rds.amazonaws.com"],"portNumber":["5432"], "localPortNumber":["5452"]}'
```

---

## 悩み

このコマンドは毎回入力するのが面倒だし、何よりも長いので、いちいちコピペするのも面倒です。

---

## 解決策 1 `alias` を使う

まっさきに思いつくのは`.bashrc`や`.zshrc` にエイリアスを設定することですが私の場合、コマンド自体を忘れてしまいます。

---

## 解決策 2 　`fzf` を使う

次に思いついたのはコマンドを csv 化して、それを `fzf` で選択して実行する方法です。

これはまあまあ良かったのですが、わざわざ `fzf` のスクリプトを作成するのに頭を悩ませるのも面倒です。

`fzf`を使うことでインタラクティブにコマンドを実行できるのは良い発想だなと気づきました。

---

## 解決策 3 `Makefile` を使う

解決策 2 を試したところでもっと簡易的に再現性のある方法でできないかと考えました。
私はよく`Makefile`をタスクランナーとして使うので、これを利用してみることにしました。

---

## ざっくりまとめ

- `alias` はコマンドを忘れる → 文書化が必要
- `fzf` はスクリプトを書くのが面倒 → インタラクティブ実行は良い
- `Makefile` は再現性がある(枯れていて文書としてまとめやすい)

---

## Makefile の自己文書化

Makefile はタスクランナーとして使われることが多い(タスクランナーに使うな！という声も聞こえてきそう 🥹 )ですが、工夫すれば自己文書化することができます。

参考: [Makefile の自己文書化](https://postd.cc/auto-documented-makefile/)

---

## Makefile の自己文書化 - 実装例

```makefile
SHELL := bash
.SHELLFLAGS := -eu -o pipefail -c
.DEFAULT_GOAL := help

.PHONY: default show-cwd delete-ds-store

default:## Display available tasks
	@echo "Available tasks:"
	@echo "  make show-cwd         - Show current working directory"
	@echo "  make delete-ds-store  - Delete all .DS_Store files"


show-cwd: ## Show current working directory
	@echo "current directory: $(shell pwd)"


delete-ds-store: ## Delete all .DS_Store files
	find . -name '.DS_Store' -type f -ls -delete

help: ## Show this help message
	@echo "Usage: make [target]"
	@echo ""
	@echo "Available targets:"
	@echo ""
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(filter-out .env,$(MAKEFILE_LIST)) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

```

---

## Makefile の自己文書化 - 実行例

```bash
❯❯❯ make
Usage: make [target]

Available targets:

default                        Display available tasks
delete-ds-store                Delete all .DS_Store files
help                           Show this help message
show-cwd                       Show current working directory
```

---

## Makefile の自己文書化

自己文書化の仕組みを使えば、コマンドを忘れても簡単に内容確認できそうです。

---

## インタラクティブな実行

`fzf`で頑張っても良いですが、Go で CLI ツールを作ってしまったほうが楽そうです。

`chambracelet/bubbletea` というライブラリを使うと、簡単に CLI アプリケーションを作成できます。

https://github.com/charmbracelet/bubbletea

---

## mk - Makefile をインタラクティブに実行する CLI ツール

<style>
img[alt~="center"] {
  display: block;
  margin: 0 auto;
}
</style>

![w:600 center](https://raw.githubusercontent.com/orangekame3/mk/refs/heads/main/img/demo.gif)

---

## mk - 特徴

- カレントディレクトリの Makefile を読み込んで、インタラクティブに実行できる
- `##`を追加することで自己文書化が可能
- `Vim` ライクなキーバインドで操作可能 ( 重要 :smile_cat:)
- 曖昧検索に対応
- 最近実行したタスクが優先表示される
- ファイル指定(リモートファイルも可)

---

## mk - インストール

### Homebrew

```bash
brew install orangekame3/tap/mk
```

### go install

```bash
go install github.com/orangekame3/mk@latest
```

---

## mk - 使い方

```bash
❯❯❯ mk
```

実際に触ってみてください。

```bash
❯❯❯ mk -f
```

---
