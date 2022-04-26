# aptos
Aptos network

Run Fullnode Aptos in CENTOS OS

## 1. Install Docker Engine: 

All infomation can be find at: https://docs.docker.com/

- Set up the repository:

```sh
sudo yum install -y yum-utils
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

- Install the latest version of Docker Engine and containerd:

 ```sudo yum install docker-ce docker-ce-cli containerd.io```
 
- Start docker:

```sh
sudo systemctl start docker
```

2. Install Docker Compose

- Download the current stable release of Docker Compose:

```sh
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

- Apply executable permissions to the binary:

```sh
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

- Test the installation: 

 ```sh
 docker-compose --version
 ```
 
3. Install Other package: jq, yq:

```sh
yum install epel-release
yum install jq
yum install yq
```
 
4. Install Aptos

- Make aptos folder and goto:

```sh
cd $HOME
mkdir $HOME/aptos
cd $HOME/aptos
```
- Get file config: 

```sh
wget https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/public_full_node/docker-compose.yaml
wget https://devnet.aptoslabs.com/genesis.blob
wget https://devnet.aptoslabs.com/waypoint.txt
wget https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/public_full_node/public_full_node.yaml
wget --no-check-certificate https://api.zvalid.com/aptos/seeds.yaml
```
- Create identity: 

```sh
docker run --rm --name aptos_tools -d -i aptoslab/tools:devnet
docker exec -it aptos_tools aptos-operational-tool generate-key --encoding hex --key-type x25519 --key-file $HOME/private-key.txt
docker exec -it aptos_tools cat $HOME/private-key.txt > $HOME/aptos/identity/private-key.txt
docker exec -it aptos_tools aptos-operational-tool extract-peer-from-file --encoding hex --key-file $HOME/private-key.txt --output-file $HOME/peer-info.yaml > $HOME/aptos/identity/id.json
PEER_ID=$(cat $HOME/aptos/identity/id.json | jq -r '.Result | keys[]')
PRIVATE_KEY=$(cat $HOME/aptos/identity/private-key.tx
```

-- Add identity (peer_id and private_key) and seed to public_full_node.yaml file config: 

```sh
/usr/local/bin/yq e -i '.full_node_networks[] +=  { "identity": {"type": "from_config", "key": "'$PRIVATE_KEY'", "peer_id": "'$PEER_ID'"} }' $HOME/aptos/public_full_node.yaml
/usr/local/bin/yq ea -i 'select(fileIndex==0).full_node_networks[0].seeds = select(fileIndex==1).seeds | select(fileIndex==0)' $HOME/aptos/public_full_node.yaml $HOME/aptos/seeds.yaml
```

5. Run docker-compose: 

```sh
docker compose up -d
```

6. Check status: 

```sh 
curl 127.0.0.1:9101/metrics 2> /dev/null | grep aptos_state_sync_version | grep type
docker logs -f aptos_fullnode_1 --tail 50
```

7. Update Aptos
```sh
cd ~/aptos
docker-compose down
rm genesis.blob
rm waypoint.txt
rm -rf /var/lib/docker/volumes/aptos_db/_data
mkdir /var/lib/docker/volumes/aptos_db/_data
wget https://devnet.aptoslabs.com/genesis.blob
wget https://devnet.aptoslabs.com/waypoint.txt

docker-compose pull

docker-compose up -d  && curl 127.0.0.1:9101/metrics 2> /dev/null | grep aptos_state_sync_version | grep type
```
