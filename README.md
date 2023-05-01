# singularity defファイル

以下、コマンドはapptainerを使う場合には
singularityのかわりにapptainerを使う。

（注）AlmaLinux 9ではsingularity-ceパッケージと
apptainerパッケージは同時にはセットすることはできない。

## 走らせ方

```
sudo singularity build --sandbox almalinux9 almalinux9.def
```

とするとカレントディレクトリにalmalinux9というディレクトリができて
AlmaLinux 9がセットされているはず。

## shellで走らせてみる

```
sudo singularity shell --writable almalinux9
```

とするとalmalinux9の中に入れていろいろ作業できる
（dnf install など）。

作業後、sifイメージを作るには

```
sudo singularity build almalinux9.sif almalinux9
```
