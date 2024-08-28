# RACE Basic Troubleshooting

The following contains some basic troubleshooting tips for problems that might come up during executing the quickstart instructions.

## Deployment Create Problems

### tarfile.ReadError: unexpected end of data

This usually results from a download of artifacts being disrupted, likely due to local network problems when downloading larger plugins. The easiest solution is to simply try the deployment create command again, and try to maximize the consistency of your network connection.

### WARNING: Plugin XXX does not contain artifacts for these nodes ['YYY']

This is _just a warning_ for the deployments described in the quickstart guide. When trying your own deployments, this warning may indicate an actual problem. E.g. if you built a custom plugin and only provided x86_64 artifacts but then try to run a deployment with an arm64 node in it.

## Deployment Up Problems

### Initial Up Failure after Long Wait

If your internet connection is particularly slow, the `deployment up` command could timeout on the initial attempt to run it, because that is when docker images are pulled to instantiate containers (e.g. the docker images for the RACE nodes and docker images specified by plugins representing 3rd-party services). If this happens, you should be able to simply run `deployment down --force` to ensure a clean state, and then re-run `deployment up`.

The docker images will have been fetched despite the timeout in the first run, so the second run should complete quickly.

### Deployment Stuck

If a deployment appears "stuck" in a limbo state, first try `deployment down --force`. If that still fails, try forcing down and removing the containers via docker directly, e.g.: `docker stop race-server-00001; docker rm race-server-00001`.