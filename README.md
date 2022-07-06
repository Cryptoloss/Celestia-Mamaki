# Celestia-Mamaki

![celestia](https://user-images.githubusercontent.com/98783018/177457169-b4d1d7eb-7b6e-4c59-8022-29c84c3c5078.png)

Cosmos Ekosisteminin Modüler Yapıdaki En Önemli Projesi olan Celestia Mamaki Testnet Katılım Kodları

**Sistem Gereksinimleri**

- Ram: 8 GB RAM
- CPU: Quad-Core
- Disk: 250 GB SSD 
- Bandwidth: 1 Gbps for Download/100 Mbps for Upload

_Contabo minumum paket yeter _

**Sosyal Medya hesaplarımız **

[LossNode Youtube](https://www.youtube.com/channel/UCKIpdWDFJN59nkVQHn5xfZA)
[LossNode Telegram](https://t.me/LossNode)
[Cryptoloss Twitter](https://twitter.com/Cryptoloss1)

_Güncelleme ile başlayalım_

`sudo apt update && sudo apt upgrade -y`

_Gerekli temel paketleri kuralım_

`sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential \
git make ncdu -y`

_Install Golang_

```
ver="1.18.2"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
```

_Sıradaki Kod_

`echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile`

_Version kontrolü_

`go version`

**go version komutunu girdikten sonra çıktı şu şekilde olmalı:**`go version go1.18.2 linux/amd64`

_P2P Kurulum_

```
cd $HOME
rm -rf celestia-app
git clone https://github.com/celestiaorg/celestia-app.git
cd celestia-app/
APP_VERSION=$(curl -s \
  https://api.github.com/repos/celestiaorg/celestia-app/releases/latest \
  | jq -r ".tag_name")
git checkout tags/$APP_VERSION -b $APP_VERSION
make install

celestia-appd version

```
```
cd $HOME
celestia-appd init nodeismi --chain-id mamaki

pruning="custom"
pruning_keep_recent="100"
pruning_interval="10"

sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \
\"$pruning_keep_recent\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \
\"$pruning_interval\"/" $HOME/.celestia-app/config/app.toml

```
_Yeni cüzdan oluşturma komutu (cüzdan ismi yazan yeri değiştirmeyi unutma)_

`celestia-appd keys add cüzdanismi`

_Eski Cüzdanı kullanmak için recover edebilirsiniz(memoniclere ihtiyac var)_

`celestia-appd keys add cüzdanismi --recover`

_Şimdi [Celestia Discorda](https://discord.gg/4NuY339T) Gidip Test Token Talep Edelim_

`$request walletadress`

_Test token aldıktan sonra kuruluma devam edelim_

```
wget -O $HOME/.celestia-app/config/genesis.json "https://raw.githubusercontent.com/celestiaorg/networks/master/mamaki/genesis.json"

BOOTSTRAP_PEERS=$(curl -sL https://raw.githubusercontent.com/celestiaorg/networks/master/mamaki/bootstrap-peers.txt | tr -d '\n') && echo $BOOTSTRAP_PEERS
sed -i.bak -e "s/^bootstrap-peers *=.*/bootstrap-peers = \"$BOOTSTRAP_PEERS\"/" $HOME/.celestia-app/config/config.toml

PEERS="7145da826bbf64f06aa4ad296b850fd697a211cc@176.57.189.212:26656, f7b68a491bae4b10dbab09bb3a875781a01274a5@65.108.199.79:20356, 853a9fbb633aed7b6a8c759ba99d1a7674b706a3@38.242.216.151:26656, 96995456b7fe3ab6524fc896dec76d9ba79d420c@212.125.21.178:26656, 268694eaf9446b2052b1161979bf2e09f3e45e81@173.212.254.166:26656, 28aaa8865f3e9bba69f257b08d5c28091b5b3167@176.57.150.79:26656"
  
sed -i.bak -e "s/^persistent-peers *=.*/persistent-peers = \"$PEERS\"/" $HOME/.celestia-app/config/config.toml
```
```
sed -i.bak -e "s/^timeout-commit *=.*/timeout-commit = \"25s\"/" $HOME/.celestia-app/config/config.toml
sed -i.bak -e "s/^skip-timeout-commit *=.*/skip-timeout-commit = false/" $HOME/.celestia-app/config/config.toml
sed -i.bak -e "s/^mode *=.*/mode = \"validator\"/" $HOME/.celestia-app/config/config.toml
```

_Celestia-appd systemd dosyaları oluşturalım_

```
sudo tee <<EOF >/dev/null /etc/systemd/system/celestia-appd.service
[Unit]
Description=celestia-appd Cosmos daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$HOME/go/bin/celestia-appd start
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```

`cat /etc/systemd/system/celestia-appd.service`

_Node başlat_

```
cd $HOME/.celestia-app
celestia-appd tendermint unsafe-reset-all --home "$HOME/.celestia-app"
sudo systemctl enable celestia-appd
sudo systemctl start celestia-appd
```

_Herşey yolundaysa resimdeki gibi bir çıktı alacaksınız aşşağıdaki kodu girditen sonra(geri gelmek için ctrl+c tuşuna bas)_

`sudo journalctl -u celestia-appd.service -f`

![system status](https://user-images.githubusercontent.com/98783018/177455062-8f638a0e-489f-45a9-8cf7-2edd3b63f921.png)


_Logları kontrol edelim_

`sudo journalctl -u celestia-appd.service -f`

**Burası çokamelli devam etmeden önce nodemuzun güncel block değerini yakalaması lazım yani aşşağıdaki kodu girdikten sonra 'FALSE' çıktısına ulaştıktan sonra ancak validatör kurabilirsiniz**

_False çıktısı kontrol komutu_

`curl -s localhost:26657/status | jq .result | jq .sync_info`


_Validatör kur (monikeradını değiştirmeyi unutma ben cryptoloss yaptım)_

```
celestia-appd tx staking create-validator \
    --amount=1000000utia \
    --pubkey=$(celestia-appd tendermint show-validator) \
    --moniker=MonikerAdi \
    --chain-id=mamaki \
    --commission-rate=0.1 \
    --commission-max-rate=0.2 \
    --commission-max-change-rate=0.01 \
    --min-self-delegation=9000000 \
    --from=cüzdanismi
```
Kodu girdikten sonra sizden şifre isteyecek şifreyi gireceksiniz ve sonrasında resimdeki bir çıktı almalısınız

![validatör](https://user-images.githubusercontent.com/98783018/177456129-00d8faf4-b4be-4280-a1c4-eea8ba4dec29.png)

_Delege komutu_

```
celestia-appd tx staking delegate \
    valoperadresi 1000000utia \
    --from=hangicüzdandandelegeetmekistiyorsanızonunadresi --chain-id=mamaki
```

Desteğe ihtiyacınız olursa buradayız

[LossNode Youtube](https://www.youtube.com/channel/UCKIpdWDFJN59nkVQHn5xfZA)
[LossNode Telegram](https://t.me/LossNode)
[Cryptoloss Twitter](https://twitter.com/Cryptoloss1)


Kodları [NakoTurk](https://twitter.com/NakoTurk) Hocam Sayesinde Toparladım Kendisine Çok Teşekkür Ederim
