---
marp: true
theme: gaia
class: invert
paginate: true
footer: "@2024 #midosuji_tech"
---

## オレオレコマンドをTaskで管理したついでに自己文書化とインタラクティブ実行にも対応してみた

Takafumi Miyanaga[@orangekame3](https://x.com/orangekame3))

![w:150](slide_qr.png)
↑https://www.orangekame3.net/LT/midousuji-tech/

---

## Who am I?

🫠 みやなが[@orangekame3](https://x.com/orangekame3))です。
大阪大学で研究員兼ソフトウェアエンジニアをしています。
![w:300 center](profile_qr.png)
https://my.prairie.cards/u/orangekame3

---

## イントロ

皆さんは業務の中で頻繁に利用するコマンドとかないでしょうか？

毎回入力するのは面倒だけど、そのために専用のスクリプトを書くのも面倒だし、みたいなやつです。

---

## サンプル

最近ですと、私の場合はプライベートネットワークに置かれた RDS のポートフォワードコマンドとかをよく使います。

```bash
aws ssm start-session \
--target i-0xxxxxxxxxx \
--document-name AWS-StartPortForwardingSessionToRemoteHost \
--parameters '{"host":["rds-foo-bar.foo-bar.ap-northeast-1.rds.amazonaws.com"],"portNumber":["5432"], "localPortNumber":["5452"]}'
```

参考: [SSMポートファーディングでPrivate Subnet内のRDSに接続する \- サーバーワークスエンジニアブログ](https://blog.serverworks.co.jp/ssm-session-manager-rds)

---

## 悩み

🫠　「コマンド、長いよ...」

毎回入力するのが面倒だし、何よりもコマンド自体を忘れてしまうことが多いです。

🫠  「業務効率化のチャンス...？」

---

## 解決策 1 `alias` を使う

まっさきに思いつくのは`.bashrc`や`.zshrc` にエイリアスを設定することですが私の場合、コマンド自体を忘れてしまいます。

```bash
alias portfoward-dev=aws ssm start-session \
--target i-0xxxxxxxxxx \
--document-name AWS-StartPortForwardingSessionToRemoteHost \
--parameters '{"host":["rds-foo-bar.foo-bar.ap-northeast-1.rds.amazonaws.com"],"portNumber":["5432"], "localPortNumber":["5452"]}'
```

🫠 「あのコマンド何だっけ？...　」

---

## 解決策 2 　`fzf` を使う

次に思いついたのはコマンドを csv 化して、それを `fzf` で選択して実行する方法です。

これはまあまあ良かったのですが、わざわざ `fzf` のスクリプトを作成するのに頭を悩ませるのも面倒です。

`fzf`を使うことでインタラクティブにコマンドを実行できるのは良い発想だなと気づきました。
参考: [junegunn/fzf: :cherry\_blossom: A command\-line fuzzy finder](https://github.com/junegunn/fzf)

🫠　「オレオレコマンドすぎるな...」

---

## 解決策 3 `Makefile` を使う

解決策 2 を試したところでもっと簡易的で再現性のある方法でできないかと考えました。

私はよく`Makefile`をタスクランナーとして使うので、これを利用してみることにしました。

🫠　「タスクランナーにMakefile使うな！は受け付けません 🙅‍♂️」

---

## Makefile の自己文書化

Makefile は工夫すれば自己文書化することができます。

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

## Makefile の自己文書化　- まとめ

自己文書化の仕組みを使えば、コマンドを忘れても簡単に内容確認できそうです。

🫠　「忘れん坊にもやさしい...」

---

## ざっくりまとめ

- `alias` はコマンドを忘れる → 文書化が必要
- `fzf` はスクリプトを書くのが面倒 → インタラクティブ実行は良い
- `Makefile` は枯れていて文書としてまとめやすい(ポータブル)

🫠　「これをベースに使いやすい環境をつくろ..」

---

## インタラクティブな実行

`fzf`で頑張っても良いですが、Go で CLI ツールを作ってしまったほうが楽そうです。

`chambracelet/bubbletea` というライブラリを使うと、簡単に CLI アプリケーションを作成できます。

参考 : [charmbracelet/bubbletea: A powerful little TUI framework 🏗](https://github.com/charmbracelet/bubbletea)

🫠　「このライブラリ使うだけでオシャレ感でるじゃん」

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
- `Vim` ライクなキーバインドで操作可能 ( 重要 🫠)
- 曖昧検索に対応
- 最近実行したタスクが優先表示される
- `-f`でファイル指定(リモートファイルも可)

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
❯❯❯ mk -f https://raw.githubusercontent.com/orangekame3/LT/refs/heads/main/midousuji-tech/Makefile
```

---

## Taskfile対応

Taskfile は Go 製のタスクランナーです。
タスクランナーを目的に作られたものなので、Makefile に比べて柔軟性があります。

参考 : [go\-task/task: A task runner / simpler Make alternative written in Go](https://github.com/go-task/task)

🫠　「Makefile警察に捕まらなくて安心だね」

---

## Taskfile の書き方

```yaml
version: '3'

tasks:
  default:
	desc: Display available tasks
	  cmds:
	  - echo "Available tasks:"
	  - task --list
  show-cwd:
	desc: Show current working directory
	  cmds:
	  - echo "current directory: $(shell pwd)"
  delete-ds-store:
	desc: Delete all .DS_Store files
	cmds:
	  - find . -name '.DS_Store' -type f -ls -delete

```
---

## Taskfileの自己文書化

```bash
❯❯❯ task           
task: [default] task -l
task: Available tasks for this project:
* build-html:            Build HTML files
* default:               Display available tasks
* delete-ds-store:       Delete all .DS_Store files
* show-cwd:              Show current working directory
```

---

## mk - Taskfile 対応

Taskfile にも対応しています。

```bash
❯❯❯ mk -t -f https://raw.githubusercontent.com/orangekame3/LT/refs/heads/main/midousuji-tech/Taskfile.yaml
```

🫠　「mkコマンドにただparser追加しただけ...」

---

## 余談

>If you call Task with the `--global` (alias `-g`) flag, it will look for your home directory instead of your working directory. In short, Task will look for a Taskfile that matches `$HOME/{T,t}askfile.{yml,yaml}` . This is useful to have automation that you can run from anywhere in your system!

参考 : [Usage \| Task](https://taskfile.dev/usage/)

🫠　「Task公式もオレオレコマンドの管理ツールとしての利用方法を提案してる...」

---

## まとめ

- 頻繁に利用するコマンドを管理する方法を考えてみた
- `Makefile` は自己文書化が可能
- `mk` はインタラクティブにコマンドを実行できる
- `mk` は　`Taskfile`にも対応している
- 使いやすい環境を作ることで業務効率化につながる
- みんなもTaskでオレオレコマンドを管理してみよう
- 以上です。ありがとうございました。🫠
