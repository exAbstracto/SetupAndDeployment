# SetupAndDeployment
Setup and deployment project


cd ~/fabric-tools
./stopFabric.sh
./teardownFabric.sh

docker kill $(docker ps -q)
docker rm $(docker ps -aq)
docker rmi $(docker images dev-* -q)


mkdir HyperledgerPlatform

cd HyperledgerPlat


// Install Prereq  cryptogen, configtxgen, configtxlator, and peer

curl -sSL https://goo.gl/byy2Qj | bash -s 1.0.5

export PATH=~/HyperledgerPlatform/bin:$PATH

1. Generate the network content

cd /home/tibi/MediBlox-Chaincode/medichain/Deployment
./build_medichain.sh -m generate
./build_medichain.sh -m up -s couchdb -a 


2. Delete any business network cards that may exist in your wallet. It is safe to ignore any errors that state that the business network cards cannot be found:

composer card delete -n PeerAdmin@medichain-org1-only
composer card delete -n PeerAdmin@medichain-org1
composer card delete -n PeerAdmin@medichain-org2-only
composer card delete -n PeerAdmin@medichain-org2
composer card delete -n tibi@medichain
composer card delete -n bogdan@medichain
composer card delete -n admin@medichain
composer card delete -n PeerAdmin@fabric-network

3. Generate the certificates for Admins for both organisations

chmod u+x crypto-config/ordererOrganizations/plurisense.com/orderers/orderer.plurisense.com/tls/ca.crt
chmod u+x crypto-config/peerOrganizations/org1.plurisense.com/peers/peer0.org1.plurisense.com/tls/ca.crt
chmod u+x crypto-config/peerOrganizations/org2.plurisense.com/peers/peer0.org2.plurisense.com/tls/ca.crt


Users

The organization Org1 is configured with a user named Admin@org1.plurisense.com. This user is an administrator.

The user Admin@org1.plurisense.com has a set of certificates and private key files stored in the directory:


crypto-config/peerOrganizations/org1.plurisense.com/users/Admin@org1.plurisense.com/msp

The organization Org2 is configured with a user named Admin@org2.plurisense.com. This user is an administrator.

The user Admin@org2.plurisense.com has a set of certificates and private key files stored in the directory:

crypto-config/peerOrganizations/org2.plurisense.com/users/Admin@org2.plurisense.com/msp

Some of these files will be used later on to interact with the Hyperledger Fabric network.

In addition to the administrator, the CAs (Certificate Authorities) for Org1 and Org2 have been configured with a default user. 
This default user has an enrolment ID of admin and an enrolment secret of adminpw. However, 
this user does not have permission to deploy a blockchain business network.

4. Run the composer card create command to create a business network card using the connection profile that just contains the peers for Org1. 
   You must specify the path to all three files that you either created or located in the previous steps: (note: the sk file will differ.)

composer card create -p connection-org1-only.json -u PeerAdmin -c Admin@org1.plurisense.com-cert.pem -k f0404fd32dc06b88f5dec219efe55ad04ad3d97c514c9e6e5ca4bd014c190c0d_sk -r PeerAdmin -r ChannelAdmin
composer card create -p connection-org1.json -u PeerAdmin -c Admin@org1.plurisense.com-cert.pem -k f0404fd32dc06b88f5dec219efe55ad04ad3d97c514c9e6e5ca4bd014c190c0d_sk -r PeerAdmin -r ChannelAdmin

composer card create -p connection-org2-only.json -u PeerAdmin -c Admin@org2.plurisense.com-cert.pem -k 1a699b417933897499ee8570ec40ee3da6e15116bb7a1d054ce7f4e29bf928e2_sk -r PeerAdmin -r ChannelAdmin
composer card create -p connection-org2.json -u PeerAdmin -c Admin@org2.plurisense.com-cert.pem -k 1a699b417933897499ee8570ec40ee3da6e15116bb7a1d054ce7f4e29bf928e2_sk -r PeerAdmin -r ChannelAdmin

5. Importing the business network cards for the Hyperledger Fabric administrator for Org1 and then for Org2

composer card import -f PeerAdmin@medichain-org1-only.card
composer card import -f PeerAdmin@medichain-org1.card

composer card import -f PeerAdmin@medichain-org2-only.card
composer card import -f PeerAdmin@medichain-org2.card

6. Installing the Hyperledger Composer runtime onto the Hyperledger Fabric peer nodes for Org 1 and then for Org2

composer runtime install -c PeerAdmin@medichain-org1-only -n medichain
composer runtime install -c PeerAdmin@medichain-org2-only -n medichain

7. Retrieving business network administrator certificates for Org1

composer identity request -c PeerAdmin@medichain-org1-only -u admin -s adminpw -d adminorg1
composer identity request -c PeerAdmin@medichain-org2-only -u admin -s adminpw -d adminorg2



8. Starting the business network

composer network start -c PeerAdmin@medichain-org1 -a medichain@0.0.20.bna -o endorsementPolicyFile=endorsement-policy.json -A adminorg1 -C adminorg1/admin-pub.pem -A adminorg2 -C adminorg2/admin-pub.pem


9. Creating a business network card to access the business network as Org1 and one to access the business network as Org2

-- org1

composer card create -p connection-org1.json -u adminorg1 -n medichain -c adminorg1/admin-pub.pem -k adminorg1/admin-priv.pem
composer card import -f  adminorg1@medichain.card
composer network ping -c adminorg1@medichain

-- org2
composer card create -p connection-org2.json -u adminorg2 -n medichain -c adminorg2/admin-pub.pem -k adminorg2/admin-priv.pem
composer card import -f  adminorg2@medichain.card
composer network ping -c adminorg2@medichain.card






