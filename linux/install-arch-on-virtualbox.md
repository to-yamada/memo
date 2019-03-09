# Virtualbox への Arch Linux インストール

以下を参考に進める。

https://wiki.archlinux.jp/index.php/インストールガイド

## インストールの準備

### Virtualbox 側の設定

* 64GB の可変ボリュームを作っておく。特にRAID構成などは取らない。
* EFIを有効にチェック入れる

### 起動

`archlinux-2019.03.01-x86_64.iso` を起動ディスクにして起動。

`Arch Linux archiso x86_64 UEFI USB` を選択する。
しばらく黒い画面のままなにも進捗していないように見えるが、1分程度待つとプロンプトが現れるので待つ。

### キーボードレイアウト

US 配列のままで良いので特に操作なし。

### 起動モードの確認

```
# ls /sys/firmware/efi/efivars
```

ディレクトリが存在すればよし。

### パーティション

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

### パーティションのフォーマット

```
# mkfs.fat -F32 /dev/sda1
# mkfs.ext4 /dev/sda2
```

### パーティションのマウント

```
# mount /dev/sda2 /mnt
# mkdir /mnt/boot
# mount /dev/sda1 /mnt/boot
```

### インターネットへの接続

Virtualbox の NAT 経由であるため特に設定なし。

Proxy 配下の場合はここで Proxy 設定。

```
# export http_proxy=http://xxx.xxx.xxx.xxx:xxxx
# export https_proxy=http://xxx.xxx.xxx.xxx:xxxx
```

接続確認

```
# ping archlinux.jp
```


### システムクロックの更新

Virtualbox 環境下であるため設定なし。

## インストール

### ミラーの選択

jaist.ac.jp, tsukuba.wide.ad.jp のものをファイル先頭側に移動させる。

```
# vim /etc/pacman.d/mirrorlist
```

### ベースシステムのインストール

```
# pacstrap /mnt base
```

## システムの設定

### fstab の生成

```
# genfstab -U /mnt >> /mnt/etc/fstab
```

### chroot

```
# arch-chroot /mnt
```

### タイムゾーン

```
# ln -sf /usr/share/zoninfo/Asia/Tokyo /etc/localtime
```

Virtualbox 環境下であるため、hwclockは実行しない。

