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

<summary>
What are Configuration Files?
</summary>

RACE uses a notion of an initial (or _genesis_) configuration for the entire network that is expressed as a set of coordinated per-node config files. Examples of data in these per-node configs include cryptographic keys and addresses for initial node-to-node connections. RIB automates the creation of these per-node configs by generating or ingesting a [range-config](https://github.com/tst-race/race-in-the-box/blob/2.6.0/documentation/files-images-templates/example-range-config.json) file which represents the network environment (e.g. RACE nodes, their IPs, etc.) and running a [config generation pipeline](https://github.com/tst-race/race-in-the-box/blob/2.6.0/documentation/how-to/deployment-setup/configuration-generation.md). While RIB is intended for orchestrating tests and demonstrations, this aspect of configuration generation could be applicable to assisting construction of real-world RACE deployments.

</details>

If you encounter any problems, try consulting the [troubleshooting tips](troubleshooting.md) or contact us at <race@twosixtech.com>.

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
212) rib:development:local:basic@code# rib deployment local create --name=basic \
>     --linux-client-count=4 \
>     --linux-server-count=6 \
>     --linux-client-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
>     --linux-server-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
>     --race-core=tag=https://github.com/tst-race/race-core/releases/tag/2.6.0-v1
Using default Network manager kit: core=plugin-network-manager-twosix-cpp
Using default Comms channels: ('twoSixDirectCpp', 'twoSixIndirectCpp', 'twoSixIndirectBootstrapCpp')
Using default Comms kits: ('core=plugin-comms-twosix-cpp',)
Using default Artifact manager kits: ('core=plugin-artifact-manager-twosix-cpp-local', 'core=plugin-artifact-manager-twosix-cpp')
Using default Android app: core=raceclient-android
Using default Linux app: core=racetestapp-linux
Using default Node daemon: core=race-node-daemon
Using default Android client image: ghcr.io/tst-race/race-images/race-runtime-android-x86_64:main
Using default Registry client image: ghcr.io/tst-race/race-images/race-runtime-linux:main
Creating Local Deployment: basic

Got genesis-link-addresses for channel /root/.race/rib/deployments/local/basic/configs/comms/PluginCommsTwoSixStub/twoSixIndirectCpp
Got genesis-link-addresses for channel /root/.race/rib/deployments/local/basic/configs/comms/PluginCommsTwoSixStub/twoSixDirectCpp
Got genesis-link-addresses for channel /root/.race/rib/deployments/local/basic/configs/comms/PluginCommsTwoSixStub/twoSixIndirectBootstrapCpp
Network manager config gen status: complete, reason: success
Creating configs archive
Created configs/etc archives for race-client-00001
Created configs/etc archives for race-server-00001
Created configs/etc archives for race-client-00004
Created configs/etc archives for race-server-00002
Created configs/etc archives for race-client-00003
Created configs/etc archives for race-server-00003
Created configs/etc archives for race-server-00004
Created configs/etc archives for race-server-00006
Created configs/etc archives for race-client-00002
Created configs/etc archives for race-server-00005
Created Local Deployment: basic
Run `rib-use local basic` to enable shortcut commands for this deployment
212) rib:development:local:basic@code# 
212) rib:development:local:basic@code# rib-use local basic
Using basic (local)
RiB logs will be written to /root/.race/rib/logs/2024-02-23-10-02-21-6269.log
6269) rib:development:local:basic@code# 
6269) rib:development:local:basic@code# # This command sets a "profile" for RIB operations to save you entering the deployment name and type each time
6269) rib:development:local:basic@code# # "local" means a local deployment contained on a single host, as-opposed to an AWS-based deployment
6269) rib:development:local:basic@code# # "basic" is just the name of this particular deployment, as chosen in the create command above
6269) rib:development:local:basic@code# 
6269) rib:development:local:basic@code# deployment up
Using default RiB mode: local
Using default verbosity: 0
Standing Up Deployment: basic
Starting deployment services
Comms (twoSixIndirectCpp) Start External Services
Comms (twoSixIndirectBootstrapCpp) Start External Services
ArtifactManager (PluginArtifactManagerTwoSixCpp) Start External Services
Waiting for 19 containers to stand up.......................done
Waiting for 10 nodes to stand up...done
Waiting for services to stand up...done
Uploading configs archive
Waiting for 10 nodes to pull configs......done
Stood Up Deployment: basic
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
6269) rib:development:local:basic@code# deployment start
Starting All Nodes In Deployment: basic (local)
Waiting for 10 nodes to start.......done
Started All Nodes In Deployment: basic (local)
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
6269) rib:development:local:basic@code# deployment message list
+------------------+---------+------+-------------------+-------------------+----------+---------------------+---------------------+----------------+
|     Trace ID     | Test ID | Size |       Sender      |     Recipient     |  Status  |      Start Time     |       End Time      | Total Time (s) |
+------------------+---------+------+-------------------+-------------------+----------+---------------------+---------------------+----------------+
| 091fa960c7fdc04d |  Hello  |  26  | race-client-00001 | race-client-00004 | received | 2024-02-23 17:42:04 | 2024-02-23 17:42:05 |    1.720529    |
| 0b6fa87f7b22566d |         |  6   | race-client-00001 | race-client-00002 | received | 2024-02-23 17:42:12 | 2024-02-23 17:42:13 |    1.052256    |
| 18824fd67e00b19d |  Hello  |  26  | race-client-00004 | race-client-00001 | received | 2024-02-23 17:42:04 | 2024-02-23 17:42:05 |    1.721441    |
| 27f45280bfdbd601 |  Hello  |  26  | race-client-00001 | race-client-00003 | received | 2024-02-23 17:42:04 | 2024-02-23 17:42:05 |    1.723083    |
| 334349ac35a45f1b |  Hello  |  26  | race-client-00003 | race-client-00004 | received | 2024-02-23 17:42:04 | 2024-02-23 17:42:05 |    1.719287    |
| 3413125dd6d44d8c |  Hello  |  26  | race-client-00002 | race-client-00003 | received | 2024-02-23 17:42:04 | 2024-02-23 17:42:05 |    1.723765    |
| 53ab7e35200f644b |  Hello  |  26  | race-client-00004 | race-client-00003 | received | 2024-02-23 17:42:04 | 2024-02-23 17:42:05 |    1.721542    |
| 59d058459c196d6a |  Hello  |  26  | race-client-00002 | race-client-00004 | received | 2024-02-23 17:42:04 | 2024-02-23 17:42:05 |    1.721362    |
| 6093fac89424acf2 |  Hello  |  26  | race-client-00001 | race-client-00002 | received | 2024-02-23 17:42:04 | 2024-02-23 17:42:05 |    1.719675    |
| 6b8a932fb23f4509 |  Hello  |  26  | race-client-00002 | race-client-00001 | received | 2024-02-23 17:42:04 | 2024-02-23 17:42:05 |    1.720083    |
| d3e5665a24a276d6 |  Hello  |  26  | race-client-00004 | race-client-00002 | received | 2024-02-23 17:42:04 | 2024-02-23 17:42:05 |    1.723224    |
| f105385ba67110aa |  Hello  |  26  | race-client-00003 | race-client-00001 | received | 2024-02-23 17:42:04 | 2024-02-23 17:42:05 |    1.720974    |
| f9f996daee06f5d1 |  Hello  |  26  | race-client-00003 | race-client-00002 | received | 2024-02-23 17:42:04 | 2024-02-23 17:42:05 |    1.722599    |
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

