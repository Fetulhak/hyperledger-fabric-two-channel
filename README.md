# hyperledger-fabric-two-channel
This repository demonstrates how to add organizations and create two channels by using hyperledger fabric test network.

Note: Before you start download the folders and files found in this repo and replace the same file found in fabric-samples/test-network. These folders andxto files contain potential modifications done on the fabric test network architecture to suit the tutorial in this repo. Good Luck!!! 

In order to add organizations to the test-network configuration given by the hyperledger fabric initial settings I have modified the following files. You can compare the modification by seeing the original files on the hyperledger fabric github source cde.

1. The configtx.yaml file is modified to include additional 3 organization (Org3, Org4 and Org5) following similar settings to the already existing two organizations (Org1 and Org2). In this same file the profile section at the bottom is modified. I have used the same profile name "ChannelUsingRaft" as the original file but modified the organizations section in the application section to include all the organizations.

2. In the compose folder i have modified the compose-ca.yaml and compose-test-net.yaml files. you can compare the diffrences and how it is modified by comparing the files with the original ones provided in the hyperledger fabric github repo. if you want to work on couchdb database you can also modify the compose-couch.yaml file but I did not use couchdb so I do not modify this file.

3. in compose folder and docker subfolder I modified docker-compose-ca.yaml and docker-compose-test-net.yaml files. you can compare the modification by seeing the original files from hyperledger fabric github repo.


4. under test-network/organizations/cryptogen folder i have added crpto-config.yaml files for the added organizations (org3,org4,org5), check how the files are created, These files are used to generate the cryptographic materials for each organization which are necessary to communicate and interact with eachother. under test-network/organizations/fabric-ca I have added org3, org4 and org5 sub-folder to put a modified fabric-ca-server-config.yaml files on each sub-folder. you can check the modification by seeing the original files for org1 and org2 in hyperledger fabric github repo. These files are used when you want to generate the cryptographic materials using fabric CA service. under test-network/organizations i have also modified the ccp-generate.sh file, you can also check for the modifications on this file as well. I have also modified the registerenroll.sh file under est-network/organizations/fabric-ca, you can check the modification, this file is used when you generate crypto materials using fabric ca.


5. In hyperledger fabric test-network the network.sh file is the main script used to up the network, by generating cryptographic material, creating channel, joining peers etc... However I have modified createOrgs() function by including options to create chryptographic materials for the added 3 organizations. you can check how the modification is done by comparing to the original network.sh file in hyperledger fabric github repo.

These are all the modifications I have done to add organizations in the existing test-network example given by the hyperledger fabric github repository. The modificatios made still now can enable to bring up the 5 peer organizations (org1 and org2 already existing on test-network example and org3, org4,org5 added based on the modifications) and the orderer organization which is already existing on the fabric test-network (no-modifications on the orderer node). When you run ./network.sh up in command line after the modification, then the crptographic materials for all the organizations (peers as well as orderer) will be generated. 


The next step is to create the genesis block and join the orderer node to the genesis block and create application channels which can be joined by our peer nodes. For these steps I have followed manual procedures as given below...

1. run "./network.sh down" and "docker system prune --volumes -f" to clean already generated containers and volumes 
2. run "./network.sh up" , if there is no error in your modification all the peer and orderer organization will be up and crypto material will be create fo all of them 
3. The next step is to create an application channel. You can read and follow the link from the hyperledger fabric documentation https://hyperledger-fabric.readthedocs.io/en/release-2.3/create_channel/create_channel_test_net.html. Fabric v2.3 introduces the capability to create a channel without requiring a system channel and I have modified the configtx.yaml file based on this approach.

3.1 To create an application channel the first step is to generate the genesis block of the channels, for this we use configtxgen tool which is found under fabric-samples/bin folder. We laso need the configtx.yaml file which is modified for our use case, it is located under fabric-samples/test-network/configtx folder. We should have to export this two paths when we are working under our $PWD which is home/fabric-samples/test-network. In my use case I want to create two application channels where "channel1" will be joined by peer orgs (Org1 and Org2) while the second channel "chennel2" will be joined by peer orgs (Org3,Org4 and Org5). 

