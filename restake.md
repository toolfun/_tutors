**`setup on Ubuntu 22.04 LTS (Jammy Jellyfish)`**
# REStake

https://github.com/eco-stake/restake
____
#### What we'll be doing, briefly
Docker    
Config    
Pull request    
____

### Upgrade packages
```
sudo apt update && sudo apt upgrade
```

### Install Docker
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

#### Updates are happening all the time so updating
```
git pull
docker-compose run --rm app npm install
docker-compose build --no-cache
```

#### There are 2 local config files
- $HOME/restake/src/networks.json
- $HOME/restake/src/networks.local.json - which will override a config in 'networks.json' if you need to

#### The base command for using with docker-compose is
```
docker-compose run --rm app npm run autostake
```

#### You should configure crontab or systemd to run restake periodically
For example, crontab, every day at 08:00
```
crontab -e
0 8 * * * cd restake && docker-compose run --rm app npm run autostake >> ./restake.log 2>&1
```

#### Check logs
```
tail -f -n 100 $HOME/restake.log
```
