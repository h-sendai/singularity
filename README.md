# singularity defファイル

## 走らせ方

```
sudo singularity build --sandbox almalinux9 almalinux9.def
```

とするとカレントディレクトリにalmalinux9というディレクトリができる。

## shellで走らせてみる

```
sudo singularity shell --sandbox --writable almalinux9
```
