Bitkini Full Node Setup Guide
Welcome to the official Bitkini Full Node Setup Guide. Follow the steps below carefully to deploy your own Bitkini full node on Ubuntu Linux 24.04.




1. Install Required Dependencies
First, update your repositories and install essential dependencies.

sudo add-apt-repository universe
sudo apt update
sudo apt install -y ninja-build build-essential cmake pkgconf python3 libevent-dev libboost-dev libsqlite3-dev qtbase5-dev qttools5-dev qttools5-dev-tools libqrencode-dev libdb5.3-dev libdb5.3++-dev libssl-dev libzmq5-dev git
If you're planning to use the GUI wallet, also run:

sudo apt install -y qt6-base-dev qt6-tools-dev qt6-declarative-dev qt6-l10n-tools qt6-tools-dev-tools libgl-dev libqt6svg6-dev libqt6networkauth6-dev
2. Create a Dedicated User
Create a dedicated user and directory for Bitkini:

sudo adduser --system --group --home /var/lib/bitkini --shell /usr/sbin/nologin bitkini
3. Clone and Build Bitkini
Clone Bitkini from GitHub and build the binaries:

cd /opt
sudo git clone https://github.com/bitkini/bitkini.git
cd bitkini
sudo mkdir build && cd build
sudo cmake .. -GNinja -DCMAKE_BUILD_TYPE=Release -DBUILD_DAEMON=ON -DBUILD_CLI=ON -DBUILD_GUI=OFF -DBUILD_TESTS=OFF -DENABLE_WALLET=ON -DWITH_ZMQ=OFF
sudo ninja -j$(nproc)
Rename and move binaries:

cd bin
sudo mv bitcoin bitkini
sudo mv bitcoind bitkinid
sudo mv bitcoin-cli bitkini-cli
sudo mv bitkinid /usr/local/bin/
sudo mv bitkini-cli /usr/local/bin/
sudo chmod 0755 /usr/local/bin/bitkinid /usr/local/bin/bitkini-cli
4. Configure Your Node
Create and edit the Bitkini configuration file:

sudo mkdir -p /var/lib/bitkini
sudo chown bitkini:bitkini /var/lib/bitkini
sudo -u bitkini nano /var/lib/bitkini/bitkini.conf
Paste the configuration below, ensuring you set a secure password:

chain=main
listen=1
server=1
txindex=1
port=6969
rpcport=8332
rpcuser=bitkini
rpcpassword=YOUR_SECURE_PASSWORD
fallbackfee=0.001
5. Setup Systemd Service
Create the Bitkini systemd service:

sudo nano /etc/systemd/system/bitkinid.service
Paste the following configuration:

[Unit]
Description=Bitkini Daemon
After=network-online.target
Wants=network-online.target

[Service]
User=bitkini
Group=bitkini
ExecStart=/usr/local/bin/bitkinid -conf=/var/lib/bitkini/bitkini.conf -datadir=/var/lib/bitkini
Restart=on-failure
TimeoutStopSec=60
PrivateTmp=true
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
Enable and start the service:

sudo systemctl daemon-reload
sudo systemctl enable bitkinid
sudo systemctl start bitkinid
6. Verify Your Node
Check your node status and logs:

sudo journalctl -u bitkinid -f
Your node should start syncing immediately. Congratulations, you're running a Bitkini full node!

7. Using bitkini-cli
Now that your Bitkini node is running, you can interact with it using the command-line tool bitkini-cli. Always include your configuration file to ensure commands execute correctly.

The general command structure is:

bitkini-cli -conf=/var/lib/bitkini/bitkini.conf <CLI-COMMAND>
You can view all supported commands using:

bitkini-cli -conf=/var/lib/bitkini/bitkini.conf help
Here are some useful example commands:

List loaded wallets:
bitkini-cli -conf=/var/lib/bitkini/bitkini.conf listwallets
Get balance of all wallets:
bitkini-cli -conf=/var/lib/bitkini/bitkini.conf getbalance
Get balance of a single wallet:
bitkini-cli -conf=/var/lib/bitkini/bitkini.conf -rpcwallet=presale1 getbalance
Generate a new Bitkini address (default - kini1...):
bitkini-cli -conf=/var/lib/bitkini/bitkini.conf getnewaddress
Generate a new Legacy address (K...):
bitkini-cli -conf=/var/lib/bitkini/bitkini.conf getnewaddress "" "legacy"
8. Wallet Management and Advanced Operations
Create a New Wallet
Create a new empty wallet (no keys) with automatic loading at startup:

bitkini-cli \
  -conf=/var/lib/bitkini/bitkini.conf \
  -named createwallet \
    wallet_name="mynewwallet" \
    disable_private_keys=false \
    blank=true \
    descriptors=true \
    load_on_startup=true
Import HD Private Keys
If you've generated a master XPRV private key with bitkini-keygen, you can import it into your wallet. Example:

xprv9s21ZrQH143K2XNDe1oMFtSrKJ2nW4n5agGrBNdHtug3MUAhDZR7vhhrJ4NtRTuafiSqhpgW7Ltxt1xBxwqzB2Zv2LJUGAsWLPfA4z2M7Gs
Execute the following commands to import your key (external addresses):

DESC="wpkh(xprv.../84'/0'/0'/0/*)"
CHECKSUM=$(bitkini-cli -conf=/var/lib/bitkini/bitkini.conf getdescriptorinfo "$DESC" | jq -r .checksum)

bitkini-cli \
  -conf=/var/lib/bitkini/bitkini.conf \
  -rpcwallet=mynewwallet \
  importdescriptors "[{
    \"desc\":\"$DESC#$CHECKSUM\",
    \"timestamp\":0,
    \"active\":true,
    \"internal\":false,
    \"range\":[0,50000]
  }]"
And then for internal addresses (change addresses):

DESC="wpkh(xprv.../84'/0'/0'/1/*)"
CHECKSUM=$(bitkini-cli -conf=/var/lib/bitkini/bitkini.conf getdescriptorinfo "$DESC" | jq -r .checksum)

bitkini-cli \
  -conf=/var/lib/bitkini/bitkini.conf \
  -rpcwallet=mynewwallet \
  importdescriptors "[{
    \"desc\":\"$DESC#$CHECKSUM\",
    \"timestamp\":0,
    \"active\":true,
    \"internal\":true,
    \"range\":[0,50000]
  }]"
Replace xprv... with your own master private key. These commands import your generated addresses into the wallet, ensuring full control over your Bitkini funds.

If you encounter any issues or have questions, join our Discord community for assistance 🌴
