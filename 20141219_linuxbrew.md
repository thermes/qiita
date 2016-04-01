What's Linuxbrew
================

"Linuxbrew is a fork of Homebrew, the Mac OS package manager, for Linux." by [Linuxbrew](https://github.com/Homebrew/linuxbrew) というわけで OS X の Homebrew を Linux 環境でも使えるようにしたものです。

Why Linuxbrew
=============

Debian や RHEL系(CentOS など) は安定性を重視していることにより公式の package が古くてアワワってなりませんか？
その度に公式の開発版を持ってきたり、公式ではないリポジトリ追加したり、非公式 package を探したりして、そして package の依存関係がゴチャっとなってディストリビューションの upgrade できなくなったりと弊害が…。

レンタルサーバーだと ssh はできるけどそもそも root権限もらえないなんてことも…。

そんなとき Linuxbrew のような

1. root権限が必要ない
1. home dir にインストールできる
1. ディストリビューションの package manager に依存しない

という package manager を使うと超便利。
いざとなったら Linuxbrew をインストールした dir を丸ごと消してしまっても問題なし。

前準備
======

いわゆる build essential (GCCと周辺ツール) と Git と Ruby が必須になります。
root権限がない場合でも下記package はなんとか頑張ってインストールしてください。

Debian系
------

```
$ sudo apt-get install build-essential curl git m4 ruby texinfo libbz2-dev libcurl4-openssl-dev libexpat-dev libncurses-dev zlib1g-dev
```

公式ページには上記のみ記載だけど、下記 package も入れておいた方が無難。でないと git が入らない…。

```
$ sudo apt-get install gettext
```

RHEL系
------

```
$ sudo yum groupinstall 'Development Tools' && sudo yum install curl git m4 ruby texinfo bzip2-devel curl-devel expat-devel ncurses-devel zlib-devel
```

公式ページには上記のみ記載だけど、下記 package も入れておいた方が無難。

```
$ sudo yum install openssl-devel
```

major version 6 系の場合は↑でインストールした gcc が使えないようなので、[Linuxbrew on CentOS 6 で system の gcc を使用する方法](http://qiita.com/thermes/items/1ecb0968ab937ff59164) の処理もしておくと吉。

インストール
============

```
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/linuxbrew/go/install)"
```

のあとに

```~/.bash_profile
export PATH="$HOME/.linuxbrew/bin:$PATH"
export MANPATH="$HOME/.linuxbrew/share/man:$MANPATH"
export INFOPATH="$HOME/.linuxbrew/share/info:$INFOPATH"
```

さらに

```~/.bash_profile
export LD_LIBRARY_PATH="$HOME/.linuxbrew/lib:$LD_LIBRARY_PATH"
```

も記載した方が良いかも。

最後に

```
$ brew doctor
```

でエラーが出なければインストール完了。
環境によっては git や pkg-config についての Warning が出るけど、この辺は Linuxbrew でインストールすることになるので問題なし。

使い方
======

基本的には Homebrew と同じです。

brew install
------------

```
$ brew install git
ずらずら〜
$ which git
~/.linuxbrew/bin/git
$ git --version
git version 2.2.0
```

ただし、Homebrew と全く同じ Formula を使っている物が大半なので、意味がなかったり build できなくなったりする option もあります。

普段よく使う tool は [brew_install.sh](https://github.com/thermes/dotfiles/blob/master/brew_install.sh) という感じで管理するようにしています。

brew upgrade
------------

Formula などの update はこんな感じ。

```
$ brew update
```

古くなっている Formula の確認。

```
$ brew outdated
git (2.1.2, 2.1.3 < 2.2.0)
```

古くなっている Formula を一括 upgrade をする。

```
$ brew upgrade
```

古くなった Formula を一括削除したいよ。

```
$ brew cleanup
```

zsh 使いたいよ
==============

```
$ brew install zsh --disable-etcdir
ずらずら〜
$ which zsh
~/.linuxbrew/bin/zsh
```

そして `which zsh | sudo tee -a /etc/shells` してから `chsh` すれば良い…のだけど、なぜか CentOS6 だと下記の通り問題発生。

```
$ chsh -s `which zsh`
vagrant のシェルを変更します。
パスワード:
chsh: "/home/vagrant/.linuxbrew/bin/zsh" は存在しません。
```

というわけで、Linuxbrew でインストールした zsh を擬似的に login shell として使う方法。
この方法は root権限がない環境で /etc/shells へ追記できなくて chsh ができない場合でも有効。

```~/.bash_profile
if [ "$PS1" ]; then
    if [ -x "$HOME/.linuxbrew/bin/zsh" ]; then
        export SHELL=$HOME/.linuxbrew/bin/zsh
        exec $SHELL -l
    fi
fi
```

その他
======

Linuxbrew は tar ball の cache として $HOME/.cache/Homebrew/ を使います。ここは brew cleanup では削除してくれないので、適宜手動で削除すると良い予感。

See Also
========

* [Homebrew FAQ](https://github.com/Homebrew/homebrew/blob/master/share/doc/homebrew/FAQ.md)
* [Formula Cookbook](https://github.com/Homebrew/homebrew/blob/master/share/doc/homebrew/Formula-Cookbook.md)
