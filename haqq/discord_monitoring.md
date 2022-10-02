# Half-Life monitoring

## Install Go
```
wget https://go.dev/dl/go1.19.1.linux-amd64.tar.gz
```
```
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.19.1.linux-amd64.tar.gz
```
Source go
```
cat <<EOF >> ~/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
```
```
source ~/.profile
```
Check version
```
go version
```


## Install half-life
```
cd $HOME && git clone https://github.com/ptts55/half-life && cd half-life && go install
```
```
cd $HOME && sudo cp go/bin/halflife /usr/local/bin
```




## Create a webhook for a discord channel and copy Webhook URL 

[How to create webhooks](https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks)


 Once you've created the webhook, copy the URL. It'll look something like this: 
`https://discord.com/api/webhooks/1026186331828785203/MVcnh2JZwu4frkbR5t-fhovjyFqEfbPxsbLCB8AkHUbBi09AonFwIuJB5eULg5as9p88`

`DISCORD_WEBHOOK_ID` = `1026186331828785203`

`DISCORD_WEBHOOK_TOKEN` = ```MVcnh2JZwu4frkbR5t-fhovjyFqEfbPxsbLCB8AkHUbBi09AonFwIuJB5eULg5as9p88```



## Copy your validator Consensus key from [explorer](https://haqq.explorers.guru/validators)

`BECH32_CONSVAL_ADDRESS` = ```haqqvalcons1lqjvdhh34crxh4w32f0a7xdr8tk7q28dtzpja0```



### Save your `DISCORD_WEBHOOK_ID` `DISCORD_WEBHOOK_TOKEN` and `BECH32_CONSVAL_ADDRESS`







## Configure config.yaml 

Change `DISCORD_WEBHOOK_ID` `DISCORD_WEBHOOK_TOKEN` `BECH32_CONSVAL_ADDRESS` for your in `config.yaml`
```
nano ~/half-life/config.yaml
```




## Start monitoring
```
halflife monitoring -f ~/half-life/config.yaml
```
