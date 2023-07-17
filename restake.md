**`setup on Ubuntu 22.04 LTS (Jammy Jellyfish)`**
# REStake

https://github.com/eco-stake/restake
____
#### What we'll be doing, briefly
Docker    
Config    
Submit operator    

#

### Install Docker
#### Upgrade packages
```
sudo apt update && sudo apt upgrade
```
#### Add the GPG key to the system for the official Docker repository 
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o docker.gpg
sudo mv docker.gpg /etc/apt/trusted.gpg.d/
```

#### Add Docker repository to apt
```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu jammy stable"
```

#### Update packages database and add newly Docker packages
```
sudo apt update
```

#### Check sources for Docker installation
```
apt-cache policy docker-ce
```
output:    
```bash
docker-ce:
  Installed: (none)
  Candidate: 5:24.0.4-1~ubuntu.22.04~jammy
  Version table:
     5:24.0.4-1~ubuntu.22.04~jammy 500
        500 https://download.docker.com/linux/ubuntu jammy/stable amd64 Packages    
```

#### Installing Docker
```
sudo apt install docker-ce
```

#### Check for Docker is running
```
sudo systemctl status docker
```

#

### Install Docker Compose
Go to https://github.com/docker/compose/releases for the last release version    
I'll use docker-compose version 2.20.0
```
dcv=v2.20.0
sudo curl -L "https://github.com/docker/compose/releases/download/$dcv/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

#### Make docker-compose executable
```
sudo chmod +x /usr/local/bin/docker-compose
```

#### Check if the installation was successful
```
docker-compose --version
```

#

### REStake configuration
#### Clone restake repo
```
git clone https://github.com/eco-stake/restake
```

#### Copy config sample
```
cd restake
cp .env.sample .env
```

#### Create new wallet that you will use for the staking transactions
Keep enough funds for transaction fees on this wallet
#### Open file and add mnemonic of this wallet
```
nano $HOME/restake/.env
```

> #### Updates are happening all the time so this command for updating
> ```
> git pull
> docker-compose run --rm app npm install
> docker-compose build --no-cache
> ```

#### There are 2 local config files
- `$HOME/restake/src/networks.json`
- `$HOME/restake/src/networks.local.json` - which will override a config in `networks.json` if you need to. You can check `networks.local.json.sample` example in $HOME/restake/src/     

My `networks.local.json` working file:
```JSON
{
  "empowerchain": {
    "prettyName": "EmpowerChain",
    "restUrl": [
      "https://empower-api.polkachu.com"
    ],
    "gasPrice": "0.025umpwr",
    "autostake": {
      "retries": 3,
      "batchPageSize": 100,
      "batchQueries": 20,
      "batchTxs": 50,
      "delegationsTimeout": 20000,
      "queryTimeout": 5000,
      "queryThrottle": 100,
      "gasModifier": 1.2
    }
  }
}
```
> Comments
```JSON
"empowerchain": { // Network name
"prettyName": "EmpowerChain", // Network's user-friendly name
"restUrl": // REST API URL for the network
"gasPrice": // Standard gas price for transactions in this network
"retries": // Number of attempts the script will make when performing an operation
"batchPageSize": // Number of delegates to be processed at once
"batchQueries": // Number of queries to be executed at once
"batchTxs": // Number of transactions to be sent at once
"delegationsTimeout": // Timeout (in milliseconds) for delegation queries
"queryTimeout": // Timeout (in milliseconds) for other queries
"queryThrottle": // Delay (in milliseconds) between queries
"gasModifier": // Modifier to be applied to the standard gas price
```

#### The base command for using with docker-compose is
```
docker-compose run --rm app npm run autostake
```

#### You should configure crontab or systemd to run restake periodically
For example, crontab, at the beginning of each hour
```
crontab -e
0 * * * * cd restake && docker-compose run --rm app npm run autostake >> ./restake.log 2>&1
```

#### Checking logs
```
tail -f -n 100 $HOME/restake.log
```

#

### Submitting
#### Update the validator registry to add your operator information to all chains for which you have set up REStake
Fork https://github.com/eco-stake/validator-registry.
You should create 2 files, `chains.json` and `profile.json`. You can find an examples at other validators in the registry.
> #### profile.json example
```
{
  "$schema": "../profile.schema.json",
  "name": "xAlex",
  "identity": "224B4B470642E35E"
}
```
> #### chains.json example
```
{
  "$schema": "../chains.schema.json",
  "name": "xAlex",
  "chains": [    
    {
      "name": "empowerchain",
      "address": "empowervaloper1eu44mtweaap45ytv6hgtkahhmkdlx9rlk0py42",
			"restake": {
			  "address": "empower1jyhp2zcr8dd62gq3rtxse6mw4wypkej7lcl8s6",
			  "run_time": ["every 1 hour"],
			  "minimum_reward": 1000000
			}
		}
	]
}
```
> #### comments
```JSON
"name": // validator moniker
"chains": // chains you configuring for REStake
"name": // the name of the chain
"address": // your validator's address in this chain
"restake":
 "address": // address from your new wallet you generated earlier for the restake transactions
 "run_time": // time in UTC or interval string ( e.g. "every 15 minutes") that you intend to run your bot
 "minimum_reward": // minimum reward to trigger autostaking, skipping else
```

#### 
```

```


