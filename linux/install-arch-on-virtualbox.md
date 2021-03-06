# Virtualbox への Arch Linux インストール

以下を参考に進める。

https://wiki.archlinux.org/index.php/Installation_guide

## インストールの準備

### Virtualbox 側の設定

* 64GB の可変ボリュームを作っておく。特にRAID構成などは取らない。
* EFIを有効にチェック入れる
* ポートフォワーディングでsshを受けられるようにしておく

### 起動

`archlinux-2020.11.01-x86_64.iso` を起動ディスクにして起動。

### ssh
ssh 経由でインストールするため以下を実行

```
# systemctl start sshd
# passwd
```

以下はssh経由でログインして実行。

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

### ベースシステム, パッケージのインストール

```
# pacstrap /mnt base linux base-devel openssh vim zsh
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
# ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
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

### proxy 設定

以下の内容の `/etc/profile.d/proxy.sh` を作成する。

```
#!/bin/sh

export http_proxy=http://xxx.xxx.xxx.xxx:xxxx
export https_proxy=http://xxx.xxx.xxx.xxx:xxxx
export ftp_proxy=http://xxx.xxx.xxx.xxx:xxxx
```

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
title Arch Linux
linux /vmlinuz-linux
initrd /initramfs-linux.img
options root=PARTUUID=**** rw
```

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

sytemctl start などは再起動後に実行する。

### 一般ユーザーの作成

archie は使用するユーザー名に置き換え

```
# useradd -m -G wheel -s /bin/zsh archie
# passwd archie
```

以下を実行し、wheelグループでsudoする権限を付加。

```
# EDITOR=vim visudo
```

* %wheel の行をアンコメント。NOPASSWDとするかはお好みで
* `Defaults env_keep += "LANG LANGUAGE LINGUAS LC_* _XKB_CHARSET"`の行をアンコメント
* 以下の行を追加。 `Defaults env_keep += "http_proxy https_proxy ftp_proxy"`

以下を実行し、rootログインは無効化しておく

```
# passwd -l root
```

### sshd 設定

`/etc/ssh/sshd_config` を編集する。

```
AllowUsers    archie
```
を追加。 `archie` は使用するユーザー名に置き換え。

また `PermitRootLogin` の設定値を `prohibit-password` を `no` に書き換える。

開始・有効化は再起動後に実行する。

### pacman 色付け

`/etc/pacman.conf` を編集し、 `Color` の行をアンコメントする。


### 再起動

```
# exit
# umount -R /mnt
# shutdown -h now
```

シャットダウン後、ストレージにある iso を除去しておく。
ネットワーク、sshdを有効化していないため、VirtualBoxからは
(ヘッドレスやでタッチモードではなく)通常起動で起動する。
起動後一般ユーザーでログインし、以下を実行。


ネットワーク有効化

```
# sudo systemctl start systemd-networkd
# sudo systemctl enable systemd-networkd
# sudo systemctl start systemd-resolved
# sudo systemctl enable systemd-resolved
# sudo ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

sshd有効化

```
# sudo systemctl start sshd.service
# sudo systemctl enable sshd.service
```


以降は一般ユーザーから ssh して実行する。

### パッケージの追加

```
% sudo pacman -Syu
% sudo pacman -S git ctags ripgrep go subversion perl-term-readkey
```

### dotfiles環境作成

```
% git clone https://github.com/to-yamada/dotfiles.git
% cd dotfiles
% sh link.sh
% go get github.com/vim-volt/volt
% volt get -l
```
