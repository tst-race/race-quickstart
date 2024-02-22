# RACE Quickstart
Welcome to the quickstart guide for trying out the Resilient Anonymous Communications for Everyone (RACE) project software!


<!-- toc -->

- [What is RACE?](#what-is-race)
- [What is Race-in-the-Box (RIB)?](#what-is-race-in-the-box-rib)
- [What is this Guide?](#what-is-this-guide)
- [Resource and Environment Requirements](#reseource-and-environment-requirements)
- [Environment Setup](#environment-setup)
- [First Deployment](#first-deployment)
- [Really Anonymous Messaging](#really-anonymous-messaging)
- [Really Resilient Networking](#really-resilient-networking)
- [Exploring Your Deployment](#exploring-your-deployment)
- [Exploring More of RACE](#exploring-more-of-race)
- [Clean Up: Removing RACE/RiB](#clean-up-removing-racerib)

<!-- tocstop -->



## What is RACE?
RACE is an open source project aimed at developing technologies to provide metadata-anonymous, secure, and resilient messaging for users around the world. RACE provides anonymity by routing messages through an overlay network of volunteer servers using cryptographic algorithms that prevent a malicious subset of these servers from determining who is messaging whom. RACE uses specialized networking protocols to prevent connections between individual members of the network from being detected or blocked. RACE is built to run in a dockerized linux environment and on Android devices. For more details on the concept, see [What is RACE: The Longer Story](https://github.com/tst-race/race-docs/blob/main/what-is-race.md).


## What is Race-in-the-Box (RIB)?
RIB is an orchestration framework for making it easy to configure and test different RACE network deployments. RIB manages fetching prebuilt code from repositories, configuring docker containers and networks, creating configuration files, and automating client messaging behavior to facilitate performance and security testing. RIB is _not_ intended to be used for instantiating and running a real-world RACE deployment.

## What is this Guide?
This is quick start guide for running a small network of RACE nodes on a personal laptop or server. Running the below commands will download prebuilt RACE code and docker images to run a self-contained overlay network of RACE servers and clients for trying out the RACE software, with the option to include Android clients (either as emulators or physical devices). The network of RACE nodes created (called a _deployment_) is _not_ a simulation, it is a set of docker containers running the actual RACE software; all the nodes just happen to be containers running on a single host. The same software would be run on separate physical machines in a full-scale deployment.

## Resource and Environment Requirements

#### At-a-Glance:
* __CPU:__ 6+ Cores
* __RAM:__ 32GB+
* __Storage:__ 40GB+
* __Software:__
  - [Docker](https://docs.docker.com/get-docker/)
  - (optional) [Android Debug Bridge (ADB)](https://www.xda-developers.com/install-adb-windows-macos-linux/)
  - (optional) Android Device (tested on Android-10/SDK-29)

This guide has been tested on Linux, MacOS laptops (including Apple Silicon), and Windows laptops using Windows Subsystem for Linux 2.0 (Ubuntu on Windows) with 32GB of RAM and 6-Core CPUs. If trying to run on lower-resource devices (particularly RAM) some of these test deployments may not run. This is because these test deployments are running _all_ RACE nodes on the host, and does not reflect the resource requirements of a real-world RACE deployment where only a single node would be running on any given host. 

The host system will need about 40GB of free storage for the docker images, software, and deployment logs that are used in this guide.

If using an Android device, the device must be able to successfully connect to your computer (e.g. `adb devices` shows its serial number).


## Environment Setup
Download and run the Race-in-the-Box (RIB) entrypoint script. This automatically pulls a docker image for orchestrating running local tests of RACE networks. 

```
mkdir race-code
cd race-code
curl https://raw.githubusercontent.com/tst-race/race-in-the-box/2.6.0-v1/entrypoints/rib_2.6.0.sh -o rib_2.6.0.sh
bash rib_2.6.0.sh
```

That command pulled and ran a docker image to give you an interactive commandline prompt for the RIB test environment. All the commands below are expected to be run on that prompt inside the RIB container.



## First Deployment
The first deployment will be a small deployment with 4 linux clients, 6 linux servers, and an optional Android client (___if ARE using an android device, switch to [this guide](README.md#first-deployment)___), and uses development exemplar versions of comms and network-manager plugins. 

### Creating the Deployment
Running the following commands will pull down prebuilt RACE software, use them to generate initial configuration files, and fetch and run a set of docker containers to run the individual RACE nodes.

<details>

<summary>What are Configuration Files?</summary>
RACE uses a notion of an initial (or _genesis_) configuration for the entire network that is expressed as a set of coordinated per-node config files. Examples of data in these per-node configs include cryptographic keys and addresses for initial node-to-node connections. RIB automates the creation of these per-node configs by generating or ingesting a [range-config](https://github.com/tst-race/race-in-the-box/blob/2.6.0/documentation/files-images-templates/example-range-config.json) file which represents the network environment (e.g. RACE nodes, their IPs, etc.) and running a [config generation pipeline](https://github.com/tst-race/race-in-the-box/blob/2.6.0/documentation/how-to/deployment-setup/configuration-generation.md). While RIB is intended for orchestrating tests and demonstrations, this aspect of configuration generation could be applicable to assisting construction of real-world RACE deployments.

</details>


```
rib deployment local create --name=basic \
    --linux-client-count=4 \
    --linux-server-count=6 \
    --linux-client-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
    --linux-server-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
    --race-core=tag=https://github.com/tst-race/race-core/releases/tag/2.6.0-v1

rib-use local basic

# This command sets a "profile" for RIB operations to save you entering the deployment name and type each time
# "local" means a local deployment contained on a single host, as-opposed to an AWS-based deployment
# "basic" is just the name of this particular deployment, as chosen in the create command above

deployment up
```

<details>
<summary>Example output</summary>


```
```

</details>

_____Note:_____ Running _up_ the first time will take some time because this is when docker container images are pulled. Most of this is the RACE runtime-linux image which is about 16GB, you can manually pull this outside of RIB to more closely monitor its progress with:

```
docker pull ghcr.io/tst-race/race-images/race-runtime-linux:main
```


### Starting the Deployment 
Now the deployment should be ready to be _started_:
```
deployment start
```

<details>
<summary>Example output</summary>

```
```

</details>

___Note:___ If the deployment fails to start, there are a few things to check first:
1. Do you have sufficient storage (at least 10GB free after `create` and `up` have finished)?
2. Is your architecture correctly detected? Run `env | grep HOST_ARCHITECTURE` in the RIB terminal, it should read x86_64 or arm64 depending on your CPU architecture.



### Using the Deployment
Once _start_ is finished, you can send messages among clients:
```
deployment message send-manual --sender=race-client-00001 --recipient=race-client-00002 --message="Hello!"
```

<details>
<summary>Example output</summary>

```
202) rib:development:local:basic@code# deployment message send-manual --sender=race-client-00001 --recipient=race-client-00002 --message="Hello!"
Sending Messages
Sent Messages To Deployment Nodes
Manual Message Send Complete
```

</details>

You can change the sender and recipient, or leave either blank to cause messages to be sent _from_ all nodes (if sender is omitted) or _to_ all nodes (if receiver is omitted):

```
deployment message send-manual --message="Hello to all and from all!"
```

You can view the status of messages that have been sent on the commandline by running:
```
deployment message list
```

<details>
<summary>Example output</summary>

```
202) rib:development:local:basic@code# deployment message list
+------------------+---------+------+-------------------+-------------------+----------+---------------------+---------------------+----------------+
|     Trace ID     | Test ID | Size |       Sender      |     Recipient     |  Status  |      Start Time     |       End Time      | Total Time (s) |
+------------------+---------+------+-------------------+-------------------+----------+---------------------+---------------------+----------------+
| de49144dc5ef7caf | default |  14  | race-client-00001 | race-client-00002 | received | 2024-01-17 15:29:41 | 2024-01-17 15:29:42 |    0.857001    |
+------------------+---------+------+-------------------+-------------------+----------+---------------------+---------------------+----------------+
```

</details>


#### Network Visualization
RIB also have a rudimentary network visualization capability. First run:
```
inspect_links.py | wc -l
```


This pulls data about the current network connectivity graph and processes it into a graph structure that is pushed to a simple graph visualization server. Browse to [http://localhost:6080]([http://localhost:6080]) and a force-directed view of the graph is shown.

![An image of a force-directed graph](https://github.com/tst-race/race-quickstart/blob/local-wip/network-vis-1.png?raw=true)

You can also visualize the paths involved in sending a specific message by running:
```
pollmsg.py
```

<details>
<summary>Example output</summary>

```
202) rib:development:local:basic@code# pollmsg.py
1
+------------------+-------------------+-------------------+---------------------+----------------+-------------------------------------------------------------+-----------------------------------------------+
|     Trace ID     |       Sender      |     Recipient     |       End Time      | Total Time (s) |                          Force URL                          |                   Jaeger URL                  |
+------------------+-------------------+-------------------+---------------------+----------------+-------------------------------------------------------------+-----------------------------------------------+
| de49144dc5ef7caf | race-client-00001 | race-client-00002 | 2024-01-17 15:29:42 |    0.857001    | http://localhost:6080/?collapse=true&trace=de49144dc5ef7caf | http://localhost:16686/trace/de49144dc5ef7caf |
+------------------+-------------------+-------------------+---------------------+----------------+-------------------------------------------------------------+-----------------------------------------------+
```

</details>

Navigate to one of the "Force URL" links to see a visualization of the network, with links highlighted that were used in transmission of this particular message.

![An image of a force-directed graph with certain edges highlighted to show the path a message took](https://github.com/tst-race/race-quickstart/blob/local-wip/trace-vis-1.png?raw=true)


#### Jaeger Tracing Visualization
Additionally, RACE messages are instrumented via OpenTracing for testing purposes. Using the output of the `pollmsg.py` command, above, you can one of the "Jaeger URL" links to see the distinct hops of the message through the series of nodes in the path. Within Jaeger, you can also see all the traces that each node is involved in, which can include events beyond just routing client messages.

![An image of a jaeger UI showing a list of API calls associated with sending a message](https://github.com/tst-race/race-quickstart/blob/local-wip/jaeger-ui-1.png?raw=true)


### Stopping the Deployment
When you are done testing with a deployment, you should _stop_, then _down_.

```
deployment stop

deployment down
```


<details>
<summary>Example output</summary>

```
```

</details>


## Really Anonymous Messaging
There are two different implementations of anonymous message routing across a resilient overlay network: Prism and Carma. We can easily run either of these instead of the stand-in used above by creating a new deployment and providing a _--network-manager-kit_ argument (last argument in the command below).

### Prism
```
rib deployment local create --name=prism \
    --linux-client-count=4 \
    --linux-server-count=10 \
    --linux-client-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
    --linux-server-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
    --race-core=tag=https://github.com/tst-race/race-core/releases/tag/2.6.0-v1 \
    --network-manager-kit=tag=2.6.0-v1,org=tst-race,repo=race-prism

rib-use local prism
deployment up

deployment start
```


<details>
<summary>Example output</summary>

```
```

</details>

___Note:___
[Prism](https://github.com/tst-race/race-prism) has adaptive organization capability which means __its nodes need a few moments__ (about 10 seconds) to "get organized" after it is started before messages will be able to be routed.

You can send messages and view them in-flight just as in the basic deployment:
```
deployment message send-manual --message="Hello to all and from all!"
sleep 5
pollmsg.py
```


When you are done testing, use the following commands to end the deployment.

```
deployment stop

deployment down
```


<details>
<summary>Example output</summary>

```
```

</details>

### Carma

```
rib deployment local create --name=carma \
    --linux-client-count=4 \
    --linux-server-count=20 \
    --linux-client-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
    --linux-server-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
    --race-core=tag=https://github.com/tst-race/race-core/releases/tag/2.6.0-v1 \
    --network-manager-kit=tag=https://github.com/tst-race/race-carma/releases/tag/2.6.0-v1/

rib-use local carma
deployment up

deployment start
```


<details>
<summary>Example output</summary>

```
```

</details>

You can send messages and view them in-flight just as in the basic deployment:
```
deployment message send-manual --message="Hello to all and from all!"
sleep 5
pollmsg.py
```

___Note:___ [Carma](https://github.com/tst-race/race-carma) has a notion of a "batch" of messages that are processed simultaneously to mitigate using timing-analyses to deanonymize messaging. Therefore, you will need to send multiple messages ___(generally at least 4)___ before the system will route them. Thus, if you send a single message it may never arrive until you send more, but it _is_ being stored in the server overlay awaiting more messages to be mixed with, so it will not be lost.

___Note 2:___ Carma actually breaks the OpenTracing-based visualization of message tracking used in the Jaeger UI and the force-directed graph renderings. Therefore, despite being delivered, messages will appear to only make it "halfway" through the Carma routing procedure: from sender, to "ingress mailbox" server, to "mixing committee" and then appear to stop. The latter half of the message's transit is essentially the reverse: it moves _from_ mixing committees, to "receiving mailbox" servers, and then to the ultimate destination client.


When you are done testing, use the following commands to end the deployment.

```
deployment stop

deployment down
```

<details>
<summary>Example output</summary>

```
```

</details>


## Really Resilient Networking
Up to this point we have been using stand-in channels for communications between nodes. Specifically, servers have been using a simple TCP socket to talk to each other, and clients and servers have been posting data to each other via a redis intermediary. In both cases, the messages have just been raw encrypted bytes and presumably easy to filter or block. Now we add in covert channels to make it impossible to detect or block the RACE system's network traffic. 


### Carma with Obfs and ssEmail

This deployment is the same as the Carma deployment above, but now we have added arguments to include [Obfs](https://github.com/tst-race/race-obfs) and [ssEmail](https://github.com/tst-race/race-semanticsteg) for communications. Carma chooses how and where to use these channels - in this case it will use Obfs for server-to-server traffic since it is high-bandwidth and low-latency, but requires revealing the IP address of the other endpoint.

Carma will use ssEmail for client-server traffic: ssEmail transmits data by encoding it into the semantic space of an image (i.e. it generates wholly new images, rather than altering existing ones) and then attaching the image to an email sent to another node. This prevents either node from learning the IP address of the other, they just know one another's email addresses. In addition to additional processing time to encode and decode images, the ssEmail channel also limits the rate at which emails are sent to avoid standing out. In short, the messages will take longer to be received but the whole system will be stealthy.

___Note:___
This deployment uses ssEmail which has fairly large machine-learning models as part of its functionality. The `create` command pulls down these models, which total about 1.6GB, so it may take a while to complete, just be patient.

```
rib deployment local create --name=carma-obfs-ssEmail \
    --linux-client-count=4 \
    --linux-server-count=20 \
    --linux-client-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
    --linux-server-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
    --race-core=tag=https://github.com/tst-race/race-core/releases/tag/2.6.0-v1 \
    --network-manager-kit=tag=https://github.com/tst-race/race-carma/releases/tag/2.6.0-v1/ \
    --comms-channel=obfs --comms-kit=tag=2.6.0-v1,repo=race-obfs,org=tst-race \
    --comms-channel=ssEmail --comms-kit=tag=2.6.0-v1,repo=race-semanticsteg,org=tst-race
    
rib-use local carma-obfs-ssEmail
deployment up

deployment start
```


<details>
<summary>Example output</summary>

```
```


</details>

___Be Patient!___ using really-covert indirect channels for client-to-server communications slows messaging down significantly, as does running multiple ML-based steganographic techniques on a single host. For reference, sending a single message between each pair of clients (i.e. 20 messages) took roughly 10 minutes for all messages to be received on a laptop with 32GB of RAM and an 8-core 2.3GHz CPU.

You can send messages and view them in-flight just as in the basic deployment:
```
deployment message send-manual --message="Hello to all and from all!"
sleep 5
pollmsg.py
```

When you are done testing, use the following commands to end the deployment.

```
deployment stop

deployment down
```


<details>
<summary>Example output</summary>


```
```

</details>

### Prism with Snowflake, destiniPixelfed, and ssEmail

This deployment is similar to the Prism deployment above, but now using [Snowflake](https://github.com/tst-race/race-snowflake), [ssEmail](https://github.com/tst-race/race-semanticsteg), and [destiniPixelfed](https://github.com/tst-race/race-destini) for communications. We have also reduced the number of servers to 6 instead of 10, because some hosts had difficulty running 10 servers due to memory constraints. 

We will demonstrate the use of some Prism-specific configuration arguments to tell it _how_ we think it should use these channels. In particular, ssEmail and destiniPixelfed are two different _indirect_ channels, meaning they use a third-party service as an intermediary for sending messages. ssEmail is described above; destiniPixelfed also uses image steganography but of a different type (it encodes messages into existing images) and transmits the image by posting it to a mock "whiteboard" service, meant to stand-in for the myriad social media and file drop services that exist on the internet. Our configuration arguments will tell Prism which types of messages to send on each channel to optimize its performance. If left to defaults, Prism will randomly load-balance messaging between the two.

___Note:___
This deployment uses ssEmail which has fairly large machine-learning models as part of its functionality, and destini, which comes bundled with a set of images and videos for encoding into. The `create` command pulls down these models and images, which total about 2GB, so it may take a while to complete, just be patient.

```
rib deployment local create --name=prism-snowflake-destiniPixelfed-ssEmail \
    --linux-client-count=4 \
    --linux-server-count=6 \
    --race-node-arch=x86_64 \
    --linux-client-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
    --linux-server-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
    --race-core=tag=https://github.com/tst-race/race-core/releases/tag/2.6.0-v1 \
    --network-manager-kit=tag=2.6.0-v1,org=tst-race,repo=race-prism \
    --comms-channel=snowflake --comms-kit=tag=2.6.0-v1,org=tst-race,repo=race-snowflake \
    --comms-channel=ssEmail --comms-kit=tag=2.6.0-v1,org=tst-race,repo=race-semanticsteg \
    --comms-channel=destiniPixelfed --comms-kit=tag=2.6.0-v1,org=tst-race,repo=race-destini,asset=race-destini-pixelfed.tar.gz

rib-use local prism-snowflake-destiniPixelfed-ssEmail

deployment config generate --network-manager-custom-args="-TssEmail=uplink,epoch -TdestiniPixelfed=ark,downlink" --force

deployment up

deployment start
```


<details>
<summary>Example output</summary>

```
```

___Be Patient!___ Prism's self-organizing behavior takes a bit longer when using stealthy channels, so __give it a few minutes__ (roughly 5 minutes, depending on host resources) before expecting messages to get through. After the organization has occurred, messages should have 30-90s latencies, although host resource constraints could increase those.

You can use the network visualization tools to determine if Prism clients are ready:
```
inspect_links.py | wc -l
```
Then browse to `localhost:6080` - if the force diagram shows client nodes with links connecting to servers, then things are ready.


Once connected, you can send messages and view them in-flight just as in the basic deployment:
```
deployment message send-manual --message="Hello to all and from all!"
sleep 5
pollmsg.py
```

When you are done testing, use the following commands to end the deployment.

```
deployment stop

deployment down
```


<details>
<summary>Example output</summary>

```
```

</details>


## Exploring Your Deployment
There are a few different ways to quickly examine what's going on while a RACE deployment is running.

### RIB Deployment Status
The `deployment status` command will display varying levels of detail, depending on the number of `-d` arguments (e.g. `-dd`), about the status of RACE nodes and other deployment infrastructure containers. This is good for checking for gross errors with the deployment, like a container crashing.

<details><summary>Example Output</summary>

```
5738) rib:development:local:basic@code# deployment status -dd
Deployment basic apps: all running
Counts:
	0/11 are down
	0/11 are error
	0/11 are initializing
	0/11 are ready to bootstrap
	0/11 are ready to generate config
	0/11 are ready to install configs
	0/11 are ready to publish configs
	0/11 are ready to start
	0/11 are ready to tar configs
	11/11 are running
	0/11 are stopped
	0/11 are unknown
Details:
	race-client-00001: running
	race-client-00002: running
	race-client-00003: running
	race-client-00004: running
	race-client-00005: running
	race-server-00001: running
	race-server-00002: running
	race-server-00003: running
	race-server-00004: running
	race-server-00005: running
	race-server-00006: running
Deployment basic containers: all running
Counts:
	0/19 are exited
	0/19 are not present
	19/19 are running
	0/19 are starting
	0/19 are unhealthy
	0/19 are unknown
Details:
	dnsproxy: running
	elasticsearch: running
	graph renderer: running
	jaeger-collector: running
	jaeger-query: running
	kibana: running
	openvpn: running
	race-client-00001: running
	race-client-00002: running
	race-client-00003: running
	race-client-00004: running
	race-server-00001: running
	race-server-00002: running
	race-server-00003: running
	race-server-00004: running
	race-server-00005: running
	race-server-00006: running
	rib-file-server: running
	rib-redis: running
Deployment basic services: all running
Counts:
	0/8 are error
	0/8 are not running
	8/8 are running
	0/8 are unknown
Details:
	External Services: all running
	RiB: all running
```

</details>

### Jaegertracing
As described [above](#jaeger-tracing-visualization), navigating to [http://localhost:16686/](http://localhost:16686/) provides the Jaeger UI for examining OpenTracing data. This allows you to look at different API events occurring on different nodes, can be particularly useful for seeing "breaks" in a distributed event, like a message which reaches a node, claims to be _sent_, but then is never _received_.

### Network Visualization
Also described [above](#network-visualization), running the  `inspect_links.py | wc -l` command and viewing the rendered visualization at [http://localhost:6080]([http://localhost:6080]) can provide information about which nodes are connected to one another and about the overall network state in terms of connectivity.

### Logs
The most detailed but very low-level way to look at a deployment is looking at the log files for the RACE nodes. For linux nodes, these are all volume-mounted into the node containers, and can be viewed at `~/.race/rib/deployments/local/<deployment name>/logs/<node-name>/`. These are very verbose and more appropriate for developer use, see [Developer Documentation](https://github.com/tst-race/race-docs/blob/main/RACE%20developer%20guide.md).

## Exploring More of RACE
This quick start guide got you up and running with core RACE system, both anonymous routing network-manager plugins, and a subset of the comms plugins that provide resilience at the network-layer. From here you can explore a number of different aspects of RACE in more depth:

- [What is RACE: The Longer Story](https://github.com/tst-race/race-docs/blob/main/what-is-race.md)
- [Full list of RACE Channels](https://github.com/tst-race/race-docs/blob/main/race-channels.md)
- [Bootstrapping: how do new clients join?](https://github.com/tst-race/race-in-the-box/blob/2.6.0/documentation/how-to/node-bootstrap-test.md)
- [Developer Documentation](https://github.com/tst-race/race-docs/blob/main/RACE%20developer%20guide.md)
- [RIB Documentation](https://github.com/tst-race/race-in-the-box/tree/2.6.0/documentation)


## Clean Up: Removing RACE/RiB
This sequence of steps should remove everything you may have just pulled and installed as part of this quickstart. 

### Down Deployments
To check if you still have a deployment running, run:
```
rib deployment local active
```

<details><summary>Example Output</summary>

```
1955) rib:development@code# rib deployment local active
Getting Active Deployments
<NAME>
```

</details>

This will output the name of an active deployment; if there is one, then run:


```
deployment local stop --name=<NAME> --force
deployment local down --name=<NAME> --force
```

You can verify it has been brought down by running the active command again.
```
1955) rib:development@code# rib deployment local active
Getting Active Deployments
None
```


### Android Device
If you did not use an Android device you can skip this.

Plug the device into the RiB host so ADB can detect it, then run:
```
deployment local bridged android unprepare --name=<NAME>
```

<details><summary>Example Output</summary>
```
1955) rib:development@code# deployment local bridged android unprepare --name=<NAME>
Unpreparing Android device <YOUR DEVICE SERIAL ID>
Unprepared Android device <YOUR DEVICE SERIAL ID>
```
</details>

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
sudo rm -rf ~/.race
```