3.2 export PATH=${PWD}/../bin:$PATH  export FABRIC_CFG_PATH=${PWD}/configtx

3.3 to create the genesis blocks for the two channels mentioned above use these two commands : configtxgen -profile ChannelUsingRaft1 -outputBlock ./channel-artifacts/channel1.block -channelID channel1 and configtxgen -profile ChannelUsingRaft2 -outputBlock ./channel-artifacts/channel2.block -channelID channel2

The profile used should be setted at the profiles section of the configtx.yaml file. In my case I have modified the profiles section in the existing configtx.yaml file by adding two profiles named "ChannelUsingRaft1" and "ChannelUsingRaft3". If you want more channels, for example three channels, you can add one more profile on the configtx.yaml file. In my case I have used one single orderer organization to manage all the channels but that can also be modified.

3.4 Now we have generated the genesis blocks for the two channels. Now it is time to create the two application channels one by one, we use "osnadmin channel join" command. before that lets set the environment variable for the orderer admin which is the one to perform "osnadmin channel join" command. set the following env. variables

export ORDERER_CA=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
export ORDERER_ADMIN_TLS_SIGN_CERT=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
export ORDERER_ADMIN_TLS_PRIVATE_KEY=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.key


3.5 Run the following command to create the first channel named channel1 on the ordering service.

osnadmin channel join --channelID channel1 --config-block ./channel-artifacts/channel1.block -o localhost:7053 --ca-file "$ORDERER_CA" --client-cert "$ORDERER_ADMIN_TLS_SIGN_CERT" --client-key "$ORDERER_ADMIN_TLS_PRIVATE_KEY"

3.6 Run the following command to create the second channel named channel2 on the ordering service.

osnadmin channel join --channelID channel2 --config-block ./channel-artifacts/channel2.block -o localhost:7053 --ca-file "$ORDERER_CA" --client-cert "$ORDERER_ADMIN_TLS_SIGN_CERT" --client-key "$ORDERER_ADMIN_TLS_PRIVATE_KEY"

3.7 You can run the following command to check the list of channels managed by the ordering service

osnadmin channel list -o localhost:7053 --ca-file "$ORDERER_CA" --client-cert "$ORDERER_ADMIN_TLS_SIGN_CERT" --client-key "$ORDERER_ADMIN_TLS_PRIVATE_KEY"


3.8 Now we have created the two applications, it is time to join peer organizations to the channels. In channel1 we will join org1 and org2 and in channel2 we will join org3,org4 and org5. Lets first join org1 and org2 to channel 1. Set the following environment variables to indicate that we are acting as the Org1 admin and targeting the Org1 peer.

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051


In order to use the peer CLI, we also need to modify the FABRIC_CONFIG_PATH:

export FABRIC_CFG_PATH=$PWD/../config/

To join the test network peer from Org1 to the channel channel1 simply pass the genesis block in a join request:

peer channel join -b ./channel-artifacts/channel1.block

We repeat these steps for the Org2 peer. Set the following environment variables to operate the peer CLI as the Org2 admin. The environment variables will also set the Org2 peer, peer0.org2.example.com, as the target peer.

export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:8051

Now repeat the command to join the peer from Org2 to channel1:

peer channel join -b ./channel-artifacts/channel1.block


Now we have compeleted joining the two peers from org1 and org2 to channel1. next we will join the peers of org3,org4 and org5 to channel2. Set the following environment variables to indicate that we are acting as the Org3 admin and targeting the Org3 peer.

export CORE_PEER_LOCALMSPID="Org3MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org3.example.com/users/Admin@org3.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051


To join the test network peer from Org3 to the channel channel2 simply pass the genesis block in a join request:

peer channel join -b ./channel-artifacts/channel2.block

We repeat these steps for the Org4 peer.

export CORE_PEER_LOCALMSPID="Org4MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org4.example.com/peers/peer0.org4.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org4.example.com/users/Admin@org4.example.com/msp
export CORE_PEER_ADDRESS=localhost:10051


Now repeat the command to join the peer from Org4 to channel2:

peer channel join -b ./channel-artifacts/channel2.block


We repeat these steps for the Org5 peer.

