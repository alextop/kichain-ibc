# IBC transactions for kichain (cross - blockchian relay)

This article describes the relay connection setup between kichain and crypto.com test networks and sending the tokens cross-chain.

Standard setup (alternatively you can download the binary):

> git clone https://github.com/cosmos/relayer.git

> cd relayer

> make install

> cd

> rly version

output:

version: 1.0.0-rc1–152-g112205b

 
 ## Initializing the relay

> rly config init

## create settings files for both networks:

> nano kichain-t-4.json

#### copy the content and place it in a file
#for local kichain node.

{
  "chain-id": "kichain-t-4",
  "rpc-addr": "http://127.0.0.1:26657",
  "account-prefix": "tki",
  "gas-adjustment": 1.5,
  "gas-prices": "0.025utki",
  "trusting-period": "48h"
}
 
#### 2nd config for local crypto.com node (running on por 26357)

> nano cry.json

{
  "chain-id": "testnet-croeseid-4",
  "rpc-addr": "http://localhost:26357",
  "account-prefix": "tcro",
  "gas-adjustment": 1.5,
  "gas-prices": "0.1basetcro",
  "trusting-period": "48h"
}

#### Adding to the relay config:

> rly chains add -f groot-011.json
> 
> rly chains add -f kichain-t-4.json
> 

#### create new wallets (alternatively, you can restore existing wallets using seed phrase):
> rly keys add testnet-croeseid-4 cro-wallet
>
>rly keys add kichain-t-4 ki-wallet
 
#### Add the newly created keys to the config of the relay:
> rly chains edit testnet-croeseid-4 key cro-wallet
>
> rly chains edit kichain-t-4 key ki-wallet

#### change the timeout of waiting for confirmation:
> nano ~/.relayer/config/config.yaml

> timeout: 30s
 
#### Fund wallets using faucet and check balances after some time:
> rly q balance testnet-croeseid-4
>
> rly q balance kichain-t-4

#### If you see tokens, initialize the light client in both networks with the command:

> rly light init testnet-croeseid-4 -f
>
> rly light init kichain-t-4 -f

#### Try to generate a channel between the networks with the command:

> rly paths generate testnet-croeseid-4 kichain-t-4 transfer  -- port=transfer

#The command does not work with cro, so

 #### Trying to open a channel for relaying:

> rly tx link transfer  --debug

#### if the channel does not open, then go to the config with the command:

> nano /root/.relayer/config/config.yaml

#### erase the lines in the path section on both networks:

> client-id: XXXX

> connection-id: XXXX
 
> channel-id: XXX

#### re-init the light client:

> rly light init testnet-croeseid-4 -f
>
> rly light init kichain-t-4 -f

#### run the command to open the channel again
 
> rly tx link transfer  -d

#### wait until you see the output of the command similar to the following:

> I[2021-09-13|01:46:13.679] ★ Channel created: [kichain-t-4]chan{channel-325}port{transfer} -> [testnet-croeseid-4]chan{channel-120}port{transfer}

#### now call the command to check the connection

> rly paths list -d

you should see:

> 0: transfer -> chns(✔) clnts(✔) conn(✔) chan(✔) (testnet-croeseid-4:transfer<>kichain-t-4:transfer)

now you are ready to transfer and relay packets.

## run relay as a service or just in the background (temporary):

> rly start transfer >> rly.log 2>&1 &

### let's try to perform an cross-network transaction (send tcro tokens):

> rly tx xfer testnet-croeseid-4 kichain-t-4 10000000basetcro tki1c4n23f3mnl9ju7ehsd6c7q82raznhqw04zq4um --path transfer -d

#### and the output should show the successful transaction and its hash, which can be checked in both crypto.com and kichain network explorers
  
> I[2021-09-13|01:48:29.937] ✔ [testnet-croeseid-4]@{401815} - msg(0:transfer) hash(A59ADEFCBBECC8DE09C222AB7393001084AA1CACA0C6D26427FE01C93697E018)

Urls as an example:

> https://crypto.org/explorer/croeseid4/tx/A59ADEFCBBECC8DE09C222AB7393001084AA1CACA0C6D26427FE01C93697E018

> https://ki.thecodes.dev/tx/6AB238CD7BF8F8CDDDB9703F137B8DBF34945B6EED2B0DC3E331C3210DCEF54E

### then try to send some tki tokens back, the results are:

> rly tx xfer kichain-t-4 testnet-croeseid-4 10000utki tcro18dld6d6zsez585c5grs0hy6af9rmwyhuufugjn --path transfer -d

> https://ki.thecodes.dev/tx/8D6618A9E80A49D104AFD59E0BA759AECB09183D3457057ED32D1B30B4407898

> https://crypto.org/explorer/croeseid4/tx/67C295E86759483424BC90E8D41810E9F5C95741AB489AA70162FFF3C41626A9
>

#### Both balances are visible in both networks! :)

> rly q balance  kichain-t-4
> 20000000ibc/238A9702992435B50A000AE039E8A7C39464E6B3084CC4853245FA7DFD417241,884980utki

> rly q balance  testnet-croeseid-4
> 934853485basetcro,10000transfer/channel-120/utki

### That's all in short.