___NOTE:___ it may take a few seconds for the event to percolate through the node and into the elasticsearch database, so you may need to try re-running the `pollmsg.py` command.


<details>
<summary>Example output</summary>

```
6269) rib:development:local:basic@code# pollmsg.py

13
+------------------+-------------------+-------------------+---------------------+----------------+-------------------------------------------------------------+-----------------------------------------------+
|     Trace ID     |       Sender      |     Recipient     |       End Time      | Total Time (s) |                          Force URL                          |                   Jaeger URL                  |
+------------------+-------------------+-------------------+---------------------+----------------+-------------------------------------------------------------+-----------------------------------------------+
| 0b6fa87f7b22566d | race-client-00001 | race-client-00002 | 2024-02-23 17:42:13 |    1.052256    | http://localhost:6080/?collapse=true&trace=0b6fa87f7b22566d | http://localhost:16686/trace/0b6fa87f7b22566d |
| f9f996daee06f5d1 | race-client-00003 | race-client-00002 | 2024-02-23 17:42:05 |    1.722599    | http://localhost:6080/?collapse=true&trace=f9f996daee06f5d1 | http://localhost:16686/trace/f9f996daee06f5d1 |
| f105385ba67110aa | race-client-00003 | race-client-00001 | 2024-02-23 17:42:05 |    1.720974    | http://localhost:6080/?collapse=true&trace=f105385ba67110aa | http://localhost:16686/trace/f105385ba67110aa |
| d3e5665a24a276d6 | race-client-00004 | race-client-00002 | 2024-02-23 17:42:05 |    1.723224    | http://localhost:6080/?collapse=true&trace=d3e5665a24a276d6 | http://localhost:16686/trace/d3e5665a24a276d6 |
| 6b8a932fb23f4509 | race-client-00002 | race-client-00001 | 2024-02-23 17:42:05 |    1.720083    | http://localhost:6080/?collapse=true&trace=6b8a932fb23f4509 | http://localhost:16686/trace/6b8a932fb23f4509 |
| 6093fac89424acf2 | race-client-00001 | race-client-00002 | 2024-02-23 17:42:05 |    1.719675    | http://localhost:6080/?collapse=true&trace=6093fac89424acf2 | http://localhost:16686/trace/6093fac89424acf2 |
| 59d058459c196d6a | race-client-00002 | race-client-00004 | 2024-02-23 17:42:05 |    1.721362    | http://localhost:6080/?collapse=true&trace=59d058459c196d6a | http://localhost:16686/trace/59d058459c196d6a |
| 53ab7e35200f644b | race-client-00004 | race-client-00003 | 2024-02-23 17:42:05 |    1.721542    | http://localhost:6080/?collapse=true&trace=53ab7e35200f644b | http://localhost:16686/trace/53ab7e35200f644b |
| 3413125dd6d44d8c | race-client-00002 | race-client-00003 | 2024-02-23 17:42:05 |    1.723765    | http://localhost:6080/?collapse=true&trace=3413125dd6d44d8c | http://localhost:16686/trace/3413125dd6d44d8c |
| 334349ac35a45f1b | race-client-00003 | race-client-00004 | 2024-02-23 17:42:05 |    1.719287    | http://localhost:6080/?collapse=true&trace=334349ac35a45f1b | http://localhost:16686/trace/334349ac35a45f1b |
| 27f45280bfdbd601 | race-client-00001 | race-client-00003 | 2024-02-23 17:42:05 |    1.723083    | http://localhost:6080/?collapse=true&trace=27f45280bfdbd601 | http://localhost:16686/trace/27f45280bfdbd601 |
| 18824fd67e00b19d | race-client-00004 | race-client-00001 | 2024-02-23 17:42:05 |    1.721441    | http://localhost:6080/?collapse=true&trace=18824fd67e00b19d | http://localhost:16686/trace/18824fd67e00b19d |
| 091fa960c7fdc04d | race-client-00001 | race-client-00004 | 2024-02-23 17:42:05 |    1.720529    | http://localhost:6080/?collapse=true&trace=091fa960c7fdc04d | http://localhost:16686/trace/091fa960c7fdc04d |
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
6269) rib:development:local:basic@code# deployment stop
Stopping All Nodes In Deployment: basic (local)
Waiting for 10 nodes to stop.........done
Stopped All Nodes In Deployment: basic (local)
6269) rib:development:local:basic@code# 
6269) rib:development:local:basic@code# deployment down
Using default RiB mode: local
Using default verbosity: 0
Tearing Down Deployment: basic
Stopping deployment services
Waiting for 19 containers to tear down...done
Waiting for 10 nodes to tear down...done
Comms (twoSixIndirectCpp) Stop External Services
Comms (twoSixIndirectBootstrapCpp) Stop External Services
ArtifactManager (PluginArtifactManagerTwoSixCpp) Stop External Services
Waiting for services to tear down...done
Tore Down Deployment: basic
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
6269) rib:development:local:basic@code# rib deployment local create --name=prism \
>     --linux-client-count=4 \
>     --linux-server-count=10 \
>     --linux-client-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
>     --linux-server-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
>     --race-core=tag=https://github.com/tst-race/race-core/releases/tag/2.6.0-v1 \
>     --network-manager-kit=tag=2.6.0-v1,org=tst-race,repo=race-prism
Using default Comms channels: ('twoSixDirectCpp', 'twoSixIndirectCpp', 'twoSixIndirectBootstrapCpp')
Using default Comms kits: ('core=plugin-comms-twosix-cpp',)
Using default Artifact manager kits: ('core=plugin-artifact-manager-twosix-cpp-local', 'core=plugin-artifact-manager-twosix-cpp')
Using default Android app: core=raceclient-android
Using default Linux app: core=racetestapp-linux
Using default Node daemon: core=race-node-daemon
Using default Android client image: ghcr.io/tst-race/race-images/race-runtime-android-x86_64:main
Using default Registry client image: ghcr.io/tst-race/race-images/race-runtime-linux:main
Creating Local Deployment: prism
Got genesis-link-addresses for channel /root/.race/rib/deployments/local/prism/configs/comms/PluginCommsTwoSixStub/twoSixIndirectBootstrapCpp
Got genesis-link-addresses for channel /root/.race/rib/deployments/local/prism/configs/comms/PluginCommsTwoSixStub/twoSixDirectCpp
Got genesis-link-addresses for channel /root/.race/rib/deployments/local/prism/configs/comms/PluginCommsTwoSixStub/twoSixIndirectCpp
Network manager config gen status: complete, reason: success
Creating configs archive
Created configs/etc archives for race-server-00010
Created configs/etc archives for race-server-00002
Created configs/etc archives for race-server-00007
Created configs/etc archives for race-server-00001
Created configs/etc archives for race-client-00001
Created configs/etc archives for race-client-00003
Created configs/etc archives for race-server-00009
Created configs/etc archives for race-server-00006
Created configs/etc archives for race-server-00004
Created configs/etc archives for race-client-00004
Created configs/etc archives for race-client-00002
Created configs/etc archives for race-server-00003
Created configs/etc archives for race-server-00005
Created configs/etc archives for race-server-00008
Created Local Deployment: prism
Run `rib-use local prism` to enable shortcut commands for this deployment
6269) rib:development:local:basic@code# 
6269) rib:development:local:basic@code# rib-use local prism
Using prism (local)
RiB logs will be written to /root/.race/rib/logs/2024-02-23-10-02-21-7133.log
7133) rib:development:local:prism@code# deployment up
Using default RiB mode: local
Using default verbosity: 0
Standing Up Deployment: prism
Starting deployment services
Comms (twoSixIndirectBootstrapCpp) Start External Services
Comms (twoSixIndirectCpp) Start External Services
ArtifactManager (PluginArtifactManagerTwoSixCpp) Start External Services
Waiting for 23 containers to stand up.......................done
Waiting for 14 nodes to stand up...done
Waiting for services to stand up...done
Uploading configs archive
Waiting for 14 nodes to pull configs......done
Stood Up Deployment: prism
7133) rib:development:local:prism@code# 
7133) rib:development:local:prism@code# deployment start
Starting All Nodes In Deployment: prism (local)
Waiting for 14 nodes to start.....done
Started All Nodes In Deployment: prism (local)
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
7133) rib:development:local:prism@code# deployment stop
Stopping All Nodes In Deployment: prism (local)
Waiting for 14 nodes to stop............done
Stopped All Nodes In Deployment: prism (local)
7133) rib:development:local:prism@code# 
7133) rib:development:local:prism@code# deployment down
Using default RiB mode: local
Using default verbosity: 0
Tearing Down Deployment: prism
Stopping deployment services
Waiting for 23 containers to tear down...done
Waiting for 14 nodes to tear down...done
Comms (twoSixIndirectBootstrapCpp) Stop External Services
Comms (twoSixIndirectCpp) Stop External Services
ArtifactManager (PluginArtifactManagerTwoSixCpp) Stop External Services
Waiting for services to tear down...done
Tore Down Deployment: prism
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
7133) rib:development:local:prism@code# rib deployment local create --name=carma \
>     --linux-client-count=4 \
>     --linux-server-count=20 \
>     --linux-client-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
>     --linux-server-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
>     --race-core=tag=https://github.com/tst-race/race-core/releases/tag/2.6.0-v1 \
>     --network-manager-kit=tag=https://github.com/tst-race/race-carma/releases/tag/2.6.0-v1/
Using default Comms channels: ('twoSixDirectCpp', 'twoSixIndirectCpp', 'twoSixIndirectBootstrapCpp')
Using default Comms kits: ('core=plugin-comms-twosix-cpp',)
Using default Artifact manager kits: ('core=plugin-artifact-manager-twosix-cpp-local', 'core=plugin-artifact-manager-twosix-cpp')
Using default Android app: core=raceclient-android
Using default Linux app: core=racetestapp-linux
Using default Node daemon: core=race-node-daemon
Using default Android client image: ghcr.io/tst-race/race-images/race-runtime-android-x86_64:main
Using default Registry client image: ghcr.io/tst-race/race-images/race-runtime-linux:main
Creating Local Deployment: carma
Got genesis-link-addresses for channel /root/.race/rib/deployments/local/carma/configs/comms/PluginCommsTwoSixStub/twoSixDirectCpp
Got genesis-link-addresses for channel /root/.race/rib/deployments/local/carma/configs/comms/PluginCommsTwoSixStub/twoSixIndirectBootstrapCpp
Got genesis-link-addresses for channel /root/.race/rib/deployments/local/carma/configs/comms/PluginCommsTwoSixStub/twoSixIndirectCpp
Network manager config gen status: complete, reason: success
Creating configs archive
Created configs/etc archives for race-server-00016
Created configs/etc archives for race-server-00008
Created configs/etc archives for race-server-00014
Created configs/etc archives for race-server-00020
Created configs/etc archives for race-server-00013
Created configs/etc archives for race-server-00002
Created configs/etc archives for race-client-00002
Created configs/etc archives for race-server-00006
Created configs/etc archives for race-server-00011
Created configs/etc archives for race-server-00005
Created configs/etc archives for race-server-00019
Created configs/etc archives for race-server-00007
Created configs/etc archives for race-server-00017
Created configs/etc archives for race-client-00003
Created configs/etc archives for race-server-00018
Created configs/etc archives for race-client-00001
Created configs/etc archives for race-server-00009
Created configs/etc archives for race-server-00010
Created configs/etc archives for race-server-00004
Created configs/etc archives for race-client-00004
Created configs/etc archives for race-server-00001
Created configs/etc archives for race-server-00012
Created configs/etc archives for race-server-00003
Created configs/etc archives for race-server-00015
Created Local Deployment: carma
Run `rib-use local carma` to enable shortcut commands for this deployment
7133) rib:development:local:prism@code# 
7133) rib:development:local:prism@code# rib-use local carma
Using carma (local)
RiB logs will be written to /root/.race/rib/logs/2024-02-23-10-02-21-7949.log
7949) rib:development:local:carma@code# deployment up
Using default RiB mode: local
Using default verbosity: 0
Standing Up Deployment: carma
Starting deployment services
Comms (twoSixIndirectBootstrapCpp) Start External Services
Comms (twoSixIndirectCpp) Start External Services
ArtifactManager (PluginArtifactManagerTwoSixCpp) Start External Services
Waiting for 33 containers to stand up........................done
Waiting for 24 nodes to stand up...done
Waiting for services to stand up...done
Uploading configs archive
Waiting for 24 nodes to pull configs......done
Stood Up Deployment: carma
7949) rib:development:local:carma@code# 
7949) rib:development:local:carma@code# deployment start
Starting All Nodes In Deployment: carma (local)
Waiting for 24 nodes to start......done
Started All Nodes In Deployment: carma (local)
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
7949) rib:development:local:carma@code# deployment stop
Stopping All Nodes In Deployment: carma (local)
Waiting for 24 nodes to stop....
.......done
Stopped All Nodes In Deployment: carma (local)
7949) rib:development:local:carma@code# 
7949) rib:development:local:carma@code# deployment down
Using default RiB mode: local
Using default verbosity: 0
Tearing Down Deployment: carma
Stopping deployment services
Waiting for 33 containers to tear down...done
Waiting for 24 nodes to tear down...done
Comms (twoSixIndirectBootstrapCpp) Stop External Services
Comms (twoSixIndirectCpp) Stop External Services
ArtifactManager (PluginArtifactManagerTwoSixCpp) Stop External Services
Waiting for services to tear down...done
Tore Down Deployment: carma
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
7949) rib:development:local:carma@code# rib deployment local create --name=carma-obfs-ssEmail \
>     --linux-client-count=4 \
>     --linux-server-count=20 \
>     --linux-client-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
>     --linux-server-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
>     --race-core=tag=https://github.com/tst-race/race-core/releases/tag/2.6.0-v1 \
>     --network-manager-kit=tag=https://github.com/tst-race/race-carma/releases/tag/2.6.0-v1/ \
>     --comms-channel=obfs --comms-kit=tag=2.6.0-v1,repo=race-obfs,org=tst-race \
>     --comms-channel=ssEmail --comms-kit=tag=2.6.0-v1,repo=race-semanticsteg,org=tst-race

Using default Artifact manager kits: ('core=plugin-artifact-manager-twosix-cpp-local', 'core=plugin-artifact-manager-twosix-cpp')
Using default Android app: core=raceclient-android
Using default Linux app: core=racetestapp-linux
Using default Node daemon: core=race-node-daemon
Using default Android client image: ghcr.io/tst-race/race-images/race-runtime-android-x86_64:main
Using default Registry client image: ghcr.io/tst-race/race-images/race-runtime-linux:main
Creating Local Deployment: carma-obfs-ssEmail
WARNING: Plugin PluginObfs does not contain artifacts for these nodes ['linux_x86_64_client']
warning: 2024/02/23 17:49:52 generating new server config
warning: 2024/02/23 17:49:52 generating new server config
warning: 2024/02/23 17:49:52 generating new server config
warning: 2024/02/23 17:49:52 generating new server config
warning: 2024/02/23 17:49:52 generating new server config
warning: 2024/02/23 17:49:52 generating new server config
warning: 2024/02/23 17:49:52 generating new server config
warning: 2024/02/23 17:49:52 generating new server config
warning: 2024/02/23 17:49:52 generating new server config
warning: 2024/02/23 17:49:52 generating new server config
warning: 2024/02/23 17:49:52 generating new server config
warning: 2024/02/23 17:49:52 generating new server config
warning: 2024/02/23 17:49:52 generating new server config
warning: 2024/02/23 17:49:52 generating new server config
warning: 2024/02/23 17:49:52 generating new server config
warning: 2024/02/23 17:49:52 generating new server config
warning: 2024/02/23 17:49:52 generating new server config
warning: 2024/02/23 17:49:52 generating new server config
warning: 2024/02/23 17:49:52 generating new server config
warning: 2024/02/23 17:49:52 generating new server config
Got genesis-link-addresses for channel /root/.race/rib/deployments/local/carma-obfs-ssEmail/configs/comms/PluginObfs/obfs
warning: [2024-02-23 17:49:52,455][line 869][get_args][DEBUG] : Ensuring range /root/.race/rib/deployments/local/carma-obfs-ssEmail/configs/race-config.json exists
warning: [2024-02-23 17:49:52,456][line 873][get_args][DEBUG] : Ensuring network-manager-request /root/.race/rib/deployments/local/carma-obfs-ssEmail/configs/network-manager/carma/network-manager-request.json exists
warning: [2024-02-23 17:49:52,456][line 879][get_args][INFO] : config-dir /root/.race/rib/deployments/local/carma-obfs-ssEmail/configs/comms/PluginSemanticSteg doesn't exist ; creating it now
warning: [2024-02-23 17:49:52,456][line 100][main][INFO] : Starting Process To Generate RACE Plugin Config
warning: [2024-02-23 17:49:52,456][line 102][main][INFO] : -
warning: Copying channel_properties.json to correct directory
warning: -
warning: [2024-02-23 17:49:52,456][line 121][main][INFO] : -
warning: Generating link profile configs
warning: -
warning: [2024-02-23 17:49:52,456][line 476][generate_link_profile_configs][INFO] : Getting range.json for RACE Link Profile Configs
warning: [2024-02-23 17:49:52,456][line 486][generate_link_profile_configs][INFO] : Reading network manager requests from /root/.race/rib/deployments/local/carma-obfs-ssEmail/configs/network-manager/carma/network-manager-request.json
warning: [2024-02-23 17:49:52,456][line 575][generate_link_profile_configs][DEBUG] : race_configs name: carma-obfs-ssEmail
warning: [2024-02-23 17:49:52,456][line 576][generate_link_profile_configs][DEBUG] : race_configs bastion: {}
warning: [2024-02-23 17:49:52,456][line 577][generate_link_profile_configs][DEBUG] : race_configs: 24 nodes
warning: [2024-02-23 17:49:52,456][line 578][generate_link_profile_configs][DEBUG] : race_configs: 1 enclaves
warning: [2024-02-23 17:49:52,456][line 579][generate_link_profile_configs][DEBUG] : race_configs services: 
warning: []
warning: [2024-02-23 17:49:52,456][line 584][generate_link_profile_configs][DEBUG] : STARTING COVER-SERVICE LOOPS WITH PXFD_INDEX 0, SFTP_INDEX 1, EMAIL_INDEX 2, NODE_PXFD_INDEX 0, NODE_SFTP_INDEX 1, NODE_EMAIL_INDEX 2
warning: [2024-02-23 17:49:52,457][line 664][generate_link_profile_configs][DEBUG] : Deleting index 3 of 4
warning: [2024-02-23 17:49:52,457][line 678][generate_link_profile_configs][DEBUG] : Adding ['race-server-00010'] to race-client-00001's creator personas
warning: [2024-02-23 17:49:52,457][line 664][generate_link_profile_configs][DEBUG] : Deleting index 3 of 8
warning: [2024-02-23 17:49:52,457][line 678][generate_link_profile_configs][DEBUG] : Adding ['race-client-00004'] to race-server-00001's creator personas
warning: [2024-02-23 17:49:52,457][line 664][generate_link_profile_configs][DEBUG] : Deleting index 2 of 8
warning: [2024-02-23 17:49:52,457][line 678][generate_link_profile_configs][DEBUG] : Adding ['race-client-00003'] to race-server-00001's creator personas
warning: [2024-02-23 17:49:52,457][line 664][generate_link_profile_configs][DEBUG] : Deleting index 1 of 8
warning: [2024-02-23 17:49:52,457][line 678][generate_link_profile_configs][DEBUG] : Adding ['race-client-00002'] to race-server-00001's creator personas
warning: [2024-02-23 17:49:52,457][line 664][generate_link_profile_configs][DEBUG] : Deleting index 3 of 8
warning: [2024-02-23 17:49:52,457][line 678][generate_link_profile_configs][DEBUG] : Adding ['race-client-00004'] to race-server-00010's creator personas
warning: [2024-02-23 17:49:52,457][line 664][generate_link_profile_configs][DEBUG] : Deleting index 2 of 8
warning: [2024-02-23 17:49:52,457][line 678][generate_link_profile_configs][DEBUG] : Adding ['race-client-00003'] to race-server-00010's creator personas
warning: [2024-02-23 17:49:52,457][line 664][generate_link_profile_configs][DEBUG] : Deleting index 1 of 8
warning: [2024-02-23 17:49:52,457][line 678][generate_link_profile_configs][DEBUG] : Adding ['race-client-00002'] to race-server-00010's creator personas
warning: [2024-02-23 17:49:52,457][line 664][generate_link_profile_configs][DEBUG] : Deleting index 3 of 4
warning: [2024-02-23 17:49:52,457][line 678][generate_link_profile_configs][DEBUG] : Adding ['race-server-00010'] to race-client-00002's creator personas
warning: [2024-02-23 17:49:52,457][line 664][generate_link_profile_configs][DEBUG] : Deleting index 3 of 4
warning: [2024-02-23 17:49:52,457][line 678][generate_link_profile_configs][DEBUG] : Adding ['race-server-00010'] to race-client-00003's creator personas
warning: [2024-02-23 17:49:52,457][line 664][generate_link_profile_configs][DEBUG] : Deleting index 3 of 4
warning: [2024-02-23 17:49:52,457][line 678][generate_link_profile_configs][DEBUG] : Adding ['race-server-00010'] to race-client-00004's creator personas
warning: [2024-02-23 17:49:52,457][line 686][generate_link_profile_configs][INFO] : GENERATED 32 LINK-PROFILES
warning: [2024-02-23 17:49:52,457][line 687][generate_link_profile_configs][INFO] : DELETED (combined) 10 LINK-PROFILES
warning: [2024-02-23 17:49:52,457][line 702][generate_user_responses_configs][INFO] : Getting range.json for RACE Link Profile Configs
warning: [2024-02-23 17:49:52,457][line 709][generate_user_responses_configs][DEBUG] : RACE_nodes: 24 nodes
warning: [2024-02-23 17:49:52,457][line 717][generate_user_responses_configs][DEBUG] : STARTING COVER-SERVICE LOOPS WITH PXFD_INDEX 0, SFTP_INDEX 1, EMAIL_INDEX 2, NODE_PXFD_INDEX 0, NODE_SFTP_INDEX 1, NODE_EMAIL_INDEX 2
warning: [2024-02-23 17:49:52,459][line 126][main][INFO] : -
warning: Done generating link profile configs. Storing RACE genesis-link-addresses in /root/.race/rib/deployments/local/carma-obfs-ssEmail/configs/comms/PluginSemanticSteg/ssEmail
warning: -
warning: [2024-02-23 17:49:52,459][line 147][main][INFO] : -
warning: Done storing link profile configs. Storing fulfilled network manager requests in /root/.race/rib/deployments/local/carma-obfs-ssEmail/configs/comms/PluginSemanticSteg/ssEmail
warning: -
warning: [2024-02-23 17:49:52,460][line 162][main][INFO] : -
warning: Process To Generate RACE genesis-link-addresses.json Complete
warning: -
Got genesis-link-addresses for channel /root/.race/rib/deployments/local/carma-obfs-ssEmail/configs/comms/PluginSemanticSteg/ssEmail
Network manager config gen status: complete, reason: success
Creating configs archive
Created configs/etc archives for race-server-00015
Created configs/etc archives for race-server-00019
Created configs/etc archives for race-client-00001
Created configs/etc archives for race-server-00012
Created configs/etc archives for race-server-00014
Created configs/etc archives for race-server-00007
Created configs/etc archives for race-server-00017
Created configs/etc archives for race-server-00016
Created configs/etc archives for race-server-00020
Created configs/etc archives for race-server-00013
Created configs/etc archives for race-client-00004
Created configs/etc archives for race-server-00009
Created configs/etc archives for race-client-00003
Created configs/etc archives for race-server-00001
Created configs/etc archives for race-server-00004
Created configs/etc archives for race-server-00010
Created configs/etc archives for race-server-00018
Created configs/etc archives for race-server-00011
Created configs/etc archives for race-server-00006
Created configs/etc archives for race-server-00002
Created configs/etc archives for race-server-00008
Created configs/etc archives for race-server-00003
Created configs/etc archives for race-client-00002
Created configs/etc archives for race-server-00005
Created Local Deployment: carma-obfs-ssEmail
Run `rib-use local carma-obfs-ssEmail` to enable shortcut commands for this deployment
7949) rib:development:local:carma@code#     
7949) rib:development:local:carma@code# rib-use local carma-obfs-ssEmail
Using carma-obfs-ssEmail (local)
RiB logs will be written to /root/.race/rib/logs/2024-02-23-10-02-21-8741.log
8741) rib:development:local:carma-obfs-ssEmail@code# deployment up
Using default RiB mode: local
Using default verbosity: 0
Standing Up Deployment: carma-obfs-ssEmail
Starting deployment services
Comms (ssEmail) Start External Services
ArtifactManager (PluginArtifactManagerTwoSixCpp) Start External Services
Waiting for 33 containers to stand up.....	`...................done
Waiting for 24 nodes to stand up...done
Waiting for services to stand up...done
Uploading configs archive
Waiting for 24 nodes to pull configs......done
Stood Up Deployment: carma-obfs-ssEmail
8741) rib:development:local:carma-obfs-ssEmail@code# 
8741) rib:development:local:carma-obfs-ssEmail@code# deployment start
Starting All Nodes In Deployment: carma-obfs-ssEmail (local)
Waiting for 24 nodes to start........done
Started All Nodes In Deployment: carma-obfs-ssEmail (local)
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
8741) rib:development:local:carma-obfs-ssEmail@code# deployment stop
Stopping All Nodes In Deployment: carma-obfs-ssEmail (local)