export CORE_PEER_LOCALMSPID="Org5MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org5.example.com/peers/peer0.org5.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org5.example.com/users/Admin@org5.example.com/msp
export CORE_PEER_ADDRESS=localhost:11051


Now repeat the command to join the peer from Org5 to channel2:

peer channel join -b ./channel-artifacts/channel2.block



3.9 Now we have finished joining peers of organizations which are members of each channel. now it is time to set anchor peers for each organization. After an organization has joined their peers to the channel, they should select at least one of their peers to become an anchor peer. Each channel member can specify their anchor peers by updating the channel. We will use the configtxlator tool to update the channel configuration and select an anchor peer for Org1 and Org2 in channel1 and Org3, Org4 and Org5 in channel2.

We will start by selecting the peer from Org1 to be an anchor peer. The first step is to pull the most recent channel configuration block using the peer channel fetch command. Set the following environment variables to operate the peer CLI as the Org1 admin:

export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051

You can use the following command to fetch the channel configuration:

peer channel fetch config channel-artifacts/config_block.pb -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com -c channel1 --tls --cafile "$ORDERER_CA"

change directory to  channel-artifacts

cd channel-artifacts

The first step is to decode the block from protobuf into a JSON object that can be read and edited. use the following command 

configtxlator proto_decode --input config_block.pb --type common.Block --output config_block.json
jq '.data.data[0].payload.data.config' config_block.json > config.json

Because we don’t want to edit this file directly, we will make a copy that we can edit. We will use the original channel config in a future step. use the following command to copy

cp config.json config_copy.json

You can use the jq tool to add the Org1 anchor peer to the channel configuration.

jq '.channel_group.groups.Application.groups.Org1MSP.values += {"AnchorPeers":{"mod_policy": "Admins","value":{"anchor_peers": [{"host": "peer0.org1.example.com","port": 7051}]},"version": "0"}}' config_copy.json > modified_config.json


After this step, we have an updated version of channel configuration in JSON format in the modified_config.json file. We can now convert both the original and modified channel configurations back into protobuf format and calculate the difference between them.

configtxlator proto_encode --input config.json --type common.Config --output config.pb
configtxlator proto_encode --input modified_config.json --type common.Config --output modified_config.pb
configtxlator compute_update --channel_id channel1 --original config.pb --updated modified_config.pb --output config_update.pb

The new protobuf named config_update.pb contains the anchor peer update that we need to apply to the channel configuration. We can wrap the configuration update in a transaction envelope to create the channel configuration update transaction.y

configtxlator proto_decode --input config_update.pb --type common.ConfigUpdate --output config_update.json
echo '{"payload":{"header":{"channel_header":{"channel_id":"channel1", "type":2}},"data":{"config_update":'$(cat config_update.json)'}}}' | jq . > config_update_in_envelope.json
configtxlator proto_encode --input config_update_in_envelope.json --type common.Envelope --output config_update_in_envelope.pb

We can now use the final artifact, config_update_in_envelope.pb, that can be used to update the channel. Navigate back to the test-network directory:

cd ..

We can add the anchor peer by providing the new channel configuration to the peer channel update command. Because we are updating a section of the channel configuration that only affects Org1, other channel members do not need to approve the channel update.


peer channel update -f channel-artifacts/config_update_in_envelope.pb -c channel1 -o localhost:7050  --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"

Now we have compeleted setting anchor peer for Org1 in channel1. Now we will set Org2 anchor peer in channel1 by following similar procedures. First let's set the environment variables for Org2

export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:8051

fetch the latest channel configuration block:

peer channel fetch config channel-artifacts/config_block.pb -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com -c channel1 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"

Navigate back to the channel-artifacts directory:

cd channel-artifacts

You can then decode and copy the configuration block.

configtxlator proto_decode --input config_block.pb --type common.Block --output config_block.json
jq '.data.data[0].payload.data.config' config_block.json > config.json
cp config.json config_copy.json

Add the Org2 peer that is joined to the channel1 as the anchor peer in the channel1 configuration:

jq '.channel_group.groups.Application.groups.Org2MSP.values += {"AnchorPeers":{"mod_policy": "Admins","value":{"anchor_peers": [{"host": "peer0.org2.example.com","port": 8051}]},"version": "0"}}' config_copy.json > modified_config.json

