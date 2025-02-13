
# Horizon Offline Voice Assistant Example Edge Service for Raspberry Pi

## Using the Offline Voice Assistant Example Edge Service

- Before following the steps in this section, install the Horizon agent on your edge device and point it to your Horizon exchange. Also get an API key that is associated with your Horizon instance.
- Set your exchange org:
```
export HZN_ORG_ID="<yourorg>"
```
- Set your exchange user credentials in the Horizon-supported environment variable and verify it:
```
export HZN_EXCHANGE_USER_AUTH="iamapikey:<myapikey>"
hzn exchange user list
```
- Choose a id and token for your edge node, create it, and verify it:
```
export HZN_EXCHANGE_NODE_AUTH="<mynodeid>:<mynodetoken>"
hzn exchange node create -n $HZN_EXCHANGE_NODE_AUTH
hzn exchange node confirm
```
- Register your edge node with Horizon to use the helloworld pattern:
```
hzn register -p IBM/pattern-ibm.processtext-arm
```
- Look at the Horizon agreement until it is finalized and then see the running container:
```
hzn agreement list
docker ps
```
- See the processtext service output:
```
# soon you will use 'hzn service log ...' for all platforms
# For now on Linux:
tail -f /var/log/syslog | grep OVA
``` 
- Unregister your edge node, stopping the processtext service:
```
hzn unregister -f
```

## First-Time Edge Service Developer - Building and Publishing Your Own Version of the Offline Voice Assistant Edge Service

If you want to create your own Horizon edge service, follow the next 2 sections to copy the Offline Voice Assistant Example Edge Service and start modifying it.

### Preconditions for Developing Your Own Service

- First, go through the steps in the section above to run the IBM processtext service on an edge node.
- Get a docker hub id at https://hub.docker.com/ , if you don't already have one. (This example is set up to store the docker image in docker hub, but by modifying DOCKER_IMAGE_BASE you can store it in another registry.) Login to the docker registry using your id:
```
echo 'mydockerpw' | docker login -u mydockehubid --password-stdin
```
- If you have the HZN_ORG_ID environment variable set from previous work, unset it (in a moment this value will now come from `horizon/hzn.json`):
```
unset HZN_ORG_ID
```
- Cd to the directory in which you want to create your new service and then:
```
hzn dev service new -o <org-id> -s <service-name> -i <docker-image-base>
# E.g.:  hzn dev service new -o bp@someemail.com -s bp.processtext -i brucemp/bp. processtext
```
- As part of the above section "Using the Offline Voice Assistant Example Edge Service", you created your Exchange user credentials and edge node credentials. Ensure they are set and verify them:
```
export HZN_EXCHANGE_USER_AUTH="iamapikey:<myapikey>"
hzn exchange user list
export HZN_EXCHANGE_NODE_AUTH="<mynodeid>:<mynodetoken>"
hzn exchange node confirm
```

### Building and Publishing Your Own Version of the Offline Voice Assistant Example Edge Service

- Build the processtext docker image:
```
make
```
- Test the service by having Horizon start it locally:
```
hzn dev service start -S
```
- See the docker container running and look at the output:
```
docker ps
# soon you will use 'hzn service log ...' for all platforms
# For now on Linux:
tail -f /var/log/syslog | grep OVA
```
- See the environment variables Horizon passes into your service container:
```
docker inspect $(docker ps -q --filter name=ibm.processtext) | jq '.[0].Config.Env'
```
- Stop the service:
```
hzn dev service stop
```
- Create a service signing key pair in `~/.hzn/keys/` (if you haven't already done so):
```
hzn key create <my-company> <my-email>
```
- Have Horizon push your docker image to your registry and publish your service in the Horizon Exchange and see it there:
```
hzn exchange service publish -f horizon/service.definition.json
hzn exchange service list
```
- Publish your edge node deployment pattern in the Horizon Exchange and see it there:
```
hzn exchange pattern publish -f horizon/pattern.json
hzn exchange pattern list
```
- Register your edge node with Horizon to use your deployment pattern (substitute for `SERVICE_NAME` the value you specified above for `hzn dev service new -s`):
```
hzn register -p pattern-SERVICE_NAME-$(hzn architecture)
```
- Look at the Horizon agreement until it is finalized and then see the running container:
```
hzn agreement list
docker ps
```
- See the processtext service output:
```
# soon you will use 'hzn service log ...' for all platforms
# For now on Linux:
tail -f /var/log/syslog | grep OVA
``` 
- Unregister your edge node, stopping the processtext service:
```
hzn unregister -f
```

## Further Learning

To see more Horizon features demonstrated, continue on to the [cpu2msghub example](../../msghub/cpu2msghub).

## Process for the Horizon Development Team to Make Updates to the Offline Voice Assistant Service

- Do the steps in the Preconditions section above, **except**:
    - export `HZN_EXCHANGE_URL` to the staging instance
    - Do **not** run `hzn dev service new ...` (use the git files in this directory instead)
    - export `HZN_EXCHANGE_USER_AUTH` to your credentials in the IBM org
- Make whatever code changes are necessary
- Increment `SERVICE_VERSION` in `horizon/hzn.json`
- Make `~/.hzn/keys/service.private.key` and `~/.hzn/keys/service.public.pem` actually be symbolic links to the common keys we use to sign all of our examples.
- Build, test, and publish service:
```
make publish 
```