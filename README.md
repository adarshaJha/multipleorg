# Multiple Org
## Multi Org/Peer

Install HLF 1.3 on [Ubuntu 16.0.4/18.0.4](https://medium.com/@eSizeDave/https-medium-com-esizedave-how-to-install-hyperledger-fabric-1-2-on-ubuntu-16-04-lts-ecdfa4dcec72)
```
curl -sSL http://bit.ly/2ysbOFE | bash -s <fabric> <fabric-ca> <thirdparty>
curl -sSL http://bit.ly/2ysbOFE | bash -s 1.3.0 1.3.0 0.4.13
```
You will be provisioning a local network with the following docker container configuration:

* 2 CA
* A SOLO orderer
* 2 Org (2 Peers per Org)

#### Links
* [Multi host setup](https://www.cnblogs.com/llongst/p/9608886.html)
* [HLF on multiple hosts](https://medium.com/@wahabjawed/hyperledger-fabric-on-multiple-hosts-a33b08ef24f)
* [Multiple peers running in different physical machines](https://www.skcript.com/svr/setting-up-a-blockchain-business-network-with-hyperledger-fabric-and-composer-running-in-multiple-physical-machine/)

#### Artifacts
* Crypto material has been generated using the **cryptogen** tool from Hyperledger Fabric and mounted to all peers, the orderering node  and CA containers. More details regarding the cryptogen tool are available [here](http://hyperledger-fabric.readthedocs.io/en/latest/build_network.html#crypto-generator).
* An Orderer genesis block (genesis.block) and channel configuration transaction (mychannel.tx) has been pre generated using the **configtxgen** tool from Hyperledger Fabric and placed within the artifacts folder. More details regarding the configtxgen tool are available [here](https://hyperledger-fabric.readthedocs.io/en/latest/build_network.html#configuration-transaction-generator).

###### This example makes use of configtxgen and is for development ONLY. Make use of Fabric-CA for production environments

```
cd /usr/local/hyperledger/fabric-samples/basic-network
cd /usr/local/hyperledger/fabric-samples/demo-app
```

#### Setup:

* [Build Your First Network](https://hyperledger-fabric.readthedocs.io/en/release-1.3/build_network.html)

* Define/Change crypto-config.yaml
  * Make sure you set EnableNodeOUs: true 
* Network Topology
  * OrdererOrgs
    * Name
    * Domain
    * Hostname 
  * PeerOrgs
    * Name 
  * Domain Users
    * Count 1 in addition to Admin
* Define/Change docker-compose.yml 
  * FABRIC_CA_SERVER_CA_KEYFILE
* Define/Change ./generate.sh (Based on Profiles defined in configtx.yaml)
* Define/Change ./start.sh (Change channel name)
* Run startFabric.sh, and replace the new key by inspecting docker logs ca.example.com
  * Everytime you generate new crypto material, make sure you update "FABRIC_CA_SERVER_CA_KEYFILE" in the "docker-compose.yml" file
* Bring up the n/w by running startFabric.sh 
  * launch network; create channel and join peer to channel
  * launch CLI container in order to install, instantiate chaincode
  * bootstrap/ledger with property
* Install node
```
npm install
```
* Enroll Admin and Register the User
  * node registerAdmin.js
  * node registerUser.js
* Start server 
  * node server.js http://localhost:8000
* Project Fauxton
  * Couchdb - http://localhost:5984/_utils (replace 5984 based on docker-compose.yml file)
* Enter the CLI container
  ```
  docker exec -it cli bash
  ```
* Fetch and Join (Under Testing)
  ```
  # Create the channel
  docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.example.com/msp"    peer0.org1.example.com peer channel create -o orderer.example.com:7050 -c ppchannel -f /etc/hyperledger/configtx/channel.tx
  # Join peer0.org1.example.com to the channel
  docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.example.com/msp"   peer0.org1.example.com peer channel join -b ppchannel.block
  # Fetch
  docker exec -e "CORE_PEER_LOCALMSPID=Org2MSP" -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org2.example.com/msp" peer0.org2.example.com peer channel fetch config -o orderer.example.com:7050 -c ppchannel
  # Join "peer0.org2.example.com" to the channel
  docker exec -e "CORE_PEER_LOCALMSPID=Org2MSP" -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org2.example.com/msp" peer0.org2.example.com peer channel join -b ppchannel_config.block
  # Fetch
  # docker exec peer1.org1.example.com peer channel fetch config -o orderer.example.com:7050 -c ppchannel
  # Join "peer1.org1.example.com" to the channel
  # docker exec peer1.org1.example.com peer channel join -b ppchannel_config.block
  # Fetch
  # docker exec peer1.org2.example.com peer channel fetch config -o orderer.example.com:7050 -c ppchannel
  # Join "peer1.org2.example.com" to the channel
  # docker exec peer1.org2.example.com peer channel join -b ppchannel_config.block
  # Update the Anchor Peers Org1MSP
  docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.example.com/msp" peer0.org1.example.com peer channel update -o orderer.example.com:7050 -c ppchannel -f /etc/hyperledger/configtx/Org1MSPanchors.tx
  # Update the Anchor Peers Org2MSP
  docker exec -e "CORE_PEER_LOCALMSPID=Org2MSP" -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org2.example.com/msp" peer0.org2.example.com peer channel update -o orderer.example.com:7050 -c ppchannel -f /etc/hyperledger/configtx/Org2MSPanchors.tx
  ```
* Query History for a Property
  ```
  peer chaincode query -C ppchannel -n demo-app -c '{"Args":["getHistoryForProperty", "2"]}'
  ```
* Rich Query Support (CouchDB)  
  ```
  peer chaincode query -C ppchannel -n demo-app -c '{"Args":["queryPropertyByHolder", "Vettel"]}'
  ```
* Delete State
  ```
  peer chaincode invoke -C ppchannel -n demo-app -c '{"Args":["delete","1"]}'
  ```

#### To Do/Findings:

* Investigate the certificates generated by cryptogen using `openssl x509 -in ./path/to/cert.pem -text -noout` 
* `msp/config.yaml` should NOT be placed into both the orderer and peers' MSP directory. It must ONLY go into the peers
* REST APIs - [HLF Fabric SDK for node.js](https://fabric-sdk-node.github.io/index.html)
  * [Channel](https://fabric-sdk-node.github.io/Channel.html)
    * queryByChaincode
    * queryBlock
    * queryBlockByHash
    * queryTransaction
    * queryInfo
  * [Client](https://fabric-sdk-node.github.io/Client.html)
    * queryInstalledChaincodes
    * queryChannels

#### Clean the network
Clean the containers and artifacts

```
--Docker Housekeeping
docker rm -f $(docker ps -aq)
--docker rmi -f $(docker images -q)

--remove chaincode images from prior runs
docker rmi -f $(docker images | grep peer[0-9]-peer[0-9] | awk '{print $3}')

--Clear any cached networks
docker network prune

--Clear out your previous key value stores that may have cached user enrollment certificates
rm -rf /tmp/hfc-*
rm -rf ~/.hfc-key-store)

--Delete docker log files
find /var/lib/docker/containers/ -type f -name "*.log" -delete
```

#### Discover IP Address
To retrieve the IP Address for one of your network entities, issue the following command:

```
# this will return the IP Address for peer0
docker inspect peer0 | grep IPAddress
```
