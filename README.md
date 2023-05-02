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

alma9-min.sifファイルができている。

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
