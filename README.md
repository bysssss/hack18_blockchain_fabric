## Fabric 구축환경

Docker 17.06.2-ce 이상
Docker Compose 1.14.0 이상
Go 1.10.x 혹은 Go 1.11.x 
Node.js 8.9.x 이상 (9.x is not supported)

<br/>

## Fabric 구축

export FABRIC_CFG_PATH=$PWD
export CHANNEL_NAME=mymarketchannel
configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME
configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Store1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Store1MSP
configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Store2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Store2MSP

docker-compose -f node-orderer.yaml up -d
docker-compose -f node-peer1.yaml up -d
docker-compose -f node-peer2.yaml up -d
docker exec -it cli1 bash

export CHANNEL_NAME=mymarketchannel
peer channel create -o orderer0.mymarket.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx
peer channel join -b $CHANNEL_NAME.block
peer channel update -o orderer0.mymarket.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Store1MSPanchors.tx

peer chaincode install -n marketcc -v 0 -l golang -p github.com/chaincode/mymarket/go
peer chaincode instantiate -o orderer0.mymarket.com:7050 -C $CHANNEL_NAME -n marketcc -l golang -v 0 -c '{"Args":[]}' -P "OR ('Store1MSP.peer','Store2MSP.peer')" --collections-config /opt/gopath/src/github.com/chaincode/mymarket/collections_config.json

peer chaincode query -C $CHANNEL_NAME -n marketcc -c '{"Args":["getProductList"]}'
peer chaincode invoke orderer0.mymarket.com:7050 -C $CHANNEL_NAME -n marketcc -c '{"Args":["registProducts","prod1","1","test"]}'

<br/>
