# Virtualbox への Arch Linux インストール

以下を参考に進める。

https://wiki.archlinux.org/index.php/Installation_guide

## インストールの準備

### Virtualbox 側の設定

* 64GB の可変ボリュームを作っておく。特にRAID構成などは取らない。
* EFIを有効にチェック入れる

### 起動

`archlinux-2019.03.01-x86_64.iso` を起動ディスクにして起動。

`Arch Linux archiso x86_64 UEFI USB` を選択する。
しばらく黒い画面のままなにも進捗していないように見えるが、1分半程度待つとプロンプトが現れるので待つ。

### キーボードレイアウト

US 配列のままで良いので特に操作なし。

### インターネットへの接続

Virtualbox の NAT 経由であるため特に設定なし。

Proxy 配下の場合はここで Proxy 設定。

```
# export http_proxy=http://xxx.xxx.xxx.xxx:xxxx
# export https_proxy=http://xxx.xxx.xxx.xxx:xxxx
```

### システムクロックの更新

Virtualbox 環境下であるため設定なし。

### パーティションの作成

```
# parted /dev/sda
(parted) mklabel gpt
(parted) mkpart primary fat32 1MiB 551MiB
(parted) set 1 esp on
(parted) mkpart primary ext4 551MiB 100%
(parted) print
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

## インストール

### ミラーの選択

`/etc/pacman.d/mirrorlist` を編集し、jaist.ac.jp, tsukuba.wide.ad.jp のものをファイル先頭側に移動させる。

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


### ロケール

`/etc/locale.gen` を編集し、`en_US.UTF-8 UTF-8` と `ja_JP.UTF-8 UTF-8` をアンコメントする。

生成
```
# locale-gen
```

### ネットワーク設定

hostname 設定

```
# echo arch > /etc/hostname
```

`/etc/hosts` を編集する。内容は以下。
```
127.0.0.1 localhost
::1       localhost
127.0.1.1 arch.localdomain arch
```

Virtualbox 環境下であるためその他の設定は必要なし。

### Initramfs

設定の必要なし

### Root パスワード

```
# passwd
```

### ブートローダー

#### systemd-boot のインストール

```
# bootctl --path=/boot install
# bootctl update
```

#### bootctl update の自動化

```
# mkdir /etc/pacman.d/hooks
```

`/etc/pacman.d/hooks/systemd-boot.hook` を編集する。内容は以下。
```
[Trigger]
Type = Package
Operation = Upgrade
Target = systemd

[Action]
Description = Updating systemd-boot
When = PostTransaction
Exec = /usr/bin/bootctl update
```

#### loaderの設定
`/boot/loader/loader.conf` を編集する。内容は以下。
```
default arch
timeout 2
editor  no
```

#### マイクロコード取得

Virtualbox 環境下であるため不要。

#### ローダーの追加

```
# blkid -s PARTUUID -o value /dev/sda2 > /boot/loader/entries/arch.conf
```
とし、PARTUUIDを書き込む。

PARTUUIDが書き込まれた `/boot/loader/entries/arch.conf` を編集する。内容は以下。

```
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options root=PARTUUID=**** rw
```

### 再起動

```
# exit
# umount -R /mnt
# shutdown -h now
```

シャットダウン後、ストレージにある iso を除去しておく。

## インストール後の設定

### Swap ファイルの作成

今のところ作成しない。

### ネットワークの有効化

`/etc/systemd/network/MyDhcp.network` を編集する。内容は以下。

```
[Match]
Name=en*

[Network]
DHCP=ipv4
```

以下のコマンドを実行。

```
# systemctl start systemd-networkd
# systemctl enable systemd-networkd
# systemctl start systemd-resolved
# systemctl enable systemd-resolved
# ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

### 一般ユーザーの作成

archie は使用するユーザー名に置き換え

```
# useradd -m -G wheel archie
# passwd archie
```
