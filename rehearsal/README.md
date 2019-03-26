# IoTeX Rehearsal Manual

## Join MainNet Rehearsal

1. Pull the docker image:

```
docker pull iotex/iotex-core:v0.5.0-rc6
```

If you have problem to pull the image from docker hub, you can also try our mirror image on gcloud
`gcr.io/iotex-servers/iotex-core:v0.5.0-rc6`.

2. Set the environment with the following commands:

```
mkdir -p ~/iotex-var
cd ~/iotex-var

export IOTEX_HOME=$PWD

mkdir -p $IOTEX_HOME/data
mkdir -p $IOTEX_HOME/log
mkdir -p $IOTEX_HOME/etc

curl https://raw.githubusercontent.com/iotexproject/iotex-testnet/master/rehearsal/config.yaml > $IOTEX_HOME/etc/config.yaml
curl https://raw.githubusercontent.com/iotexproject/iotex-testnet/master/rehearsal/genesis.yaml > $IOTEX_HOME/etc/genesis.yaml
```

3. Edit `$IOTEX_HOME/etc/config.yaml`, look for `externalHost` and `producerPrivKey`, replace `[...]` with your external
IP and private key and uncomment the lines.

4. Run the following command to start a node:

```
docker run -d --name iotex-node \
        -p 4689:4689 \
        -p 14014:14014 \
        -p 8080:8080 \
        -v=$IOTEX_HOME/data:/var/data:rw \
        -v=$IOTEX_HOME/log:/var/log:rw \
        -v=$IOTEX_HOME/etc/config.yaml:/etc/iotex/config_override.yaml:ro \
        -v=$IOTEX_HOME/etc/genesis.yaml:/etc/iotex/genesis.yaml:ro \
        iotex/iotex-core:v0.5.0-rc6 \
        iotex-server \
        -config-path=/etc/iotex/config_override.yaml \
        -genesis-path=/etc/iotex/genesis.yaml \
        -plugin=gateway
```

Now your node should be started successfully.

Note that the command above will also make your node be a gateway, which could process API requests from users. If you
don't want to enable this plugin, you could remove two lines from the command above: `-p 14014:14014 \` and
`-plugin=gateway`.

5. Make sure TCP ports 4689, 14014, 8080 are open on your firewall and load balancer (if any).

## <a name="ioctl"/>Interact with MainNet Rehearsal


You can install ioctl (a command-line interface for interacting with IoTeX blockchain)

```
curl https://raw.githubusercontent.com/iotexproject/iotex-core/master/install-cli.sh | sh
```

Make sure ioctl is pointed to the mainnet rehearsal endpoint:
```
ioctl config set endpoint api.iotex.one:80
```

## Misc

For the remaining information, it will be the same as testnet before. Please refer to the document [here](../README.md).

For those who has customized deployment process already, you don't need to exactly follow the instructions above. However,
please make sure you will use `iotex/iotex-core:v0.5.0-rc6` and the latest config file `rehearsal/config.yaml` and genesis
file `rehearsal/genesis.yaml`.