We can now convert both the original and updated channel configurations back into protobuf format and calculate the difference between them.

configtxlator proto_encode --input config.json --type common.Config --output config.pb
configtxlator proto_encode --input modified_config.json --type common.Config --output modified_config.pb
configtxlator compute_update --channel_id channel1 --original config.pb --updated modified_config.pb --output config_update.pb


Wrap the configuration update in a transaction envelope to create the channel configuration update transaction:

configtxlator proto_decode --input config_update.pb --type common.ConfigUpdate --output config_update.json
echo '{"payload":{"header":{"channel_header":{"channel_id":"channel1", "type":2}},"data":{"config_update":'$(cat config_update.json)'}}}' | jq . > config_update_in_envelope.json
configtxlator proto_encode --input config_update_in_envelope.json --type common.Envelope --output config_update_in_envelope.pb


Navigate back to the test-network directory

cd ..

Update the channel and set the Org2 anchor peer by issuing the following command:


peer channel update -f channel-artifacts/config_update_in_envelope.pb -c channel1 -o localhost:7050  --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"


Now we have compeleted setting anchor peers for all the organizations which are memebres of channel1. Now it is time to set anchor peers for all the organizations which are members of channel2. Let's start by setting anchor peer for Org3 in channel2. Start by setting the environment variable for Org3

export CORE_PEER_LOCALMSPID="Org3MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org3.example.com/users/Admin@org3.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051

fetch the latest channel configuration block: This time we have to change the channel ID to channel2 as you can see from the command below -c option has channel2 as channel id.

peer channel fetch config channel-artifacts/config_block.pb -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com -c channel2 --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"

Navigate back to the channel-artifacts directory:

cd channel-artifacts

You can then decode and copy the configuration block.

configtxlator proto_decode --input config_block.pb --type common.Block --output config_block.json
jq '.data.data[0].payload.data.config' config_block.json > config.json
cp config.json config_copy.json

Add the Org3 peer that is joined to the channel2 as the anchor peer in the channel2 configuration:

jq '.channel_group.groups.Application.groups.Org3MSP.values += {"AnchorPeers":{"mod_policy": "Admins","value":{"anchor_peers": [{"host": "peer0.org3.example.com","port": 9051}]},"version": "0"}}' config_copy.json > modified_config.json

We can now convert both the original and updated channel configurations back into protobuf format and calculate the difference between them. change channel id to channel2 here as well.

configtxlator proto_encode --input config.json --type common.Config --output config.pb
configtxlator proto_encode --input modified_config.json --type common.Config --output modified_config.pb
configtxlator compute_update --channel_id channel2 --original config.pb --updated modified_config.pb --output config_update.pb


Wrap the configuration update in a transaction envelope to create the channel configuration update transaction: change channel id to channel2.

configtxlator proto_decode --input config_update.pb --type common.ConfigUpdate --output config_update.json
echo '{"payload":{"header":{"channel_header":{"channel_id":"channel2", "type":2}},"data":{"config_update":'$(cat config_update.json)'}}}' | jq . > config_update_in_envelope.json
configtxlator proto_encode --input config_update_in_envelope.json --type common.Envelope --output config_update_in_envelope.pb


Navigate back to the test-network directory

cd ..

Update the channel and set the Org3 anchor peer by issuing the following command: change channel id to channel2 by using -c channel2 in below command


peer channel update -f channel-artifacts/config_update_in_envelope.pb -c channel2 -o localhost:7050  --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"


Now we have compeleted setting Org3 anchor peer in channel2. Follow similar procedures above by changing the environment varioables to Org4 and Org5 to set their anchor peers in channel2.


3.10 Now we have finished setting anchor peers for all organizations in channel1 and channel2. Next we will deploy chaincode on the channels. We can confirm that the channel was created successfully by deploying a chaincode to the channel. You can follow or read the procedures to deploy chaincode on a given channel by using the hyperledger fabric test-network from this link: https://hyperledger-fabric.readthedocs.io/en/release-2.3/deploy_chaincode.html. A chaincode is deployed to a channel using a process known as the Fabric chaincode lifecycle. 