Waiting for 24 nodes to stop..........done
Stopped All Nodes In Deployment: carma-obfs-ssEmail (local)
8741) rib:development:local:carma-obfs-ssEmail@code# 
8741) rib:development:local:carma-obfs-ssEmail@code# deployment down
Using default RiB mode: local
Using default verbosity: 0
Tearing Down Deployment: carma-obfs-ssEmail
Stopping deployment services
Waiting for 33 containers to tear down...done
Waiting for 24 nodes to tear down...done
Comms (ssEmail) Stop External Services
ArtifactManager (PluginArtifactManagerTwoSixCpp) Stop External Services
Waiting for services to tear down...done
Tore Down Deployment: carma-obfs-ssEmail
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
8741) rib:development:local:carma-obfs-ssEmail@code# rib deployment local create --name=prism-snowflake-destiniPixelfed-ssEmail \
>     --linux-client-count=4 \
>     --linux-server-count=6 \
>     --linux-client-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
>     --linux-server-image=ghcr.io/tst-race/race-images/race-runtime-linux:main \
>     --race-core=tag=https://github.com/tst-race/race-core/releases/tag/2.6.0-v1 \
>     --network-manager-kit=tag=2.6.0-v1,org=tst-race,repo=race-prism \
>     --comms-channel=snowflake --comms-kit=tag=2.6.0-v1,org=tst-race,repo=race-snowflake \
>     --comms-channel=ssEmail --comms-kit=tag=2.6.0-v1,org=tst-race,repo=race-semanticsteg \
>     --comms-channel=destiniPixelfed --comms-kit=tag=2.6.0-v1,org=tst-race,repo=race-destini,asset=race-destini-pixelfed.tar.gz
Using default Artifact manager kits: ('core=plugin-artifact-manager-twosix-cpp-local', 'core=plugin-artifact-manager-twosix-cpp')
Using default Android app: core=raceclient-android
Using default Linux app: core=racetestapp-linux
Using default Node daemon: core=race-node-daemon
Using default Android client image: ghcr.io/tst-race/race-images/race-runtime-android-x86_64:main
Using default Registry client image: ghcr.io/tst-race/race-images/race-runtime-linux:main
Creating Local Deployment: prism-snowflake-destiniPixelfed-ssEmail
WARNING: Plugin SnowflakePluginComms does not contain artifacts for these nodes ['linux_x86_64_client']

