# Proxy 環境での CoreOS インストール

Container linux
===============

インストーラの作成
---------

https://coreos.com/os/docs/latest/booting-with-iso.html

から stable 版の最新 iso ファイルを取得する。

### 物理PCへインストールする場合

取得した iso ファイルからブート用USBを作成する。

### virtualbox などの仮想マシンへインストールする場合

isoファイルやインストールしたいマシン上でisoファイルからインストーラを実行する。


インストール
----

作成したインストーラから Container Linux を起動。
この時点ではまだインストールは行われておらず、RAM上で動作している状態。

### ネットワーク設定

RAM 上で動作している Container Linux のネットワーク設定を行う。
(Container Linux はインストール時に www から最新版のイメージをダウンロードするためオフライン状態ではインストール自体ができない)。

/etc/systemd/network/static.network ファイルを生成し、以下の内容を記述する。
`enp0sXXXX` の部分は使用するPCによって変わる。インストールする機器上で `ip a` などを実行すればわかる。

```
[Match]
Name=enp0sXXXX

[Network]
DHCP=none
Address=XXX.XXX.XXX.XXX/XX
Gateway=XXX.XXX.XXX.XXX
DNS=XXX.XXX.XXX.XXX
```

記述した状態で以下のコマンドを実行し適用する。

```sh
sudo systemctl restart systemd-networkd
```

以下のコマンドでプロキシも設定する。

```sh
export http_proxy=http://XXX.XXX.XXX.XXX:XXXX
export https_proxy=http://XXX.XXX.XXX.XXX:XXXX
```

### cloud-config.yml の作成

OS の設定を記述した cloud-config.yml を作成する。

```yml
# This config is meant to be consumed by the config transpiler, which will
# generate the corresponding Ignition config. Do not pass this config directly
# to instances of Container Linux.

# 自動更新の停止
locksmith:
  reboot_strategy: "off"

systemd:
  units:
    # update-engine の proxy 設定
    - name: update-engine.service
      dropins:
        - name: 50-proxy.conf
          contents: |
            [Service]
            Environment="ALL_PROXY=http://XXX.XXX.XXX.XXX:XXXX"
    # docker の proxy 設定
    - name: docker.service
      enable: true
      dropins:
        - name: 20-http-proxy.conf
          contents: |
            [Service]
            Environment="HTTP_PROXY=http://XXX.XXX.XXX.XXX:XXXX"

networkd:
  units:
    - name: 00-eth0.network
      contents: |
        [Match]
        Name=enp0sXXXX

        [Network]
        DHCP=none
        Address=XXX.XXX.XXX.XXX/XX
        Gateway=XXX.XXX.XXX.XXX
        DNS=XXX.XXX.XXX.XXX

passwd:
  users:
    - name: core
      groups: [ sudo, docker ]
      ssh_authorized_keys:
        - ssh-rsa AAAABBBBCCCC.......
```

### ignition.json の生成

作成した cloud-config.yml から ignition.json を生成する。

変換ツールは以下を参照。

https://github.com/coreos/container-linux-config-transpiler/releases

```
./ct-v0.9.0-x86_64-unknown-linux-gnu -pretty -strict -in-file /mnt/cloud-config.yaml > ~/ignition.json
```

### インストール実行

coreos-install で ignition.json を指定してインストールする。

container linux上で

```
sudo -E coreos-install -d /dev/sda -C stable -i ~/ignition.json
```

とするとインストールが実行される。

sudo の -E オプションは上で http_proxy を設定しており、これを root からも参照するために必要。
coreos-install の -d オプションはインストール先を指定する。
coreos-install の -C オプションは Container Linux の Channel を指定する。
stable, beta, alpha が指定できる。

### docker-compose のインストール

参考元: https://qiita.com/Ahijo0523/items/1d4055de7a74decd37fb

```
COMPOSE_VERSION=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep 'tag_name' | cut -d\" -f4)
OUTPUT_PATH="/opt/bin"
sudo mkdir -p ${OUTPUT_PATH}
sudo curl -L "https://github.com/docker/compose/releases/download/${COMPOSE_VERSION}/docker-compose-$(uname -s)-$(uname -m)" -o ${OUTPUT_PATH}/docker-compose
sudo chmod +x ${OUTPUT_PATH}/docker-compose
docker-compose -v
```
