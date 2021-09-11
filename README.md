![markdown logo](https://github.com/rognar9nok/relayer-transfer-ibc-protocol/blob/https/github.com/rognar9nok/testnets-1/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%20(257).png)

# Relayer IBC transfer guide 
 In order to perform an IBC transfer, we need to have 2 or more chains. If relayer installed separately from clients, for this needs to be changed in the client config file laddr = "tcp: //0.0.0.0: 26657"

In this example, the relayer will be installed on a server with Umee chain.

## First you need to check if the chains support the IBC protocol.
 `umeed q ibc-transfer params` 

If the answer is the same, then everything is fine, we continue.

*receive_enabled: true
 send_enabled: true*

Checking the current version
 `git version`
output
 *git version 2.25.1*

## Install Golang (go)
1) Remove any existing installation of go

   `sudo rm -rf /usr/local/go`

2) Install latest/required Go version (installing go1.16.5)

   `curl https://dl.google.com/go/go1.16.5.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf -`

3) Update env variables to include go

```
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF

source $HOME/.profile
```
4) Check the version of go installed

  `go version`

## Installing and configuring the Relayer:
``` 
   git clone https://github.com/cosmos/relayer.git
   cd relayer
   git checkout v0.9.3
   make install
   rly config init
 ```
 Check the current version

` 
 rly version
`
*version: 0.9.3
commit: 4b81fa59055e3e94520bdfae1debe2fe0b747dc1
cosmos-sdk: v0.42.4
go: go1.16.5 linux/amd64*

 After installing the relayer, go to and edit config.yaml

 By default, the config should be located at this path (you can use any other editor)

   `nano /root/.relayer/config/config.yaml`

 Add the following to config.yaml

```
global:
  api-listen-addr: :5183
  timeout: 30s
  light-cache-size: 20
chains:
- key:
  chain-id: umee-betanet-1
  rpc-addr: http://0.0.0.0:26657
  account-prefix: umee
  gas-adjustment: 1.5
  gas-prices: 0.025uumee
  trusting-period: 48h
- key:
  chain-id: kichain-t-4
  rpc-addr: http://ip-address:26657
  account-prefix: tki
  gas-adjustment: 1.5
  gas-prices: 0.025utki
  trusting-period: 48h
paths: {}
```
 Import the keys for the relay to use them when signing and relaying transactions
```
  rly keys restore umee-betanet-1 umee-wallet "your mnemonic"
  rly keys restore kichain-t-4 kichain-wallet "your mnemonic"
```

 Add keys to the chains config
```
  rly chains edit umee-betanet-1 key umee-wallet
  rly chains edit kichain-t-4 key kichain-wallet
```
  Check the list of chain keys

```
  rly keys list umee-betanet-1
  rly keys list kichain-t-4
```
 Check balance
```
  rly q bal umee-betanet-1
  rly q bal kichain-t-4
```
  Initiate the light client chains

  `rly light init umee-betanet-1 -f` *successfully created light client for umee-betanet-1 by trusting endpoint http://0.0.0.0:26657...*

  `rly light init kichain-t-4 -f`   *successfully created light client for kichain-t-4 by trusting endpoint http://ip-address:26657...*

  Create a path named ibc-1 (you can change ibc-to your own)

  `rly paths generate umee-betanet-1 kichain-t-4 ibc-1 --port=transfer`

 Check paths

  `rly paths show ibc-1 --yaml`

  `rly chains list` 

  If the answer is identical, then continue on: *0: umee-betanet-1       -> key(✔) bal(✔) light(✔) path(✔)
 1: kichain-t-4          -> key(✔) bal(✔) light(✔) path(✔)*

### The next step is to create id, connection, channel.

 This can be done manually or automatically with the command below:

  `rly tx link ibc-1`

  `rly paths list -d` *path check ibc-1  -> chns(✔) clnts(✔) conn(✔) chan(✔) (kichain-t-4:transfer<>umee-betanet-1:transfer)*

  We made the connection between Kichain and Umee. It remains to execute the ibc transaction from one chain to another

  `rly tx transfer kichain-t-4 umee-betanet-1 1000000utki umee13.... --path ibc-1` *[kichain-t-4]@{xxx} - msg(0:transfer) hash(xxx)*

  If everything went well, then you will receive a hash with the completed transaction, congratulations!