Got genesis-link-addresses for channel /root/.race/rib/deployments/local/prism-snowflake-destiniPixelfed-ssEmail/configs/comms/SnowflakePluginComms/snowflake
warning: [2024-02-23 17:54:01,703][line 869][get_args][DEBUG] : Ensuring range /root/.race/rib/deployments/local/prism-snowflake-destiniPixelfed-ssEmail/configs/race-config.json exists
warning: [2024-02-23 17:54:01,703][line 873][get_args][DEBUG] : Ensuring network-manager-request /root/.race/rib/deployments/local/prism-snowflake-destiniPixelfed-ssEmail/configs/network-manager/prism/network-manager-request.json exists
warning: [2024-02-23 17:54:01,703][line 879][get_args][INFO] : config-dir /root/.race/rib/deployments/local/prism-snowflake-destiniPixelfed-ssEmail/configs/comms/PluginSemanticSteg doesn't exist ; creating it now
warning: [2024-02-23 17:54:01,703][line 100][main][INFO] : Starting Process To Generate RACE Plugin Config
warning: [2024-02-23 17:54:01,703][line 102][main][INFO] : -
warning: Copying channel_properties.json to correct directory
warning: -
warning: [2024-02-23 17:54:01,703][line 121][main][INFO] : -
warning: Generating link profile configs
warning: -
warning: [2024-02-23 17:54:01,703][line 476][generate_link_profile_configs][INFO] : Getting range.json for RACE Link Profile Configs
warning: [2024-02-23 17:54:01,703][line 486][generate_link_profile_configs][INFO] : Reading network manager requests from /root/.race/rib/deployments/local/prism-snowflake-destiniPixelfed-ssEmail/configs/network-manager/prism/network-manager-request.json
warning: [2024-02-23 17:54:01,703][line 575][generate_link_profile_configs][DEBUG] : race_configs name: prism-snowflake-destiniPixelfed-ssEmail
warning: [2024-02-23 17:54:01,703][line 576][generate_link_profile_configs][DEBUG] : race_configs bastion: {}
warning: [2024-02-23 17:54:01,703][line 577][generate_link_profile_configs][DEBUG] : race_configs: 10 nodes
warning: [2024-02-23 17:54:01,703][line 578][generate_link_profile_configs][DEBUG] : race_configs: 1 enclaves
warning: [2024-02-23 17:54:01,703][line 579][generate_link_profile_configs][DEBUG] : race_configs services: 
warning: []
warning: [2024-02-23 17:54:01,703][line 584][generate_link_profile_configs][DEBUG] : STARTING COVER-SERVICE LOOPS WITH PXFD_INDEX 0, SFTP_INDEX 1, EMAIL_INDEX 2, NODE_PXFD_INDEX 0, NODE_SFTP_INDEX 1, NODE_EMAIL_INDEX 2
warning: [2024-02-23 17:54:01,703][line 686][generate_link_profile_configs][INFO] : GENERATED 0 LINK-PROFILES
warning: [2024-02-23 17:54:01,703][line 687][generate_link_profile_configs][INFO] : DELETED (combined) 0 LINK-PROFILES
warning: [2024-02-23 17:54:01,703][line 702][generate_user_responses_configs][INFO] : Getting range.json for RACE Link Profile Configs
warning: [2024-02-23 17:54:01,703][line 709][generate_user_responses_configs][DEBUG] : RACE_nodes: 10 nodes
warning: [2024-02-23 17:54:01,703][line 717][generate_user_responses_configs][DEBUG] : STARTING COVER-SERVICE LOOPS WITH PXFD_INDEX 0, SFTP_INDEX 1, EMAIL_INDEX 2, NODE_PXFD_INDEX 0, NODE_SFTP_INDEX 1, NODE_EMAIL_INDEX 2
warning: [2024-02-23 17:54:01,705][line 126][main][INFO] : -
warning: Done generating link profile configs. Storing RACE genesis-link-addresses in /root/.race/rib/deployments/local/prism-snowflake-destiniPixelfed-ssEmail/configs/comms/PluginSemanticSteg/ssEmail
warning: -
warning: [2024-02-23 17:54:01,705][line 147][main][INFO] : -
warning: Done storing link profile configs. Storing fulfilled network manager requests in /root/.race/rib/deployments/local/prism-snowflake-destiniPixelfed-ssEmail/configs/comms/PluginSemanticSteg/ssEmail
warning: -
warning: [2024-02-23 17:54:01,705][line 162][main][INFO] : -
warning: Process To Generate RACE genesis-link-addresses.json Complete
warning: -
Got genesis-link-addresses for channel /root/.race/rib/deployments/local/prism-snowflake-destiniPixelfed-ssEmail/configs/comms/PluginSemanticSteg/ssEmail
Got genesis-link-addresses for channel /root/.race/rib/deployments/local/prism-snowflake-destiniPixelfed-ssEmail/configs/comms/DestiniPixelfed/destiniPixelfed
Network manager config gen status: complete, reason: success
Creating configs archive
Created configs/etc archives for race-client-00004
Created configs/etc archives for race-client-00002
Created configs/etc archives for race-client-00001
Created configs/etc archives for race-server-00001
Created configs/etc archives for race-server-00002
Created configs/etc archives for race-server-00005
Created configs/etc archives for race-server-00006
Created configs/etc archives for race-client-00003
Created configs/etc archives for race-server-00003
Created configs/etc archives for race-server-00004
Created Local Deployment: prism-snowflake-destiniPixelfed-ssEmail
Run `rib-use local prism-snowflake-destiniPixelfed-ssEmail` to enable shortcut commands for this deployment
8741) rib:development:local:carma-obfs-ssEmail@code# 
8741) rib:development:local:carma-obfs-ssEmail@code# rib-use local prism-snowflake-destiniPixelfed-ssEmail
Using prism-snowflake-destiniPixelfed-ssEmail (local)
RiB logs will be written to /root/.race/rib/logs/2024-02-23-10-02-21-9233.log
9233) rib:development:local:prism-snowflake-destiniPixelfed-ssEmail@code# 
9233) rib:development:local:prism-snowflake-destiniPixelfed-ssEmail@code# deployment config generate --network-manager-custom-args="-TssEmail=uplink,epoch -TdestiniPixelfed=ark,downlink" --force
Generating configs for deployment prism-snowflake-destiniPixelfed-ssEmail (local)
Got genesis-link-addresses for channel /root/.race/rib/deployments/local/prism-snowflake-destiniPixelfed-ssEmail/configs/comms/SnowflakePluginComms/snowflake
warning: [2024-02-23 17:54:03,922][line 869][get_args][DEBUG] : Ensuring range /root/.race/rib/deployments/local/prism-snowflake-destiniPixelfed-ssEmail/configs/race-config.json exists
warning: [2024-02-23 17:54:03,922][line 873][get_args][DEBUG] : Ensuring network-manager-request /root/.race/rib/deployments/local/prism-snowflake-destiniPixelfed-ssEmail/configs/network-manager/prism/network-manager-request.json exists
warning: [2024-02-23 17:54:03,922][line 879][get_args][INFO] : config-dir /root/.race/rib/deployments/local/prism-snowflake-destiniPixelfed-ssEmail/configs/comms/PluginSemanticSteg doesn't exist ; creating it now
warning: [2024-02-23 17:54:03,922][line 100][main][INFO] : Starting Process To Generate RACE Plugin Config
warning: [2024-02-23 17:54:03,922][line 102][main][INFO] : -
warning: Copying channel_properties.json to correct directory
warning: -
warning: [2024-02-23 17:54:03,923][line 121][main][INFO] : -
warning: Generating link profile configs
warning: -
warning: [2024-02-23 17:54:03,923][line 476][generate_link_profile_configs][INFO] : Getting range.json for RACE Link Profile Configs
warning: [2024-02-23 17:54:03,923][line 486][generate_link_profile_configs][INFO] : Reading network manager requests from /root/.race/rib/deployments/local/prism-snowflake-destiniPixelfed-ssEmail/configs/network-manager/prism/network-manager-request.json
warning: [2024-02-23 17:54:03,923][line 575][generate_link_profile_configs][DEBUG] : race_configs name: prism-snowflake-destiniPixelfed-ssEmail
warning: [2024-02-23 17:54:03,923][line 576][generate_link_profile_configs][DEBUG] : race_configs bastion: {}
warning: [2024-02-23 17:54:03,923][line 577][generate_link_profile_configs][DEBUG] : race_configs: 10 nodes
warning: [2024-02-23 17:54:03,923][line 578][generate_link_profile_configs][DEBUG] : race_configs: 1 enclaves
warning: [2024-02-23 17:54:03,923][line 579][generate_link_profile_configs][DEBUG] : race_configs services: 
warning: []
warning: [2024-02-23 17:54:03,923][line 584][generate_link_profile_configs][DEBUG] : STARTING COVER-SERVICE LOOPS WITH PXFD_INDEX 0, SFTP_INDEX 1, EMAIL_INDEX 2, NODE_PXFD_INDEX 0, NODE_SFTP_INDEX 1, NODE_EMAIL_INDEX 2
warning: [2024-02-23 17:54:03,923][line 686][generate_link_profile_configs][INFO] : GENERATED 0 LINK-PROFILES
warning: [2024-02-23 17:54:03,923][line 687][generate_link_profile_configs][INFO] : DELETED (combined) 0 LINK-PROFILES
warning: [2024-02-23 17:54:03,923][line 702][generate_user_responses_configs][INFO] : Getting range.json for RACE Link Profile Configs
warning: [2024-02-23 17:54:03,923][line 709][generate_user_responses_configs][DEBUG] : RACE_nodes: 10 nodes
warning: [2024-02-23 17:54:03,923][line 717][generate_user_responses_configs][DEBUG] : STARTING COVER-SERVICE LOOPS WITH PXFD_INDEX 0, SFTP_INDEX 1, EMAIL_INDEX 2, NODE_PXFD_INDEX 0, NODE_SFTP_INDEX 1, NODE_EMAIL_INDEX 2
warning: [2024-02-23 17:54:03,925][line 126][main][INFO] : -
warning: Done generating link profile configs. Storing RACE genesis-link-addresses in /root/.race/rib/deployments/local/prism-snowflake-destiniPixelfed-ssEmail/configs/comms/PluginSemanticSteg/ssEmail
warning: -
warning: [2024-02-23 17:54:03,926][line 147][main][INFO] : -
warning: Done storing link profile configs. Storing fulfilled network manager requests in /root/.race/rib/deployments/local/prism-snowflake-destiniPixelfed-ssEmail/configs/comms/PluginSemanticSteg/ssEmail
warning: -
warning: [2024-02-23 17:54:03,926][line 162][main][INFO] : -
warning: Process To Generate RACE genesis-link-addresses.json Complete
warning: -
Got genesis-link-addresses for channel /root/.race/rib/deployments/local/prism-snowflake-destiniPixelfed-ssEmail/configs/comms/PluginSemanticSteg/ssEmail
Got genesis-link-addresses for channel /root/.race/rib/deployments/local/prism-snowflake-destiniPixelfed-ssEmail/configs/comms/DestiniPixelfed/destiniPixelfed
Network manager config gen status: complete, reason: success
Creating configs archive
Created configs/etc archives for race-server-00005
Created configs/etc archives for race-client-00001
Created configs/etc archives for race-server-00002
Created configs/etc archives for race-server-00003
Created configs/etc archives for race-client-00002
Created configs/etc archives for race-server-00004
Created configs/etc archives for race-server-00006
Created configs/etc archives for race-client-00004
Created configs/etc archives for race-client-00003
Created configs/etc archives for race-server-00001
9233) rib:development:local:prism-snowflake-destiniPixelfed-ssEmail@code# 
9233) rib:development:local:prism-snowflake-destiniPixelfed-ssEmail@code# deployment up
Using default RiB mode: local
Using default verbosity: 0
Standing Up Deployment: prism-snowflake-destiniPixelfed-ssEmail
Starting deployment services
Comms (snowflake) Start External Services
Comms (ssEmail) Start External Services
Comms (destiniPixelfed) Start External Services
ArtifactManager (PluginArtifactManagerTwoSixCpp) Start External Services
Waiting for 19 containers to stand up.......................done
Waiting for 10 nodes to stand up...done
Waiting for services to stand up...done
Uploading configs archive
Waiting for 10 nodes to pull configs......done
Stood Up Deployment: prism-snowflake-destiniPixelfed-ssEmail
9233) rib:development:local:prism-snowflake-destiniPixelfed-ssEmail@code# 
9233) rib:development:local:prism-snowflake-destiniPixelfed-ssEmail@code# deployment start
Starting All Nodes In Deployment: prism-snowflake-destiniPixelfed-ssEmail (local)
Waiting for 10 nodes to start........done
Started All Nodes In Deployment: prism-snowflake-destiniPixelfed-ssEmail (local)
```
</details>

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
9233) rib:development:local:prism-snowflake-destiniPixelfed-ssEmail@code# deployment stop
Stopping All Nodes In Deployment: prism-snowflake-destiniPixelfed-ssEmail (local)
Waiting for 10 nodes to stop.......
.........................
Waiting for 2 nodes to stop..........done
Stopped All Nodes In Deployment: prism-snowflake-destiniPixelfed-ssEmail (local)
9233) rib:development:local:prism-snowflake-destiniPixelfed-ssEmail@code# 
9233) rib:development:local:prism-snowflake-destiniPixelfed-ssEmail@code# deployment down

Using default RiB mode: local
Using default verbosity: 0
Tearing Down Deployment: prism-snowflake-destiniPixelfed-ssEmail
Stopping deployment services
Waiting for 19 containers to tear down...done
Waiting for 10 nodes to tear down...done
Comms (snowflake) Stop External Services
Comms (ssEmail) Stop External Services
Comms (destiniPixelfed) Stop External Services
ArtifactManager (PluginArtifactManagerTwoSixCpp) Stop External Services
Waiting for services to tear down...done
Tore Down Deployment: prism-snowflake-destiniPixelfed-ssEmail
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
	0/10 are down
	0/10 are error
	0/10 are initializing
	0/10 are ready to bootstrap
	0/10 are ready to generate config
	0/10 are ready to install configs
	0/10 are ready to publish configs
	0/10 are ready to start
	0/10 are ready to tar configs
	10/10 are running
	0/10 are stopped
	0/10 are unknown
Details:
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
