バージョン切り替えができるよう Ubuntu でも pyenv でインストールを実行しようとしたところ、以下のエラーが発生。

```bash
$ pyenv install --list
pyenv: no such command `install'
```

clone し直したりしても一向に直らず苦戦。
時間をかけて解決したので共有します。

# 環境

環境は VM 上の Ubuntu です。

- VMware：VMware Fusion 11 Pro
- Ubuntu：Ubuntu 18.04 LTS（日本語 Remix）

# 実行内容

## 1. pyenv をホームディレクトリに .pyenv として clone

まずは pyenv を GitHub から clone。

```zsh
$ git clone https://github.com/pyenv/pyenv ./.pyenv
```

## 2. pyenv の環境変数を設定

続いて pyenv の環境変数を設定。
自分は zsh を使用しています。
コピペする際はご自身の環境をご確認ください。

```zsh
$ cat << EOF >> ${HOME}/.zshenv
export PYENV_ROOT=\${HOME}/.pyenv
export PATH=\${PYENV_ROOT}/bin:\${PATH}
eval "\$(pyenv init -)"
EOF
$ source ${HOME}/.zshenv
```

## 3. Python インストール用のパッケージをインストール

Python のインストールに必要なパッケージをあらかじめインストールしておきます。

```zsh
$ sudo apt-get install -y zlib1g-dev libbz2-dev libreadline6 libreadline6-dev
 libsqlite3-dev libssl-dev
```

## 4. インストールできるものを確認

最新版の anaconda をインストールするためにリストを表示しようとしました。
すると…

```zsh
$ pyenv install --list
pyenv: no such command `install'
```

似たような状態の記事を発見したので、こちらを参考にしてみました。
[pyenv で install が見つからなかった
](https://qiita.com/4geru/items/b645a6f2d9a8cb0e94d2)
しかし一向に状況は変わらず…

# 解決方法

別な記事を発見。
コメントからプラグインの存在を知りました。
[pyenv で no such command `install'というエラー](https://teratail.com/questions/116260)

そこで、プラグインのインストールシェルを回してみることに。
（一般ユーザーで行ったところ、スーパーユーザー権限が必要な部分がありました。）

```zsh
$ sudo ./.pyenv/plugins/python-build/install.sh
```

これを行ったところ、問題が無事解決。
`install`コマンドが利用できるようになりました。

# 参考

- [【pyenv】Ubuntu16.04 に Python 環境を構築する【2018 年】](https://qiita.com/hashibiroko/items/60a685e50aebbde7f84e)
- [pyenv で install が見つからなかった
  ](https://qiita.com/4geru/items/b645a6f2d9a8cb0e94d2)
- [pyenv で no such command `install'というエラー](https://teratail.com/questions/116260)
