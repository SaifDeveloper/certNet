export PATH=${PWD}/../bin:$PATH

# Cryptogen

cryptogen generate --config=./organizations/cryptogen/crypto-config-iiit.yaml --output="organizations"

cryptogen generate --config=./organizations/cryptogen/crypto-config-acme.yaml --output="organizations"

cryptogen generate --config=./organizations/cryptogen/crypto-config-mhrd.yaml --output="organizations"

cryptogen generate --config=./organizations/cryptogen/crypto-config-orderer.yaml --output="organizations"

# Genesis Block

export FABRIC_CFG_PATH=${PWD}/configtx

configtxgen -profile TwoOrgsOrdererGenesis -channelID system-channel -outputBlock ./system-genesis-block/genesis.block

# Start peer, orderer containers

export IMAGE_TAG=latest
export COMPOSE_PROJECT_NAME=net

docker-compose -f docker-compose-test-net.yaml -f docker-compose-couch.yaml up -d

# Create Channel

peer channel create -o localhost:7050 -c mychannel --ordererTLSHostnameOverride orderer.certification-network.com -f ./channel-artifacts/mychannel.tx --outputBlock ./channel-artifacts/mychannel.block --tls --cafile /home/saif/codeStuff/cert-net/organizations/ordererOrganizations/certification-network.com/orderers/orderer.certification-network.com/msp/tlscacerts/tlsca.certification-network.com-cert.pem

# Fetching the most recent configuration block for the channel

peer channel fetch config config_block.pb -o orderer.certification-network.com:7050 --ordererTLSHostnameOverride orderer.certification-network.com -c certificationchannel --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/organizations/ordererOrganizations/certification-network.com/orderers/orderer.certification-network.com/msp/tlscacerts/tlsca.certification-network.com-cert.pem

peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.certification-network.com --tls --cafile /home/saif/codeStuff/cert-net/organizations/ordererOrganizations/certification-network.com/orderers/orderer.certification-network.com/msp/tlscacerts/tlsca.certification-network.com-cert.pem --channelID certificationchannel --name basic 
--peerAddresses localhost:7051 --tlsRootCertFiles /home/saif/codeStuff/cert-net/organizations/peerOrganizations/iiit.certification-network.com/peers/peer0.iiit.certification-network.com/tls/ca.crt -
-peerAddresses localhost:9051 --tlsRootCertFiles /home/saif/codeStuff/cert-net/organizations/peerOrganizations/mhrd.certification-network.com/peers/peer0.mhrd.certification-network.com/tls/ca.crt 
--peerAddresses localhost:11051 --tlsRootCertFiles /home/saif/codeStuff/cert-net/organizations/peerOrganizations/acme.certification-network.com/peers/peer0.acme.certification-network.com/tls/ca.crt -
-version 1.0 --sequence 1