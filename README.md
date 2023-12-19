# RACE Quickstart (WIP)

## Pre-Release Preliminaries
- Create a personal access token (classic) for your github account (Upper-right profile button -> Settings -> Developer Settings -> Personal access tokens -> Tokens (classic) -> Generate new token)
- The token _needs_ *repo* and *read:packages* permissions
- Copy and save the generated TOKEN somewhere, it will be needed later.
- _Note_ there will be more explanatory text around the following sections

## Environment Setup
Download and run the Race-in-the-Box (RIB) entrypoint script, which automatically pulls a docker image for orchestrating running local tests of RACE networks. The rest of these instructions assume you may want to extend to a more extensive exploration of RACE and so create a local directory structure for that purpose.

```
mkdir race-code
cd race-code
curl https://raw.githubusercontent.com/tst-race/race-in-the-box/2.6.0-beta-1/entrypoints/rib_2.6.0.sh?token=GHSAT0AAAAAACDMSZOK4VHZTU6EFRA7YR7EZMBYNAQ -o rib_2.6.0.sh
bash rib_2.6.0.sh --version=2.6.0-beta-1 --ui
```

This command will pull and run a docker image to give you an interactive commandline prompt for the RIB test environment.

Special pre-release step:
```
rib github config --access-token<TOKEN from above>
```

## First Deployment
The first deployment will be a small deployment with 4 linux clients, 6 linux servers, and an optional Android client, and uses development exemplar versions of comms and network-manager plugins. 

```
rib deployment local create --name=basic \
    --linux-client-count=4 \
    --linux-server-count=6 \
    --race-node-arch=x86_64 \
    --android-client-count=1 \
    --android-client-bridge-count=1 \
    --race-core=tag=https://github.com/tst-race/race-core/releases/tag/2.6.0-beta-1 \
    --linux-client-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
    --linux-server-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
    --registry-client-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
    --android-client-image=ghcr.io/tst-race/race-images/race-runtime-android-x86_64:main

rib-use local basic
deployment up
```

_Note:_ Running *up* the first time will take some time because this is when docker container images are pulled.

_If_ you are including a physical android device in the deployment, then do the following:
1. Ensure the phone is on the same LAN as your main computer (the phone will connect to a docker container running there).
2. Plug the phone into the main computer and check that *adb devices* shows the device [Instructions for troubleshooting].
3. Run these additional steps (_after_ *up* has finished)
```
deployment bridged android prepare --persona=race-client-00005
deployment bridged android connect
```

Now the deployment should be ready to be _started_:
```
deployment start
```

Once _start_ is finished, you can send messages among clients:
```
deployment message send-manual --sender=race-client-00001 --recipient=race-client-00002 --message="Hello!"
```

You can view the status of message that have been sent by running:
```
deployment message list
```

You can change the sender and recipient, or leave either blank to cause messages to be sent _from_ all nodes (if sender is omitted) or _to_ all nodes (if receiver is omitted).

## Really Anonymous Messaging
There are two different implementations of anonymous message routing across a resilient overlay network: PRISM and CARMA. We can easily run either of these instead of the stand-in used above by creating a new deployment and providing a _--network-manager-kit__ argument (last argument in the command below).

### PRISM
```
rib deployment local create --name=prism \
    --linux-client-count=4 \
    --linux-server-count=10 \
    --race-node-arch=x86_64 \
    --android-client-count=1 \
    --android-client-bridge-count=1 \
    --race-core=tag=https://github.com/tst-race/race-core/releases/tag/2.6.0-beta-1 \
    --linux-client-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
    --linux-server-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
    --registry-client-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
    --android-client-image=ghcr.io/tst-race/race-images/race-runtime-android-x86_64:main \
    --network-manager-kit=tag=https://github.com/tst-race/race-prism/releases/tag/2.6.0-beta-1/

rib-use local prism
deployment up

# Optional commands if you are using a physical android client
deployment bridged android prepare --persona=race-client-00005
deployment bridged android connect
# End optional commands

deployment start
```

Prism has adaptive organization capability which means its node need a few moments to "get organized" after it is started before messages will be able to be routed.

### CARMA

```
rib deployment local create --name=carma \
    --linux-client-count=4 \
    --linux-server-count=20 \
    --race-node-arch=x86_64 \
    --android-client-count=1 \
    --android-client-bridge-count=1 \
    --race-core=tag=https://github.com/tst-race/race-core/releases/tag/2.6.0-beta-1 \
    --linux-client-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
    --linux-server-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
    --registry-client-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
    --android-client-image=ghcr.io/tst-race/race-images/race-runtime-android-x86_64:main \
    --network-manager-kit=tag=https://github.com/tst-race/race-carma/releases/tag/2.6.0-beta-1/

rib-use local carma
deployment up

# Optional commands if you are using a physical android client
deployment bridged android prepare --persona=race-client-00005
deployment bridged android connect
# End optional commands

deployment start
```

Carma has a notion of a "batch" of messages that are processed simultaneously to mitigate using timing-analyses to deanonymize messaging. Therefore, you will need to send multiple messages (generally at least 4) before the system will route them. This if you send a single message, it may never arrive until you send more, but it _is_ being stored in the server overlay awaiting more messages to be mixed with, so it will not be lost.