3.10.1 first we need to package the chaincode before it can be installed on our peers. In this tutorial I will use the GO language based chaincode or smart contract to deploy on peers of the network. 


Before we package the chaincode, we need to install the chaincode dependences. Navigate to the folder that contains the Go version of the asset-transfer (basic) chaincode.

cd fabric-samples/asset-transfer-basic/chaincode-go

To install the smart contract dependencies, run the following command

GO111MODULE=on go mod vendor

Now that we that have our dependences, we can create the chaincode package. Navigate back to our working directory in the test-network folder so that we can package the chaincode

cd ../../test-network

You can use the peer CLI to create a chaincode package in the required format. The peer binaries are located in the bin folder of the fabric-samples repository. Use the following command to add those binaries to your CLI Path:

export PATH=${PWD}/../bin:$PATH

You also need to set the FABRIC_CFG_PATH to point to the core.yaml file in the fabric-samples repository:

export FABRIC_CFG_PATH=$PWD/../config/

You can now create the chaincode package using the peer lifecycle chaincode package command:

peer lifecycle chaincode package basic.tar.gz --path ../asset-transfer-basic/chaincode-go/ --lang golang --label basic_1.0

This command will create a package named basic.tar.gz in your current directory.

3.10.2 After we package the asset-transfer (basic) smart contract, we can install the chaincode on our peers. The chaincode needs to be installed on every peer that will endorse a transaction. 

Let’s install the chaincode on the Org1 peer first. Set the following environment variables to operate the peer CLI as the Org1 admin user. The CORE_PEER_ADDRESS will be set to point to the Org1 peer, peer0.org1.example.com.

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051

Issue the peer lifecycle chaincode install command to install the chaincode on the peer:

peer lifecycle chaincode install basic.tar.gz


We can now install the chaincode on the Org2 peer. Set the following environment variables to operate as the Org2 admin and target the Org2 peer, peer0.org2.example.com.

export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:8051

peer lifecycle chaincode install basic.tar.gz


We can now install the chaincode on the Org3 peer. Set the following environment variables to operate as the Org3 admin and target the Org3 peer, peer0.org3.example.com.

export CORE_PEER_LOCALMSPID="Org3MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org3.example.com/users/Admin@org3.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051

peer lifecycle chaincode install basic.tar.gz



We can now install the chaincode on the Org4 peer. Set the following environment variables to operate as the Org4 admin and target the Org4 peer, peer0.org4.example.com.

export CORE_PEER_LOCALMSPID="Org4MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org4.example.com/peers/peer0.org4.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org4.example.com/users/Admin@org4.example.com/msp
export CORE_PEER_ADDRESS=localhost:10051

peer lifecycle chaincode install basic.tar.gz


We can now install the chaincode on the Org5 peer. Set the following environment variables to operate as the Org5 admin and target the Org5 peer, peer0.org5.example.com.

export CORE_PEER_LOCALMSPID="Org5MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org5.example.com/peers/peer0.org5.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org5.example.com/users/Admin@org5.example.com/msp
export CORE_PEER_ADDRESS=localhost:11051

peer lifecycle chaincode install basic.tar.gz



3.10.3 After you install the chaincode package, you need to approve a chaincode definition for your organization. The definition includes the important parameters of chaincode governance such as the name, version, and the chaincode endorsement policy. The set of channel members who need to approve a chaincode before it can be deployed is governed by the Application/Channel/lifecycleEndorsement policy. By default, this policy requires that a majority of channel members need to approve a chaincode before it can be used on a channel.

If an organization has installed the chaincode on their peer, they need to include the packageID in the chaincode definition approved by their organization. You can find the package ID of a chaincode by using the peer lifecycle chaincode queryinstalled command to query your peer.

peer lifecycle chaincode queryinstalled

You should see output similar to the following:

Installed chaincodes on peer:
Package ID: basic_1.0:69de748301770f6ef64b42aa6bb6cb291df20aa39542c3ef94008615704007f3, Label: basic_1.0

We are going to use the package ID when we approve the chaincode, so let’s go ahead and save it as an environment variable. 

