# singularity/Apptainer

- Singularity https://sylabs.io/
- Apptainer https://apptainer.org/

## セットアップ

AlmaLinux 8, 9ではEPELにRPMパッケージがある。
Singularity-CEとApptainerを同時にインストールすることは
できない。

### Singularity-CE AlmaLinux 8, 9
```
dnf install epel-release
dnf update
dnf install singularity-ce
```

``dnf update``で``epel-release``パッケージがアップデート
される。

### Apptainer AlmaLinux 8, 9
```
dnf install epel-release
dnf update
dnf install singularity-ce
```

``dnf update``で``epel-release``パッケージがアップデート
される。

## クイックスタート

Dockerイメージをダウンロードして試すことができる。

```console
% apptainer build alma9-min.sif docker://almalinux/9-minimal
INFO:    Starting build...
Getting image source signatures
Copying blob 5f587440817f done  
Copying config dc8031a199 done  
Writing manifest to image destination
Storing signatures
2023/05/02 09:20:37  info unpack layer: sha256:5f587440817f7c56198a3cc9df5d654dd56b5394b2a2a83cf5e2fb635046f2f9
INFO:    Creating SIF file...
INFO:    Build complete: alma9-min
% ls
alma9-min.sif*
% file alma9-min.sif
alma9-min.sif: a /usr/bin/env run-singularity script executable (binary data)
```

alma9-min.sifファイルができている
（sif: Singularity Image Format）。

（注）ダウンロードしたファイルは``$HOME/.apptainer/``以下に保存されていて
次回以降キャッシュとして利用される。

実行:
```
% apptainer shell alma9-min.sif
```

これでプロンプトが
``Apptainer>``にかわりalma9-minコンテナに入る。
``apptainer``コマンドを実行したときのディレクトリと
ホームディレクトリがマウントされ使えるようになっている。

``apptainer``コマンドを実行するマシンに``/usr/local``があり、
apptainerイメージにも``/usr/local``がある場合

```
host% cd /usr/local
host% apptainer shell ~/path/to/alma9-min.sif
```

と実行するとhost側の``/usr/local``が見えるようになっている。

## イメージの書き換え

イメージを書きかえるにはいったん.sifファイルを展開する。

```console
% apptainer build --sandbox alma9-min alma9-min.sif
INFO:    Starting build...
INFO:    Verifying bootstrap image alma9-min.sif
WARNING: integrity: signature not found for object group 1
WARNING: Bootstrap image could not be verified, but build will continue.
INFO:    Creating sandbox directory...
INFO:    Build complete: alma9-min
% ls
alma9-min/  alma9-min.sif*
% ls alma9-min
afs/  dev/	    etc/   lib@    media/  opt/   root/  sbin@	       srv/  tmp/  var/
bin@  environment@  home/  lib64@  mnt/    proc/  run/	 singularity@  sys/  usr/
```

書き換えように実行:

```console
% apptainer shell --writable alma9-min
Apptainer> touch /A
Apptainer> ls -l /A
-rw-rw-r--. 1 you you 0 May  2 09:48 /A
```

書き換え後、.sifイメージを作る：
```console
% apptainer build --fakeroot my.sif alma9-min
INFO:    Starting build...
INFO:    Creating SIF file...
INFO:    Build complete: my.sif
```

実行してみる:
```console
% apptainer shell my.sif
Apptainer> ls /
A    bin  environment  home  lib64  mnt  proc  run   singularity  sys  usr
afs  dev  etc	       lib   media  opt  root  sbin  srv	  tmp  var
```

## 自力でイメージを作る

dockerその他からイメージをダウンロードしてそれを
もとに自分用イメージを作れるが、ときとしてダウンロード
したイメージが使えないことがある。

（例）
RedisTimeSeriesをコンパイルしようとすると
途中でcurlパッケージがあるかどうか調べる。
はいっていなければ``yum install curl``でインストールを
試みる。``docker://almalinux/9-base``はcurlパッケージ
ではなく、curl-minimalパッケージが入っていて
``yum install curl``が失敗する。がyumが
curlを使うのでcurl-minimalが入っている環境では
手動で``yum remove curl-minimal``することができない。
（例おわり）

イメージ作成にはまずdefファイルを作る。

Apptainer Definition File:
https://apptainer.org/docs/user/1.0/definition_files.html

### defファイルの例

- [almalinux9.def](almalinux9.def)
- [rocky-httpd.def](rocky-httpd.def)

``%post``セクションに自分が必要とする
パッケージをインストールするyumコマンドを書く
（``yum -y``としてインタラクションが発生しないようにする）。

