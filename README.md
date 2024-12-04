Hardware Requirements

Minimum

4CPU 8RAM 100GB
Recommended

8CPU 16RAM 200GB
Rent On Hetzner | Rent On OVH
Dependencies Installation

**Install dependencies for building from source**
```
sudo apt update
sudo apt install -y curl git jq lz4 build-essential
```

**Install Go**
```
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.22.7.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.profile
source .profile
```

**Clone project repository**
```
cd && rm -rf core
git clone https://github.com/Team-Kujira/core
cd core
git checkout v1.1.0
```

**Build binary**
```
make install
```

**Prepare cosmovisor directories**
```
mkdir -p $HOME/.kujira/cosmovisor/genesis/bin
ln -s $HOME/.kujira/cosmovisor/genesis $HOME/.kujira/cosmovisor/current -f
```

**Copy binary to cosmovisor directory**
```
cp $(which kujirad) $HOME/.kujira/cosmovisor/genesis/bin
```

**Set node CLI configuration**
```
kujirad config chain-id kaiyo-1
kujirad config keyring-backend file
kujirad config node tcp://localhost:11857
```

# Initialize the node
kujirad init "Your Node Name" --chain-id kaiyo-1

# Download genesis and addrbook files
curl -L https://snapshots.nodejumper.io/kujira/genesis.json > $HOME/.kujira/config/genesis.json
curl -L https://snapshots.nodejumper.io/kujira/addrbook.json > $HOME/.kujira/config/addrbook.json

# Set seeds
sed -i -e 's|^seeds *=.*|seeds = "ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:11856,20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:11856,322abfd7c0bcdf8a3d98ccb46ae2572bae0e8301@seed-kujira.starsquid.io:15602,824fa337b806bd48ce9505d74ba3e5adea80da93@seeds.goldenratiostaking.net:1628,ebc272824924ea1a27ea3183dd0b9ba713494f83@kujira-mainnet-seed.autostake.com:26796,400f3d9e30b69e78a7fb891f60d76fa3c73f0ecc@kujira.rpc.kjnodes.com:11359,8542cd7e6bf9d260fef543bc49e59be5a3fa9074@seed.publicnode.com:26656,654ba97f74254965a80c0fac0f277f6f6e5506b6@seed-node.mms.team:29656,10ed1e176d874c8bb3c7c065685d2da6a4b86475@seed-kujira.ibs.team:16678,ebc272824924ea1a27ea3183dd0b9ba713494f83@kujira-mainnet-peer.autostake.com:26796"|' $HOME/.kujira/config/config.toml

# Set minimum gas price
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0034ukuji,0.01186factory/kujira1qk00h5atutpsv900x202pxx42npjr9thg58dnqpa72f2p7m2luase444a7/uusk,0.0119ibc/295548A78785A1007F232DE286149A6FF512F180AF5657780FC89C009E2C348F,0.00129ibc/27394FB092D2ECCD56123C74F36E4C1F926001CEADA9CA97EA622B25F41E5EB2,0.01795ibc/47BD209179859CDE4A2806763D7189B6E6FE13A17880FE2B42DE1E6C1E329E23,0.65943ibc/3607EB5B5E64DD1C0E12E07F077FF470D5BC4706AFCBC98FE1BA960E5AE4CE07,160416396197ibc/F3AA7EF362EC5E791FE78A0F4CCC69FEE1F9A7485EB1A8CAB3F6601C00522F10,0.02689ibc/EFF323CC632EC4F747C61BCE238A758EFDB7699C3226565F7C20DA06509D59A5,0.01495ibc/DA59C009A0B3B95E0549E6BF7B075C8239285989FF457A8EDDBB56F10B2A6986,0.03139ibc/A358D7F19237777AF6D8AD0E0F53268F8B18AE8A53ED318095C14D6D7F3B2DB5,0.90403ibc/4F393C3FCA4190C0A6756CE7F6D897D5D1BE57D6CCB80D0BC87393566A7B6602,559196837ibc/004EBF085BBED1029326D56BE8A2E67C08CECE670A94AC1947DF413EF5130EB2,5772801ibc/1B38805B1C75352B28169284F96DF56BDEBD9E8FAC005BDCC8CF0378C82AA8E7,0.01807factory/kujira1643jxg8wasy5cfcn7xm8rd742yeazcksqlg4d7/umnta,0.01194ibc/FE98AAD68F02F03565E9FA39A5E627946699B2B07115889ED812D8BA639576A9,0.00019ibc/E5CA126979E2FFB4C70C072F8094D07ECF27773B37623AD2BF7582AD0726F0F3"|' $HOME/.kujira/config/app.toml