export CC_PACKAGE_ID=basic_1.0:69de748301770f6ef64b42aa6bb6cb291df20aa39542c3ef94008615704007f3    note: replace the packege id generated from your network 


Chaincode is approved at the organization level, so the command only needs to target one peer. The approval is distributed to the other peers within the organization using gossip. Therefore, we can use either Org1 or Org2 peers to approve the chaincode in channel1 and either Org3, Org4 or Org5 peers to approve chaincode on channel2. lets's first approve the chaincode on channel 1 by using Org1 peer, to do so lets set the environment variable to Org1 peer

export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051

Approve the chaincode definition using the peer lifecycle chaincode approveformyorg command: note that channel id should be channel1

peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID channel1 --name basic --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem



We still need to approve the chaincode definition in channel1 as Org2. Set the following environment variables to operate as the Org2 admin:

export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:8051


peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID channel1 --name basic --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem



Now let's approve the chaincode on channel2 member organizations. Let's start by setting environment variable as Org3 peer:


export CORE_PEER_LOCALMSPID="Org3MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org3.example.com/users/Admin@org3.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051


peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID channel2 --name basic --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem


Let's approve from Org4 peer:

export CORE_PEER_LOCALMSPID="Org4MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org4.example.com/peers/peer0.org4.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org4.example.com/users/Admin@org4.example.com/msp
export CORE_PEER_ADDRESS=localhost:10051


peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID channel2 --name basic --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem




Let's approve from Org5 peer:

export CORE_PEER_LOCALMSPID="Org5MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org5.example.com/peers/peer0.org5.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org5.example.com/users/Admin@org5.example.com/msp
export CORE_PEER_ADDRESS=localhost:11051


peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID channel2 --name basic --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem



3.10.4 Committing the chaincode definition to the channel

After a sufficient number of organizations have approved a chaincode definition, one organization can commit the chaincode definition to the channel. If a majority of channel members have approved the definition, the commit transaction will be successful and the parameters agreed to in the chaincode definition will be implemented on the channel.

You can use the peer lifecycle chaincode commit command to commit the chaincode definition to the channel. The commit command also needs to be submitted by an organization admin.

Let's first commit the chaincode in channel1. In order to commit we can use either of Org1 or Org2 peers. Let's set the environment variable to Org2 and commit through Org2 peer

export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:8051


You can use the peer lifecycle chaincode commit command to commit the chaincode definition to the channel1

peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID channel1 --name basic --version 1.0 --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:8051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt


Now we can commit the chaincode definition in channel2 using either Org3,Org4 or Org5 peers. Let's set the environment variable to Org4 and commit from this peer:


export CORE_PEER_LOCALMSPID="Org4MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org4.example.com/peers/peer0.org4.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org4.example.com/users/Admin@org4.example.com/msp
export CORE_PEER_ADDRESS=localhost:10051


You can use the peer lifecycle chaincode commit command to commit the chaincode definition to the channel2


peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID channel2 --name basic --version 1.0 --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt --peerAddresses localhost:10051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org4.example.com/peers/peer0.org4.example.com/tls/ca.crt  --peerAddresses localhost:11051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org5.example.com/peers/peer0.org5.example.com/tls/ca.crt


Now we have compeleted commiting the chaincode definitions on both channels. now let's practice by invoking the chaincode from diffrent peers,

3.10.5 Invoke the chincode 

After the chaincode definition has been committed to a channel, the chaincode will start on the peers joined to the channel where the chaincode was installed. The asset-transfer (basic) chaincode is now ready to be invoked by client applications. 

In order to invoke chaincode from a certain peer first you should set the environment variable of that specufuc peer. For example if we want to invoke the chincode in channel1 we can access it only from peers of Org1 and org2. We cannot access or invoke chaincode in channel1 from peers of Org3, Org4 or Org5.

Let's invoke the chincode in channel1 by using Org1 peer. first set Org1 environment variable.


export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051

Use the following command create an initial set of assets on the ledger. Note that the invoke command needs target a sufficient number of peers to meet chaincode endorsement policy.

peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C channel1 -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:8051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"InitLedger","Args":[]}'


