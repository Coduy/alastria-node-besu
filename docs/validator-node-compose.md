# Validator Node Installation (with Docker Compose)

1. [Docker and Docker Compose installation](#docker-and-docker-compose-installation)
2. [Clone Repo](#clone-repo)
3. [New Node](#new-node)
   - [New Orion Configuration](#new-orion-configuration)
   - [New Besu Configuration](#new-besu-configuration)
4. [Existing Node](#existing-node)
   - [Orion Configuration](#orion-configuration)
   - [Besu Configuration](#besu-configuration)
5. [Environment Variables](#environment-variables)
6. [Launch Node](#launch-node)
7. [Stop Node](#stop-node)
8. [Request access to the network](#access)

## Docker and Docker Compose installation

Follow this guide: [Docker Intallation](https://docs.docker.com/get-docker/)
Follow this guide: [Docker Compose Intallation](https://docs.docker.com/compose/install/)

## Clone Repo

Clone repo and navigate to [validator compose directory](../compose/validator-node)

```sh
git clone https://github.com/alastria/alastria-node-besu.git
cd alastria-node-besu/compose/validator-node
```

## New Node

### New Orion configuration

Create a password for your orion node private key

```sh
echo passw0rd > keys/orion/passwordFile
```

Generate your orion node key pair. This will prompt you to enter a password, use the same one you used in the passwordFile

```sh
docker container run -v `pwd`/keys/orion:/keys/orion -w /keys/orion -it --rm pegasyseng/orion:1.5.1-SNAPSHOT -g nodekey
```

### New Besu Configuration

Generate your besu node key and place it under [keys/besu](../compose/validator-node/keys/besu)

```sh
docker container run -v `pwd`/keys/besu:/data -w /data -it --rm hyperledger/besu:1.4 --data-path=/data public-key export --to=/data/key.pub
```

Create a new ethereum account (for example, using metamask) and place its private key in the [keys/besu/signer.key](../compose/validator-node/keys/besu/signer.key) file

```sh
echo <signer-account-private-key> > keys/besu/signer.key
```

## Existing Node

### Orion configuration

Enter the password you used for your orion node private key

```sh
echo <your_password> > keys/orion/passwordFile
```

Copy and paste your orion key pair to `keys/orion`

```sh
cp <existing-orion-private-key> keys/orion
cp <existing-orion-public-key> keys/orion
```

TODO: backup orion database (docker-volumes)

### Besu Configuration

Place the private key of your signer account in the `keys/besu/signer.key` file. If you don allready have a signer key, create a new ethereum account (for example, using metamask) and place its private key in the [keys/besu/signer.key](../compose/validator-node/keys/besu/signer.key) file

```sh
echo <signer-account-private-key> > keys/besu/signer.key
```

Copy your existing node key to `keys/besu`

```sh
cp <besu-node-key> keys/besu
```

Create a docker volume named `validator-node_besu-database`

```sh
docker volume create validator-node_besu-database
```

Copy the contents of your existing node database to the newly created volume. For example

```sh
sudo cp -r ~/alastria-red-b/besu-node/data/database/* /var/lib/docker/volumes/validator-node_besu-database/_data
```

## Environment Variables

Set the `BESU_P2P_HOST` environment variable to the public IP address of your node

```sh
export BESU_P2P_HOST=`dig +short myip.opendns.com @resolver1.opendns.com 2>/dev/null || curl -s --retry 2 icanhazip.com`
```

Set the `ORION_NODEURL` environment variable to the public IP address of your node

```sh
export ORION_NODEURL=http://$BESU_P2P_HOST:8080
```

Set the `ACCOUNT_EMAIL` environment variable to your email

```sh
export ACCOUNT_EMAIL=<email@company.com>
```

Set the `NODE_NAME` environment variable to the name you want your node to display in ethstats

```sh
export NODE_NAME=<company-validator-node>
```

## Launch Node

To launch your node, run

```sh
docker-compose up -d
```

(The -d flag is to run in detached mode)

## Stop Node

To stop your node, run

```sh
docker-compose down
```

## <a name="access"></a>Request access to the network

For adding your new validator Node to the Alastria Red B network, please follow this steps:

1. [Get your **enode**](#enode)
2. [Get your **Node Address**](#node_address)
3. [**Request the Registration of your node** in the network to Alastria Besu Core Team](#request_registration)

### <a name="enode"></a>1. Get your enode

```sh
curl -X POST --data '{"jsonrpc":"2.0","method":"net_enode","params":[],"id":1}' http://127.0.0.1:8545
```

> :warning: _Write down this value (it is your **enode**)_

### <a name="node_address"></a>2. Get your Node Address

```sh
$ cd $HOME/alastria-red-b/besu-node
$ docker container run -v `pwd`:`pwd` -w `pwd` -it --rm besu --data-path=data public-key export-address --to=data/nodeAddress
```

> :warning: _Write down this value (it is your **Node Address**)_

### <a name="request_registration"></a>3. Request the registration of your Node

- Follow the [Guide in the Wiki](https://github.com/alastria/alastria-node-besu/wiki#0-permissioning), sending:
  - your **enode** (for registering your Node as a **Whitelisted Node** in the network)
  - your **Node Address** (for voting up your node as a Validator)