# Set pruning
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.kujira/config/app.toml

# Enable prometheus
sed -i -e 's|^prometheus *=.*|prometheus = true|' $HOME/.kujira/config/config.toml

# Change ports
sed -i -e "s%:1317%:11817%; s%:8080%:11880%; s%:9090%:11890%; s%:9091%:11891%; s%:8545%:11845%; s%:8546%:11846%; s%:6065%:11865%" $HOME/.kujira/config/app.toml
sed -i -e "s%:26658%:11858%; s%:26657%:11857%; s%:6060%:11860%; s%:26656%:11856%; s%:26660%:11861%" $HOME/.kujira/config/config.toml

# Download latest chain data snapshot
curl "https://snapshots.nodejumper.io/kujira/kujira_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.kujira"

# Install Cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.6.0

# Create a service
sudo tee /etc/systemd/system/kujira.service > /dev/null << EOF
[Unit]
Description=Kujira node service
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.kujira
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.kujira"
Environment="DAEMON_NAME=kujirad"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable kujira.service

# Start the service and check the logs
sudo systemctl start kujira.service
sudo journalctl -u kujira.service -f --no-hostname -o cat
Secure Server Setup (Optional)

# generate ssh keys, if you don't have them already, DO IT ON YOUR LOCAL MACHINE
ssh-keygen -t rsa

# save the output, we'll use it later on instead of YOUR_PUBLIC_SSH_KEY
cat ~/.ssh/id_rsa.pub
# upgrade system packages
sudo apt update
sudo apt upgrade -y

# add new admin user
sudo adduser admin --disabled-password -q

# upload public ssh key, replace YOUR_PUBLIC_SSH_KEY with the key above
mkdir /home/admin/.ssh
echo "YOUR_PUBLIC_SSH_KEY" >> /home/admin/.ssh/authorized_keys
sudo chown admin: /home/admin/.ssh
sudo chown admin: /home/admin/.ssh/authorized_keys

echo "admin ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# disable root login, disable password authentication, use ssh keys only
sudo sed -i 's|^PermitRootLogin .*|PermitRootLogin no|' /etc/ssh/sshd_config
sudo sed -i 's|^ChallengeResponseAuthentication .*|ChallengeResponseAuthentication no|' /etc/ssh/sshd_config
sudo sed -i 's|^#PasswordAuthentication .*|PasswordAuthentication no|' /etc/ssh/sshd_config
sudo sed -i 's|^#PermitEmptyPasswords .*|PermitEmptyPasswords no|' /etc/ssh/sshd_config
sudo sed -i 's|^#PubkeyAuthentication .*|PubkeyAuthentication yes|' /etc/ssh/sshd_config

sudo systemctl restart sshd

Ư$Z^3e5sudo apt install -y fail2ban

# install and configure firewall
sudo apt install -y ufw
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow ssh
sudo ufw allow 9100
sudo ufw allow 26656

# make sure you expose ALL necessary ports, only after that enable firewall
sudo ufw enable

# make terminal colorful
sudo su - admin
source <(curl -s https://raw.githubusercontent.com/nodejumper-org/cosmos-scripts/master/utils/enable_colorful_bash.sh)

# update servername, if needed, replace YOUR_SERVERNAME with wanted server name
sudo hostnamectl set-hostname YOUR_SERVERNAME

# now you can logout (exit) and login again using ssh admin@YOUR_SERVER_IP