We can use a query function to read the set of cars that were created by the chaincode:

peer chaincode query -C channel1 -n basic -c '{"Args":["GetAllAssets"]}'

Use the following command to change the owner of an asset on the ledger by invoking the asset-transfer (basic) chaincode:

peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C channel1 -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:8051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"TransferAsset","Args":["asset6","Christopher"]}'

or you can query the asset-transfer (basic) chaincode for specific asset ("asset6") by using "ReadAsset" function


peer chaincode query -C channel1 -n basic -c '{"Args":["ReadAsset","asset6"]}'



Now et's invoke the chaincode in channel2 by using Org3 peer. first set Org3 environment variable.


export CORE_PEER_LOCALMSPID="Org3MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org3.example.com/users/Admin@org3.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051

Use the following command create an initial set of assets on the ledger. Note that the invoke command needs target a sufficient number of peers to meet chaincode endorsement policy.

peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C channel2 -n basic --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt --peerAddresses localhost:10051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org4.example.com/peers/peer0.org4.example.com/tls/ca.crt  --peerAddresses localhost:11051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org5.example.com/peers/peer0.org5.example.com/tls/ca.crt -c '{"function":"InitLedger","Args":[]}'


We can use a query function to read the set of cars that were created by the chaincode:

peer chaincode query -C channel2 -n basic -c '{"Args":["GetAllAssets"]}'

Use the following command to change the owner of an asset on the ledger by invoking the asset-transfer (basic) chaincode:

peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C channel2 -n basic --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt --peerAddresses localhost:10051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org4.example.com/peers/peer0.org4.example.com/tls/ca.crt  --peerAddresses localhost:11051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org5.example.com/peers/peer0.org5.example.com/tls/ca.crt -c '{"function":"TransferAsset","Args":["asset6","fetulhak"]}'

or you can query the asset-transfer (basic) chaincode for specific asset ("asset6") by using "ReadAsset" function


peer chaincode query -C channel2 -n basic -c '{"Args":["ReadAsset","asset6"]}'





















































docker-compose -f compose/compose-org3.yaml -f compose/podman/podman-compose-org3.yaml up -d



configtxgen -profile ChannelUsingRaft -outputBlock ./channel-artifacts/channel1.block -channelID channel1









export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:8051




basic_1.0:e4de097efb5be42d96aebc4bde18eea848aad0f5453453ba2aad97f2e41e0d57


export CC_PACKAGE_ID=basic_1.0:e4de097efb5be42d96aebc4bde18eea848aad0f5453453ba2aad97f2e41e0d57


peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID channel1 --name basic --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem




peer lifecycle chaincode checkcommitreadiness --channelID channel1 --name basic --version 1.0 --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --output json



peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID channel1 --name basic --version 1.0 --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:8051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt  --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt  --peerAddresses localhost:10051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org4.example.com/peers/peer0.org4.example.com/tls/ca.crt  --peerAddresses localhost:11051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org5.example.com/peers/peer0.org5.example.com/tls/ca.crt



































peer lifecycle chaincode querycommitted --channelID channel1 --name basic --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem








peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C channel1 -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:8051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt  --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt  --peerAddresses localhost:10051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org4.example.com/peers/peer0.org4.example.com/tls/ca.crt  --peerAddresses localhost:11051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org5.example.com/peers/peer0.org5.example.com/tls/ca.crt    -c '{"function":"InitLedger","Args":[]}'



peer chaincode query -C channel1 -n basic -c '{"Args":["GetAllAssets"]}'



peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C channel1 -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:8051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt  --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt  --peerAddresses localhost:10051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org4.example.com/peers/peer0.org4.example.com/tls/ca.crt  --peerAddresses localhost:11051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org5.example.com/peers/peer0.org5.example.com/tls/ca.crt  -c '{"function":"TransferAsset","Args":["asset6","Christopher"]}'




export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org4MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org4.example.com/peers/peer0.org4.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org4.example.com/users/Admin@org4.example.com/msp
export CORE_PEER_ADDRESS=localhost:10051



peer chaincode query -C channel1 -n basic -c '{"Args":["ReadAsset","asset6"]}'




































