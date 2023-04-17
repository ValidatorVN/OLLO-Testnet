# OLLO-Testnet

Cộng đồng chạy Node & Validator VietNam, nơi thảo luận và chia sẻ kinh nghiệm về chạy node/validator, không bàn luận chính trị. Chanel: https://t.me/RunnodeVietNamese Youtube: https://www.youtube.com/@nodevalidatorvietnam


Cấu hình đề xuất

    4CPU
    4GB RAM
    300GB of disk space (SSD)
    
1/ Cập nhật phiên bản mới:

    sudo apt update && sudo apt upgrade -y
    
2/ Bổ sung package:

    sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git make ncdu -y
    
3/ Cài đặt Golang:

    ver="1.19.1" 
    cd $HOME 
    wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" 
    
    
    sudo rm -rf /usr/local/go 
    sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" 
    rm "go$ver.linux-amd64.tar.gz"

Chuyển thư mục Golang:

    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
    source $HOME/.bash_profile

Kiểm tra phiên bản:

    go version
    
4/ Cài đặt node:

    git clone https://github.com/OllO-Station/ollo.git
    cd ollo
    git checkout v0.0.1
    make install
    
    
Thêm thông tin node, tên node:

    ollod config keyring-backend test
    ollod config chain-id ollo-testnet-1
    ollod init NodeVietNam --chain-id ollo-testnet-1
    SEEDS=""
    PEERS=""
    sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.ollo/config/config.toml

    sed -i 's|^pruning *=.*|pruning = "custom"|g' $HOME/.ollo/config/app.toml
    sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $HOME/.ollo/config/app.toml
    sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $HOME/.ollo/config/app.toml
    sed -i 's|^snapshot-interval *=.*|snapshot-interval = 0|g' $HOME/.ollo/config/app.toml

    sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001utollo"|g' $HOME/.ollo/config/app.toml
    sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.ollo/config/config.toml
    
Tạo hệ thống:

    sudo tee /etc/systemd/system/ollod.service > /dev/null << EOF
    [Unit]
    Description=Ollo Node
    After=network-online.target
    [Service]
    User=$USER
    ExecStart=$(which ollod) start
    Restart=on-failure
    RestartSec=10
    LimitNOFILE=10000
    [Install]
    WantedBy=multi-user.target
    EOF

    ollod tendermint unsafe-reset-all --home $HOME/.ollo --keep-addr-book
    
 Chạy hệ thống & logs:
 
    sudo systemctl daemon-reload
    sudo systemctl enable ollod
    sudo systemctl start ollod

    sudo journalctl -u ollod -f --no-hostname -o cat
    
    
Tạo ví OLLO:  

    ollod keys add wallet
    
    
 Tạo validator:
 
    ollod tx staking create-validator \
    --amount=40000000utollo \
    --pubkey=$(ollod tendermint show-validator) \
    --moniker=Node&ValidatorVietnam \
    --identity=6CB6AC3E672AAB9D \
    --details=https://twitter.com/NodeValidatorVN \
    --chain-id=ollo-testnet-1 \
    --commission-rate=0.1 \
    --commission-max-rate=0.2 \
    --commission-max-change-rate=0.05 \
    --min-self-delegation=1 \
    --fees=2000utollo \
    --from=wallet \
    -y