### defファイルの使い方

```
sudo apptainer build --sandbox almalinux9 almalinux9.def
```

とするとカレントディレクトリにalmalinux9というディレクトリができて
AlmaLinux 9がセットされているはず。

### shellで走らせてみる

```
sudo apptainer shell --writable almalinux9
```

とするとalmalinux9の中に入れていろいろ作業できる
（dnf install など）。

作業後、sifイメージを作るには

```
sudo apptainer build almalinux9.sif almalinux9
```

## --fakerootオプション

apptainer fakerootオプション:
https://apptainer.org/docs/user/1.0/fakeroot.html

使うには``/etc/subuid``、``/etc/subgid``の整備が必要。
``useradd``などのツールを使ってユーザーを登録すると
自動的に``/etc/subuid``にも追加してくれる。
``userdel``でユーザーを消すとそのユーザーの行が
``/etc/subuid``から消える。

追加には``usermod``コマンドも使える。
例:
```
usermod --add-subuids 100000-165535 --add-subgids 100000-165535 you
```

``/etc/subuid``の例:
```
you:100000:65536
you2:165536:65536
```

youユーザーはsubuidとして100000から100000+655536個分使う
ことができる。

you2ユーザーは165536から65536個使うことができる。

これらが既存ホスト環境のUIDとかぶらないようにするのは
管理者の責任になる（ようだ）。

``/etc/subuid``が整備されていればrootにならずに

```
% apptainer build --sandbox --fakeroot alma9 almalinux9.def
```
で作った環境の/とかの所有者はroot:rootになる。

実行:
```
% apptainer shell --writable --fakeroot alma9
```
とすると``$HOME``が``/root``になっていて
そこにはapptainerコマンドを実行したユーザーのホームディレクトリが
ある。

```console
% apptainer shell --writable --fakeroot alma9
INFO:    underlay of /etc/localtime required more than 50 (177) bind mounts
Apptainer> touch ABC
Apptainer> ls -ld ABC
-rw-rw-r--. 1 root root 0 May  2 14:39 ABC
Apptainer> 
```

apptainer環境を抜けると上のABCファイルは
```
host% ls -l ~/ABC
-rw-rw-r--. 1 you you 0 May  2 14:39 /home/you/ABC
```
となっている。

```
% apptainer shell --writable --fakeroot alma9
```

でコンテナ環境にはいるとroot権限をもったのと同様に
なっているのでプログラムをコンパイルして
コンテナ環境の/usr/local/binにインストールするなどが
sudoなしにできるようになっている。

sifイメージの作成は``--fakeroot``をつけて
```
apptainer build --fakeroot alma9.sif alma9
```
とすればよい。

## Singularity/Apptainerの違い（か？）

AlmaLinux 9でdnf install singurality-ceで入る
singularity-ce 3.11.1-1.el9
とApptainer
apptainer-1.1.7-1.el9.x86_64
の違い（か？）

Apptainerで作ったAlmaLinux 9 'minimal install' + 'Development Tool"
の2GBのsifファイルをsingularityで使うと
```
singularity shell alma9.sif
```
はすぐプロンプトがでたが
```
singularity shell --fakeroot alma9.sif
```
は
```
INFO:    Converting SIF file to temporary sandbox...
```
とでてvmstatでみるとたしかに/tmp以下にファイルを展開している
ようであった。

``apptainer shell``は``--fakeroot``あるなしにかかわらずすぐに
使えるようになる。

## おまけ

最小defファイル
```
BootStrap: yum
MirrorURL: http://ftp.riken.jp/Linux/centos-stream/9-stream/BaseOS/x86_64/os/
Include: yum
```

のときの動作。
上のdefファイルではインストールするパッケージを指定していないが、
apptainerのコードをみると
```
conveyorPacker_yum.go:    include = `/etc/redhat-release coreutils
    ` + include
```
というのがある
[github上へのソース](https://github.com/apptainer/apptainer/blob/main/internal/pkg/build/sources/conveyorPacker_yum.go#L199)

デフォルトでいれるRPMパッケージは
- coreutils
- /etc/redhat-releaseがはいっているRPMパッケージ
- ``include``にyumが入っているのでyumパッケージ
をインストールするようだ。

AlmaLinux上で
```
yum install /etc/redhat-release coreutils yum \
--installroot=/home/test --releasever=/
```
とすると消費するディスク量がわかる。
coreutilsの依存物などでAlmaLinux 9では153個のRPMパッケージ
が入り、/usrで257MB消費する。
パッケージキャッシュがはいっている/varは85MB消費する。
