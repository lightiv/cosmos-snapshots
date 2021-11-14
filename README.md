# cosmos-snapshots  
List of snapshots:   
http://135.181.60.250/      - Akash Network (Mainnet)  
http://162.255.116.68/snapshots/ - Akash Network (Mainnet)  
http://135.181.60.250:8081/ - Sifchain (Betanet)  
http://135.181.60.250:8083/ - Sentinel (Mainnet)  
http://135.181.60.250:8084/ - Desmos (Mainnet)  
http://135.181.60.250:8085/ - Osmosis (Mainnet)  
http://135.181.60.250:8086/ - Bitcanna (Mainnet)  
http://snapshots.alexvalidator.com:8081/  - Oasis (Mainnet)  
https://snapshots.stakecraft.com/ - Juno (Mainnet)  
http://cosmos-snap.staketab.com/ixo - IXO (Mainnet)  
http://cosmos-snap.staketab.com/medibloc - Medibloc (Mainnet)  
http://cosmos-snap.staketab.com/stargaze - Stargaze (Mainnet)  

https://www.notion.so/Stake-Systems-Fast-Sync-Service-5cb0dffb78174d3494b93f87d242939d
- juno
- osmosis
- agoric
- regen
- cosmoshub
- irishub
- matic
- emoney
- bitsong
- terra
- kava
- certik
- microtick
- celo-baklava non-archive

[Akash snapshot instruction](https://github.com/c29r3/cosmos-snapshots/blob/main/Akash.md)  
[Sifchain snapshot instruction](https://github.com/c29r3/cosmos-snapshots/blob/main/Sifchain.md)  
[Sentinel snapshot instruction](https://github.com/c29r3/cosmos-snapshots/blob/main/Sentinel.md)  
[Desmos snapshot instruction](https://github.com/c29r3/cosmos-snapshots/blob/main/Desmos.md)  
[Osmosis snapshot instruction](https://github.com/c29r3/cosmos-snapshots/blob/main/Osmosis.md)  
[Bitcanna snapshot instruction](https://github.com/c29r3/cosmos-snapshots/blob/main/Bitcanna.md)  
[Oasis snapshot instruction](https://github.com/Bambarello/cosmos-snapshots/blob/main/Oasis.md)  
[IXO snapshot instruction](https://github.com/staketab/nginx-cosmos-snap/blob/main/ixo.md)  
[Medibloc snapshot instruction](https://github.com/staketab/nginx-cosmos-snap/blob/main/medibloc.md)  
[Stargaze snapshot instruction](https://github.com/staketab/nginx-cosmos-snap/blob/main/stargaze.md)  


## MIRRORS  
http://rpc01-skynet.paullovette.com/ - provided by Paul Lovette  
http://162.255.116.68/snapshots/ - privded by Min (Min#6706)  
https://snapshots.stakecraft.com/    - provided by Alex Novy  
http://snapshots.alexvalidator.com:8081/  - provided by Bambarello  
http://cosmos-snap.staketab.com/  - provided by Staketab  
https://www.notion.so/Stake-Systems-Fast-Sync-Service-5cb0dffb78174d3494b93f87d242939d - provided by Stake Systems  


## Run your own backup server with snapshots  
Install requirements  
```bash
sudo apt update && \
sudo apt install curl git docker.io -y
```

Clone github repo  
`git clone https://github.com/c29r3/cosmos-snapshots.git && cd cosmos-snapshots`  

Create folder for snapshots  
`mkdir $HOME/akash`

Start Nginx via docker  
```bash
cd $HOME; \
docker run --name nginx \
--restart always \
-v $(pwd)/default.conf:/etc/nginx/conf.d/default.conf \
-v $(pwd)/akash/:/root/ \
-p 80:80 \
-d nginx
```

Fill in the variables and the file `akash_snapshot.sh`  
```
#!/bin/bash
CHAIN_ID="akashnet-2"
SNAP_PATH="/home/$USER/akash/akash"
LOG_PATH="/home/$USER/cosmos-snapshots/daily-logs/akash_log.txt"
DATA_PATH="/home/$USER/.akash/data/"
SERVICE_NAME="akash.service"
RPC_ADDRESS="http://localhost:26657"
SNAP_NAME=$(echo "${CHAIN_ID}_$(date '+%Y-%m-%d').tar")
OLD_SNAP=$(ls ${SNAP_PATH} | egrep -o "${CHAIN_ID}.*tar")


now_date() {
    echo -n $(TZ=":America/Los_Angeles" date '+%Y-%m-%d_%H:%M:%S')
}


log_this() {
    YEL='\033[1;33m' # yellow
    NC='\033[0m'     # No Color
    local logging="$@"
    printf "|$(now_date)| $logging\n" | tee -a ${LOG_PATH}
}

LAST_BLOCK_HEIGHT=$(curl -s ${RPC_ADDRESS}/status | jq -r .result.sync_info.latest_block_height)
log_this "LAST_BLOCK_HEIGHT ${LAST_BLOCK_HEIGHT}"

log_this "Stopping ${SERVICE_NAME}"
systemctl stop ${SERVICE_NAME}; echo $? >> ${LOG_PATH}

log_this "Creating new snapshot"
time tar cf ${HOME}/${SNAP_NAME} -C ${DATA_PATH} . &>>${LOG_PATH}

log_this "Starting ${SERVICE_NAME}"
systemctl start ${SERVICE_NAME}; echo $? >> ${LOG_PATH}

log_this "Removing old snapshot(s):"
cd ${SNAP_PATH}
rm -fv ${OLD_SNAP} &>>${LOG_PATH}

log_this "Moving new snapshot to ${SNAP_PATH}"
mv ${HOME}/${CHAIN_ID}*tar ${SNAP_PATH} &>>${LOG_PATH}


du -hs ${SNAP_PATH} | tee -a ${LOG_PATH}

log_this "Done\n---------------------------\n"
```
Create new snapshot  
`./akash_snapshot.sh`  

Check snapshot  
```bash
MY_IP=$(curl -s 2ip.ru); \
curl -s http://${MY_IP}
```

## Automation  
You can add script to the cron  
```cron
# start every day at 00:00
0 0 * * * /bin/bash -c '/root/akash_snapshot.sh'
```
