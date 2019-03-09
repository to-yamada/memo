# Virtualbox への Arch Linux インストール

## Virtualbox 側の設定

* 64GB の可変ボリュームを作っておく。特にRAID構成などは取らない。
* EFIを有効にチェック入れる

## 起動

`archlinux-2019.03.01-x86_64.iso` を起動ディスクにして起動。

`Arch Linux archiso x86_64 UEFI USB` を選択する。
しばらく黒い画面のままなにも進捗していないように見えるが、1分程度待つとプロンプトが現れるので待つ。

## キーボードレイアウト

US 配列のままで良いので特に操作なし。

## 起動モードの確認

```
# ls /sys/firmware/efi/efivars
```

ディレクトリが存在すればよし。

## パーティション

デバイスの確認

```
# fdisk -l
```

パーティションの作成

```
# parted /dev/sda
(parted) mklabel gpt
(parted) mkpart primary fat32 1MiB 551MiB
(parted) set 1 esp on
(parted) mkpart primary ext4 551MiB 100%
(parted) quit
```


