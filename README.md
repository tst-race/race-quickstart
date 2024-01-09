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
    --race-core=tag=https://github.com/tst-race/race-core/releases/tag/2.6.0-beta-2 \
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

_Note_: there is a "test-id" that is prepended to messages sent via these commands to assist in performance testing. This will show up in the Android client as "default" prepended to messages (e.g. sending the above --message="Hello!" will result in "default Hello!" appearing in the app UI).

You can change the sender and recipient, or leave either blank to cause messages to be sent _from_ all nodes (if sender is omitted) or _to_ all nodes (if receiver is omitted).

When you are done testing with a deployment, you should _stop_, (_disconnect_ and then _unprepare_ if you are using a bridged Android device) then _down_.

```
deployment stop

deployment bridged android disconnect
deployment bridged android unprepare

deployment down
```

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
    --race-core=tag=https://github.com/tst-race/race-core/releases/tag/2.6.0-beta-2 \
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

When you are done testing, use the following commands to end the deployment.

```
deployment stop

deployment bridged android disconnect
deployment bridged android unprepare

deployment down
```

### CARMA

```
rib deployment local create --name=carma \
    --linux-client-count=4 \
    --linux-server-count=20 \
    --race-node-arch=x86_64 \
    --android-client-count=1 \
    --android-client-bridge-count=1 \
    --race-core=tag=https://github.com/tst-race/race-core/releases/tag/2.6.0-beta-2 \
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


When you are done testing, use the following commands to end the deployment.

```
deployment stop

deployment bridged android disconnect
deployment bridged android unprepare

deployment down
```

## Really Covert Network
Up to this point we have been using stand-in channels for communications between nodes. Specifically, servers have been using a simple TCP socket to talk to each other, and clients and servers have been posting data to each other via a redis intermediary. In both cases, the messages have just been raw encrypted bytes and presumably easy to filter or block. Now we add in covert channels to make it impossible to detect or block the RACE system's network traffic. 


### Carma with Obfs and ssEmail

This deployment is the same as the Carma deployment above, but now we have added arguments to include Obfs and ssEmail for communications. Carma chooses how and where to use these channels - in this case it will use Obfs for server-to-server traffic since it is high-bandwidth and low-latency, but requires revealing the IP address of the other endpoint.

Carma will use ssEmail for client-server traffic: ssEmail transmits data by encoding it into the semantic space of an image (i.e. it generates wholly new images, rather than altering existing ones) and then attaching the image to an email sent to another node. This prevents either node from learning the IP address of the other, they just know one another's email addresses. In addition to additional processing time to encode and decode images, the ssEmail channel also limits the rate at which emails are sent to avoid standing out. In short, the messages will take longer to be received but the whole system will be stealthy.

```
rib deployment local create --name=carma-ta2s \
    --linux-client-count=4 \
    --linux-server-count=20 \
    --race-node-arch=x86_64 \
    --android-client-count=1 \
    --android-client-bridge-count=1 \
    --race-core=tag=https://github.com/tst-race/race-core/releases/tag/2.6.0-beta-2 \
    --linux-client-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
    --linux-server-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
    --registry-client-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
    --android-client-image=ghcr.io/tst-race/race-images/race-runtime-android-x86_64:main \
    --network-manager-kit=tag=https://github.com/tst-race/race-carma/releases/tag/2.6.0-beta-1/ \
    --comms-channel=obfs --comms-kit=tag=2.6.0-beta-1,repo=race-obfs,org=tst-race \
    --comms-channel=ssEmail --comms-kit=tag=2.6.0-beta-1,repo=race-semanticsteg,org=tst-race
    
rib-use local carma
deployment up

# Optional commands if you are using a physical android client
deployment bridged android prepare --persona=race-client-00005
deployment bridged android connect
# End optional commands

deployment start
```



## Removing RACE/RiB
This sequence of steps should remove everything you may have just pulled and installed as part of this quickstart. 

### Down Deployments
To check if you still have a deployment running, run:
```
rib deployment local active
```
This will output the name of an active deployment; if there is one, then run:
```
deployment local stop --name=<NAME> --force
deployment local down --name=<NAME> --force
```

You can verify it has been brought down by running the active command again.


### Android Device
If you did not use an Android device you can skip this.

Plug the device into the RiB host so ADB can detect it, then run:
```
deployment local bridged android unprepare --name=<NAME>
```
_Note_: <NAME> can be the name of any existing deployment

### Remove Docker Images
Exit the RiB container, then run the following to delete all RACE-related images:

This command finds and deletes all images from _tst-race_, the Github organization hosting the RACE code.
```
docker rmi $(docker images | grep tst-race | tr -s [:space:] | cut -d' ' -f3)
```

Additionally, RiB and RACE uses a set of third-party docker images that you _may_ have already had installed for other reasons. If you want to remove these as well, do:
```
docker rmi defreitas/dns-proxy-server:latest
docker rmi jaegertracing/jaeger-query:1.34.1
docker rmi jaegertracing/jaeger-collector:1.34.1
docker rmi docker.elastic.co/elasticsearch/elasticsearch:7.16.1
docker rmi docker.elastic.co/kibana/kibana:7.16.1
docker rmi ixdotai/openvpn:0.2.3
docker rmi instrumentisto/coturn:latest
docker rmi redis:6.0.6
```

Lastly, RiB creates a network for all the containers to connect to, this can be removed with:
```
docker network rm rib-overlay-network
```

### Remove RiB Files
When the entrypoint script (`rib_2.6.0.sh`) is run, a directory is created for storing RACE plugin code and deployments. By default this is created at ~/.race, unless the `--rib_state_path` argument is passed to set a different location. Removing this directory will remove all RACE content aside from the docker images previously removed.

```
docker rm -rf ~/.race
```
