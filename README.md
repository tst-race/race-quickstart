# RACE Quickstart (WIP)

```
## Pre-Release Preliminaries
- Create a personal access token (classic) for your github account (Upper-right profile button -> Settings -> Developer Settings -> Personal access tokens -> Tokens (classic) -> Generate new token)
- The token _needs_ *repo* and *read:packages* permissions
- Copy and save the generated TOKEN somewhere, it will be needed later.
- *Note* there will be more explanatory text around the following sections
```

<!-- toc -->

- [What is RACE?](#what-is-race)
- [What is this Guide?](#what-is-this-guide)
- [Environment Setup](#environment-setup)
- [First Deployment](#first-deployment)
- [Really Anonymous Messaging](#really-anonymous-messaging)
- [Really Resilient Networking](#really-resilient-networking)
- [Exploring Your Deployment](#exploring-your-deployment)
- [Exploring More of RACE](#exploring-more-of-race)
- [Clean Up: Removing RACE/RiB](#clean-up-removing-racerib)

<!-- tocstop -->

Welcome to the quickstart guide for trying out the Resilient Anonymous Communications for Everyone (RACE) project software!


## What is RACE?
RACE is an open source project aimed at developing technologies to provide metadata-anonymous, secure, and resilient messaging for users around the world. RACE provides anonymity by routing messages through an overlay network of volunteer servers using cryptographic algorithms that prevent a malicious subset of these servers from determining who is messaging whom. RACE uses specialized networking protocols to prevent connections between individual members of the network from being detected or blocked. RACE is built to run in a dockerized linux environment and on Android devices.

## What is this Guide?
This is quick start guide for running a small network of RACE nodes on a personal laptop or server. Running the below commands will download prebuilt RACE code and docker images to run a self-contained overlay network of RACE servers and clients for trying out the RACE software, with the option to include Android clients (either as emulators or physical devices). The network of RACE nodes created (called a _deployment_) is _not_ a simulation, it is a set of docker containers running the actual RACE software; all the nodes just happen to be containers running on a single host. The same software would be run on separate physical machines in a full-scale deployment.

## Environment Setup
Download and run the Race-in-the-Box (RIB) entrypoint script. This automatically pulls a docker image for orchestrating running local tests of RACE networks. 

```
mkdir race-code
cd race-code
curl https://raw.githubusercontent.com/tst-race/race-in-the-box/2.6.0-beta-1/entrypoints/rib_2.6.0.sh?token=GHSAT0AAAAAACDMSZOK4VHZTU6EFRA7YR7EZMBYNAQ -o rib_2.6.0.sh
bash rib_2.6.0.sh --version=2.6.0-beta-1 --ui
```

That command pulled and ran a docker image to give you an interactive commandline prompt for the RIB test environment. All the commands below are expected to be run on that prompt inside the RIB container.

Special pre-release step:
```
rib github config --access-token=<TOKEN from above>
```

### What is Race-in-the-Box (RIB)?
RIB is an orchestration framework for making it easy to configure and test different RACE network deployments. RIB manages fetching prebuilt code from repositories, configuring docker containers and networks, creating configuration files, and automating client messaging behavior to facilitate performance and security testing. RIB is _not_ intended to be used for instantiating and running a real-world RACE deployment.


## First Deployment
The first deployment will be a small deployment with 4 linux clients, 6 linux servers, and an optional Android client, and uses development exemplar versions of comms and network-manager plugins. 

### Creating the Deployment
Running the following commands will pull down prebuilt RACE software, use them to generate initial configuration files, and fetch and run a set of docker containers to run the individual RACE nodes.

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

_____Note:_____ Running _up_ the first time will take some time because this is when docker container images are pulled. 


### Android Bridge Device
_If_ you are including a physical android device in the deployment, then do the following:
1. Ensure the phone is on the same LAN as your main computer (the phone will connect to a docker container running there).
2. Plug the phone into the main computer and check that `adb devices` shows the device [Instructions for troubleshooting].
3. Run these additional steps (_after_ `up` has finished)

```
deployment bridged android prepare --persona=race-client-00005
deployment bridged android connect
```

___Note:___ If you are having problems, with the _connect_ step failing or hanging forever, it is very likely a network issue. We suggest continuing without the Android client for now, and digging further later using our [Android bridged deployment troubleshooting tips]().

### Starting the Deployment 
Now the deployment should be ready to be _started_:
```
deployment start
```

### Using the Deployment
Once _start_ is finished, you can send messages among clients:
```
deployment message send-manual --sender=race-client-00001 --recipient=race-client-00002 --message="Hello!"
```

You can change the sender and recipient, or leave either blank to cause messages to be sent _from_ all nodes (if sender is omitted) or _to_ all nodes (if receiver is omitted). If you are using an Android client, you can also manually send messages in the UI and look at received messages.

You can view the status of messages that have been sent on the commandline by running:
```
deployment message list
```



#### Network Visualization
RIB also have a rudimentary network visualization capability. First run:
```
/race_in_the_box/scripts/voa/inspect_links.py
```

This pulls data about the current network connectivity graph and processes it into a graph structure that is pushed to a simple graph visualization server. Browse to [http://localhost:6080] and a force-directed view of the graph is shown.

You can also visualize the paths involved in sending a specific message by running:
```
/race_in_the_box/scripts/voa/pollmsg.py
```
Navigate to one of the "Force URL" links to see a visualization of the network, with links highlighted that were used in transmission of this particular message.


#### Jaeger Tracing Visualization
Additionally, RACE messages are instrumented via OpenTracing for testing purposes. Using the output of the `pollmsg.py` command, above, you can one of the "Jaeger URL" links to see the distinct hops of the message through the series of nodes in the path. Within Jaeger, you can also see all the traces that each node is involved in, which can include events beyond just routing client messages.

### Stopping the Deployment
When you are done testing with a deployment, you should _stop_, (_disconnect_ and then _unprepare_ if you are using a bridged Android device) then _down_.

```
deployment stop

deployment bridged android disconnect
deployment bridged android unprepare

deployment down
```


## Really Anonymous Messaging
There are two different implementations of anonymous message routing across a resilient overlay network: Prism and Carma. We can easily run either of these instead of the stand-in used above by creating a new deployment and providing a _--network-manager-kit_ argument (last argument in the command below).

### Prism
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
    --network-manager-kit=tag=2.6.0-beta-2,org=tst-race,repo=race-prism

rib-use local prism
deployment up

# Optional commands if you are using a physical android client
deployment bridged android prepare --persona=race-client-00005
deployment bridged android connect
# End optional commands

deployment start
```

[Prism]() has adaptive organization capability which means its node need a few moments to "get organized" after it is started before messages will be able to be routed.

When you are done testing, use the following commands to end the deployment.

```
deployment stop

deployment bridged android disconnect
deployment bridged android unprepare

deployment down
```

### Carma

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

[Carma]() has a notion of a "batch" of messages that are processed simultaneously to mitigate using timing-analyses to deanonymize messaging. Therefore, you will need to send multiple messages ___(generally at least 4)___ before the system will route them. Thus, if you send a single message it may never arrive until you send more, but it _is_ being stored in the server overlay awaiting more messages to be mixed with, so it will not be lost.


When you are done testing, use the following commands to end the deployment.

```
deployment stop

deployment bridged android disconnect
deployment bridged android unprepare

deployment down
```

## Really Resilient Networking
Up to this point we have been using stand-in channels for communications between nodes. Specifically, servers have been using a simple TCP socket to talk to each other, and clients and servers have been posting data to each other via a redis intermediary. In both cases, the messages have just been raw encrypted bytes and presumably easy to filter or block. Now we add in covert channels to make it impossible to detect or block the RACE system's network traffic. 


### Carma with Obfs and ssEmail

This deployment is the same as the Carma deployment above, but now we have added arguments to include [Obfs]() and [ssEmail]() for communications. Carma chooses how and where to use these channels - in this case it will use Obfs for server-to-server traffic since it is high-bandwidth and low-latency, but requires revealing the IP address of the other endpoint.

Carma will use ssEmail for client-server traffic: ssEmail transmits data by encoding it into the semantic space of an image (i.e. it generates wholly new images, rather than altering existing ones) and then attaching the image to an email sent to another node. This prevents either node from learning the IP address of the other, they just know one another's email addresses. In addition to additional processing time to encode and decode images, the ssEmail channel also limits the rate at which emails are sent to avoid standing out. In short, the messages will take longer to be received but the whole system will be stealthy.

```
rib deployment local create --name=carma-obfs-ssEmail \
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
    
rib-use local carma-obfs-ssEmail
deployment up

# Optional commands if you are using a physical android client
deployment bridged android prepare --persona=race-client-00005
deployment bridged android connect
# End optional commands

deployment start
```

### Prism with Snowflake, destiniPixelfed, and ssEmail

This deployment is the same as the Prism deployment above, but now using [Snowflake](), [ssEmail](), and [destiniPixelfed]() for communications. We will demonstrate the use of some Prism-specific configuration arguments to tell it _how_ we think it should use these channels. In particular, ssEmail and destiniPixelfed are two different _indirect_ channels, meaning they use a third-party service as an intermediary for sending messages. ssEmail is described above; destiniPixelfed also uses image steganography but of a different type (it encodes messages into existing images) and transmits the image by posting it to a mock "whiteboard" service, meant to stand-in for the myriad social media and file drop services that exist on the internet. Our configuration arguments will tell Prism which types of messages to send on each channel to optimize its performance. If left to defaults, Prism will randomly load-balance messaging between the two.

```
rib deployment local create --name=prism-snowflake-destiniPixelfed-ssEmail \
    --linux-client-count=4 \
    --linux-server-count=10 \
    --race-node-arch=x86_64 \
    --android-client-count=1 \
    --android-client-bridge-count=1 \
    --race-core=local=/code/race-core/kits \
    --linux-client-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
    --linux-server-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
    --registry-client-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
    --android-client-image=ghcr.io/tst-race/race-images/race-runtime-android-x86_64:main \
    --network-manager-kit=tag=2.6.0-beta-2,org=tst-race,repo=race-prism \
    --comms-channel=snowflake --comms-kit=tag=2.6.0-beta-1,org=tst-race,repo=race-snowflake \
    --comms-channel=ssEmail --comms-kit=tag=2.6.0-beta-2,org=tst-race,repo=race-semanticsteg \
    --comms-channel=destiniPixelfed --comms-kit=tag=2.6.0-beta-2,org=tst-race,repo=race-destini,asset=race-destini-pixelfed.tar.gz

rib-use local prism-snowflake-destiniPixelfed-ssEmail
```

#### Custom configuration Argument
As described above, we will specify a custom argument to Prism's config generation logic to specify how we want specific channels used.

```
deployment config generate --network-manager-custom-args="-TssEmail=uplink,epoch -TdestiniPixelfed=ark,downlink" --force

deployment up

# Optional commands if you are using a physical android client
deployment bridged android prepare --persona=race-client-00005
deployment bridged android connect
# End optional commands

deployment start
```

___Note:___ Prism's self-organizing behavior takes a bit longer when using stealthy channels, so give it a few minutes before expecting messages to get through.


## Exploring Your Deployment


## Exploring More of RACE
This quick start guide got you up and running with core RACE system, both anonymous routing network-manager plugins, and a subset of the comms plugins that provide resilience at the network-layer. From here you can explore a number of different aspects of RACE in more depth:

- [What is RACE: the longer story]()
- [Full list of RACE Comms Plugins]()
- [Bootstrapping: how do new clients join?](https://github.com/tst-race/race-in-the-box/blob/2.6.0/documentation/how-to/node-bootstrap-test.md)
- [Developer Documentation]()


## Clean Up: Removing RACE/RiB
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
___Note:___ <NAME> can be the name of any existing deployment (e.g. `basic`)

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
rm -rf ~/.race
```
