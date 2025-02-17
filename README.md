Generating certifiates using cryptogen: export PATH=${PWD}/../bin:${PWD}:$PATH 
cryptogen generate --config=./organizations/cryptogen/crypto-config-producer.yaml --output="organizations"
cryptogen generate --config=./organizations/cryptogen/crypto-config-wholeseller.yaml --output="organizations" 
cryptogen generate --config=./organizations/cryptogen/crypto-config-supplier.yaml --output="organizations" 
cryptogen generate --config=./organizations/cryptogen/crypto-config-orderer.yaml --output="organizations"
export DOCKER_SOCK=/var/run/docker.sock 
IMAGE_TAG=latest docker-compose -f compose/compose-test-net.yaml -f compose/docker/docker-compose-test-net.yaml up



Creating channel 1: 

export PATH=${PWD}/../bin:${PWD}:$PATH 
export FABRIC_CFG_PATH=${PWD}/configtx 
export CHANNEL_NAME=channel1

configtxgen -profile ThreeOrgsApplicationGenesis -outputBlock ./channel-artifacts/${CHANNEL_NAME}.block -channelID $CHANNEL_NAME 

cp ../config/core.yaml ./configtx/.


export ORDERER_CA=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer1.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

export ORDERER_ADMIN_TLS_SIGN_CERT=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer1.example.com/tls/server.crt

export ORDERER_ADMIN_TLS_PRIVATE_KEY=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer1.example.com/tls/server.key

osnadmin channel join --channelID $CHANNEL_NAME --config-block ./channel-artifacts/${CHANNEL_NAME}.block -o localhost:7053 --ca-file "$ORDERER_CA" --client-cert "$ORDERER_ADMIN_TLS_SIGN_CERT" --client-key "$ORDERER_ADMIN_TLS_PRIVATE_KEY"

export ORDERER_CA=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer2.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

export ORDERER_ADMIN_TLS_SIGN_CERT=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/server.crt

export ORDERER_ADMIN_TLS_PRIVATE_KEY=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/server.key

osnadmin channel join --channelID $CHANNEL_NAME --config-block ./channel-artifacts/${CHANNEL_NAME}.block -o localhost:7054 --ca-file "$ORDERER_CA" --client-cert "$ORDERER_ADMIN_TLS_SIGN_CERT" --client-key "$ORDERER_ADMIN_TLS_PRIVATE_KEY"

export ORDERER_CA=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer3.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

export ORDERER_ADMIN_TLS_SIGN_CERT=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer3.example.com/tls/server.crt

export ORDERER_ADMIN_TLS_PRIVATE_KEY=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer3.example.com/tls/server.key

osnadmin channel join --channelID $CHANNEL_NAME --config-block ./channel-artifacts/${CHANNEL_NAME}.block -o localhost:7055 --ca-file "$ORDERER_CA" --client-cert "$ORDERER_ADMIN_TLS_SIGN_CERT" --client-key "$ORDERER_ADMIN_TLS_PRIVATE_KEY"

Here in envVar.sh each file we have defined setGlobals() in such a way that it can input both the number of the organisation and the number of peer to be joined.

source ./scripts/setOrgPeerContext.sh 1 0

peer channel join -b ./channel-artifacts/${CHANNEL_NAME}.block

source ./scripts/setOrgPeerContext.sh 1 1

peer channel join -b ./channel-artifacts/${CHANNEL_NAME}.block

source ./scripts/setOrgPeerContext.sh 2 0

peer channel join -b ./channel-artifacts/${CHANNEL_NAME}.block

source ./scripts/setOrgPeerContext.sh 2 1

peer channel join -b ./channel-artifacts/${CHANNEL_NAME}.block

source ./scripts/setOrgPeerContext.sh 3 0

peer channel join -b ./channel-artifacts/${CHANNEL_NAME}.block

source ./scripts/setOrgPeerContext.sh 3 1

peer channel join -b ./channel-artifacts/${CHANNEL_NAME}.block


in setAnchorPeer.sh please change ORDERER_CA to ORDERER1_CA to avoid errors


source ./scripts/setOrgAnchorPeerContext.sh 1

docker exec cli ./scripts/setAnchorPeer.sh 1 $CHANNEL_NAME

source ./scripts/setOrgAnchorPeerContext.sh 2

docker exec cli ./scripts/setAnchorPeer.sh 2 $CHANNEL_NAME

source ./scripts/setOrgAnchorPeerContext.sh 3

docker exec cli ./scripts/setAnchorPeer.sh 3 $CHANNEL_NAME



Deploying the chaincode


source ./scripts/setChaincodeContext.sh


source ./scripts/setOrgPeerContext.sh 1 0

peer lifecycle chaincode package supply1.tar.gz --path ${CC_SRC_PATH} --lang ${CC_RUNTIME_LANGUAGE} --label supply1_${VERSION}

peer lifecycle chaincode install supply1.tar.gz


source ./scripts/setOrgPeerContext.sh 1 1

peer lifecycle chaincode install supply1.tar.gz

source ./scripts/setOrgPeerContext.sh 2 0

peer lifecycle chaincode install supply1.tar.gz
source ./scripts/setOrgPeerContext.sh 2 1

peer lifecycle chaincode install supply1.tar.gz
source ./scripts/setOrgPeerContext.sh 3 0

peer lifecycle chaincode install supply1.tar.gz
source ./scripts/setOrgPeerContext.sh 3 1

peer lifecycle chaincode install supply1.tar.gz

peer lifecycle chaincode queryinstalled 2>&1 | tee outfile

source ./scripts/setPackageID.sh outfile

source ./scripts/setOrgPeerContext.sh 1

peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer1.example.com --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA --channelID $CHANNEL_NAME --name supply1 --version ${VERSION} --init-required --package-id ${PACKAGE_ID} --sequence ${VERSION}


peer lifecycle chaincode checkcommitreadiness --channelID $CHANNEL_NAME --name supply1 --version ${VERSION} --sequence ${VERSION} --output json --init-required
source ./scripts/setOrgPeerContext.sh 3
peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer1.example.com --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA --channelID $CHANNEL_NAME --name supply1 --version ${VERSION} --init-required --package-id ${PACKAGE_ID} --sequence ${VERSION}
source ./scripts/setOrgPeerContext.sh 5
peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer1.example.com --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA --channelID $CHANNEL_NAME --name supply1 --version ${VERSION} --init-required --package-id ${PACKAGE_ID} --sequence ${VERSION}


source ./scripts/setOrgPeerContext.sh 1
peer lifecycle chaincode checkcommitreadiness --channelID $CHANNEL_NAME --name supply1 --version ${VERSION} --sequence ${VERSION} --output json --init-required


source ./scripts/setPeerConnectionParam.sh 1 2 3 4 5 6
peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer1.example.com --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA --channelID $CHANNEL_NAME --name supply1 $PEER_CONN_PARAMS --version ${VERSION} --sequence ${VERSION} --init-required

source ./scripts/setOrgPeerContext.sh 1
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer1.example.com --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C channel1 -n supply1 $PEER_CONN_PARAMS --isInit -c '{"function":"InitLedger","Args":[]}'

peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer1.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer1.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C channel1 -n supply1 $PEER_CONN_PARAMS -c '{"function":"GetAllAssets","Args":[]}'
