# HyE - YouTube Docker
This repository contains the deployment files for docker-compose and kubernetes of the HyE - YouTube Service, developed as part of the Master Thesis _``Implementing and evaluating recommendation sharing to increase diversity on YouTube''._

## Components
The entire service consists of three sub-systems (frontend, proxy service, recommendation service) and the deployment of two additional components (ethereum blockchain and nginx reverse proxy), which are briefly explained in the following.

### Ethereum blockchain
The deployment features one ethereum blockchain node which is built from [this Dockerfile](https://github.com/rwth-acis/las2peer-ethereum-cluster/blob/master/docker-images/monitored-geth-client/Dockerfile). Since las2peer in general and our service in particular makes use of smart contracts deployed to this blockchain, it should be started first.

### HyE - YouTube Proxy
The Proxy service is a [las2peer](https://github.com/rwth-acis/las2peer) service and as such its docker deployment starts a las2peer node which automatically attempts to connect to an ethereum blockchainlocated at **ethereum:8545** (both docker-compose and kubernetes implement a DNS to resolve this address to the container's actual IP address).
The container started as part of this deployment runs the docker image built from [this Dockerfile](https://github.com/rwth-acis/HyE-YouTube-Proxy/blob/master/docker/Dockerfile) in the respective repository.
In order to deploy and start the service, it has to be uploaded to the las2peer network.
This is done by logging into the Node Front-End, which by default is served under http://localhost:8081/las2peer, and uploading the service via the [Publish Service](http://localhost:8080/las2peer/webapp/publish-service) tab.
*Note that the upload will probably fail, due to a dependency which exceeds las2peer maximum allowed size of 5MB, however the service can still be started on the node launched by the provided docker image, since the service is contained in its library and classpath*.
Once the upload is finished, start the service via the [View Services](http://localhost:8080/las2peer/webapp/view-services) tab and afterwards visit [/hye-youtube/init](http://localhost:8080/hye-youtube/init) to initialize the service.
***For more details regarding the configuration of this service, please refer to the [README](https://github.com/rwth-acis/hye-youtube-proxy#configurations-properties) contained in its repository***

#### Frontend dependencies
The HyE - YouTube frontend requires the following additional services to function, which also have to be uploaded and started via the las2peer Node Front-End:

* [las2peer User Information Service](https://github.com/rwth-acis/las2peer-user-information-service) In order to upload and start this service, please download its source code [here](https://github.com/rwth-acis/las2peer-user-information-service/releases/tag/v0.2.3), build it according to instructions and deploy it to the las2peer network
* [las2peer File Service](https://github.com/rwth-acis/las2peer-fileservice) In order to upload and start this service, please download its source code [here](https://github.com/rwth-acis/las2peer-FileService/releases/tag/v3.0.0), build it according to instructions and deploy it to the las2peer network
* [las2peer Contact Service](https://github.com/rwth-acis/las2peer-contact-service) This service is deployed, just like the others, as described above, however it additionally requires the creation of a user agent. This is also best achieved using the Node Front-End via the [Agent Tools](http://localhost:8080/las2peer/webapp/agent-tools) tab. The credentials have to match the data passed to the backend deployment (see `CONTACT_STORER_NAME` and `CONTACT_STORER_PW` in the table below).

#### Configuration parameters
The following parameters are passed as enviornment variables either to the respective container in the docker-compose file or the respective kubernetes deployment file.

| Variable Name | Default Value | Description |
| ------------- | ------------- | ----------- |
| `LAS2PEER_ETH_HOST` | - | Denotes the location of the ethereum blockchain and should generally be set to `ethereum:8545` |
| `COMMUNITY_TAG_INDEX` | 0xeB510FB89C25cc1Af61fE45B965b5Fe27F1BCBa0 | Location of the community tag registry on the blockchain (required by las2peer core) |
| `USER_REGISTRY_ADDRESS` | 0x0e8acA05A8B35516504690Fa97fAEa69bbAFf901 | Location of the user registry on the blockchain (required by las2peer core) |
| `REPUTATION_REGISTRY_ADDRESS` | 0xd35284B7644732094338559c13a5CE880D247D37 | Location of the reputation registry on the blockchain (required by las2peer core) |
| `GROUP_REGISTRY_ADDRESS` | 0x1A37393184eD5D0040521728cBbfc819e07E9d20 | Location of the group registry on the blockchain (required by las2peer core) |
| `SERVICE_REGISTRY_ADDRESS` | 0x4930DC85997124F6cFBe8fAE727EA69E9577BBBc | Location of the service registry on the blockchain (required by las2peer core) |
| `CONSENT_REGISTRY_ADDRESS` | 0xC58238a482e929584783d13A684f56Ca5249243E | See [service README](https://github.com/rwth-acis/hye-youtube-proxy#configurations-properties) |
| `CONTACT_STORER_NAME` | contactstorer | Name of the user created as part of the las2peer Contact Service deployment |
| `CONTACT_STORER_PW` | supersecretpassword | Password associated with the user created as part of the las2peer Contact Service deployment |
| `WEBCONNECTOR_URL` | http://localhost:8080/hye-youtube/ | See `rootUri` in [service README](https://github.com/rwth-acis/hye-youtube-proxy#configurations-properties) |
| `FRONTEND_URLS` | localhost:8000,hye-youtube.de | See [service README](https://github.com/rwth-acis/hye-youtube-proxy#configurations-properties) |
| `SLEEP_FOR` | 30 | Time in seconds to wait before deploying the smart contracts (i.e., the above described registries) to the ethereum blockchain (generally, does not need to be changed) |

### HyE - YouTube Recommendations
***Yet to be implemented***

### HyE - YouTube Frontend
A web frontend based on Google's [Lit Element](https://lit.dev/) providing access to the above described service's functionality.
The source code of the docker image can be found in the project's repository [here](https://github.com/rwth-acis/hye-youtube-frontend/blob/master/Dockerfile).
Make sure to set the `BASE_URL` environment variable to URL under which your deployment of the frontend will be available.
By default this value is set to *http://localhost:8081/*.

### Nginx
This container starts an nginx reverse proxy which manages the web paths to the respective service paths.
This is necessary as requests, sometimes authenticated through cookies, have to be made to the different sub-systems of the project which is only possible if these requests do not violate the [CORS policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS).
This containers configuration can be adjusted by editing the repsective conf file for docker-compose or the configmap file for the kubernetes deployment.

## Kubernetes
The files stored in the kubernetes directory contain the configurations used by the service instance deployed to the tech4comp cluster available under https://hye.tech4comp.dbis.rwth-aachen.de/.