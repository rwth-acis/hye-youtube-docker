# HyE - YouTube Docker
This repository contains the deployment files for docker-compose and kubernetes of the HyE - YouTube Service, developed as part of the Master Thesis _``Implementing and evaluating recommendation sharing to increase diversity on YouTube''._

## Startup
The provided docker-compose should start all required components automatically.
To use it simply run `docker-compose up`.
However, their are a couple of additional setup steps needed:

### Create database and user
Once the MySQL database is up and running, exec into it with `docker exec -it hye-youtube-docker_mysql_1 /bin/sh`, start the mysql client via the command `mysql -p`, and enter the root password.
Then [create the database](https://www.mysqltutorial.org/mysql-create-database/) and [database user](https://www.digitalocean.com/community/tutorials/how-to-create-a-new-user-and-grant-permissions-in-mysql).

### Start missing services
Upon first startup, the blockchain is initialized and the smart contracts deployed.
This takes a while which causes the *l2pservices* and *recommendations* containers to fail, intern leading to the *frontend* container to not getting started.
Thus, after the las2peer node inside the *backend* container is up and running and the database has been set up, run the `docker-compose up` command again, to start the previously failed containers.

## Components
The entire service consists of four sub-systems (frontend, proxy service, recommendation service, and mllib) and the deployment of four additional components (ethereum blockchain, nginx reverse proxy, a MySQL database, and three additional las2peer services), which are briefly explained in the following.

### Ethereum blockchain
The deployment features one ethereum blockchain node which is built from [this Dockerfile](https://github.com/rwth-acis/las2peer-ethereum-cluster/blob/master/docker-images/monitored-geth-client/Dockerfile). Since las2peer in general and our service in particular makes use of smart contracts deployed to this blockchain, it should be started first.

### HyE - YouTube Proxy
The Proxy service is a [las2peer](https://github.com/rwth-acis/las2peer) service and as such its docker deployment starts a las2peer node which automatically attempts to connect to an ethereum blockchainlocated at **ethereum:8545** (both docker-compose and kubernetes implement a DNS to resolve this address to the container's actual IP address).
The container started as part of this deployment runs the docker image built from [this Dockerfile](https://github.com/rwth-acis/HyE-YouTube-Proxy/blob/master/docker/Dockerfile) in the respective repository.
The Proxy service is then automatically started together with the node and web connector, which by default is served under http://localhost:8081/las2peer.
*Uploading the service via the Node Frontend is not possible, due to a dependency which exceeds las2peer maximum allowed size of 5MB.*
Once the node has started, visit [/hye-youtube/init](http://localhost:8080/hye-youtube/init) to initialize the service.
***For more details regarding the configuration of this service, please refer to the [README](https://github.com/rwth-acis/hye-youtube-proxy#configurations-properties) contained in its repository.***

#### Configuration parameters
The following parameters are passed as environment variables either to the respective container in the docker-compose file or the respective kubernetes deployment file.

| Variable Name | Default Value | Description |
| ------------- | ------------- | ----------- |
| `LAS2PEER_PORT` | 9011 | Port connected to by las2peer |
| `LAS2PEER_ETH_HOST` | - | Denotes the location of the ethereum blockchain and should generally be set to `ethereum:8545` |
| `COMMUNITY_TAG_INDEX` | 0xeB510FB89C25cc1Af61fE45B965b5Fe27F1BCBa0 | Location of the community tag registry on the blockchain (required by las2peer core) |
| `USER_REGISTRY_ADDRESS` | 0x0e8acA05A8B35516504690Fa97fAEa69bbAFf901 | Location of the user registry on the blockchain (required by las2peer core) |
| `REPUTATION_REGISTRY_ADDRESS` | 0xd35284B7644732094338559c13a5CE880D247D37 | Location of the reputation registry on the blockchain (required by las2peer core) |
| `GROUP_REGISTRY_ADDRESS` | 0x1A37393184eD5D0040521728cBbfc819e07E9d20 | Location of the group registry on the blockchain (required by las2peer core) |
| `SERVICE_REGISTRY_ADDRESS` | 0x4930DC85997124F6cFBe8fAE727EA69E9577BBBc | Location of the service registry on the blockchain (required by las2peer core) |
| `CONSENT_REGISTRY_ADDRESS` | 0xC58238a482e929584783d13A684f56Ca5249243E | See [service README](https://github.com/rwth-acis/hye-youtube-proxy#configurations-properties) |
| `HYE_SERVICE_AGENT_NAME` | hyeAgent | Name of the las2peer service agent (the service attempts to create it upon container startup) |
| `HYE_SERVICE_AGENT_PW` | changeme | Password associated with the service agent |
| `WEBCONNECTOR_URL` | http://localhost:8080/hye-youtube/ | See `rootUri` in [service README](https://github.com/rwth-acis/hye-youtube-proxy#configurations-properties) |
| `FRONTEND_URLS` | localhost:8000,hye-youtube.de | See [service README](https://github.com/rwth-acis/hye-youtube-proxy#configurations-properties) |
| `SLEEP_FOR` | 30 | Time in seconds to wait before deploying the smart contracts (i.e., the above described registries) to the ethereum blockchain (generally, does not need to be changed) |

### las2peer services
The HyE - YouTube frontend requires the following additional services to function:

* [las2peer User Information Service](https://github.com/rwth-acis/las2peer-user-information-service)
* [las2peer File Service](https://github.com/rwth-acis/las2peer-fileservice)
* [las2peer Contact Service](https://github.com/rwth-acis/las2peer-contact-service)

These services are started by the l2pservices node which attempts to connect to a locally running blockchain node and a las2peer bootstrap node (the one, running the Proxy services).

#### Configuration parameters
The following parameters are passed as environment variables either to the respective container in the docker-compose file or the respective kubernetes deployment file.

| Variable Name | Default Value | Description |
| ------------- | ------------- | ----------- |
| `LAS2PEER_PORT` | 9011 | Port connected to by las2peer |
| `BOOTSTRAP` | - | Domain and port of running las2peer node to connect to, instead of starting a new network |
| `LAS2PEER_ETH_HOST` | - | Denotes the location of the ethereum blockchain and should generally be set to `ethereum:8545` |
| `CONTACT_STORER_NAME` | hyeAgent | Name of the las2peer service agent used to manage contact data |
| `CONTACT_STORER_PW` | changeme | Password associated with the service agent |

### HyE - YouTube Recommendations
The Recommendations service is a also a [las2peer](https://github.com/rwth-acis/las2peer) service starting a las2peer node and trying to connect to the blockchain.
This service's Dockerfile can be inspected [here](https://github.com/rwth-acis/hye-youtube-recommendations/blob/master/Dockerfile), in the respective repository.
Upon starting the container, and thus the las2peer node, the service is started automatically as well.
***For more details about this service, please refer to the [README](https://github.com/rwth-acis/hye-youtube-recommendations/blob/master/README.md) contained in its repository***

#### Dependencies
This service relies on multiple other services to properly function.
This includes a MySQL database and the Python MlLib service, which is also part of this docker-compose deployment, but additional setup steps are required, such as the creation of a Google OAuth client and API token.
For detailed information, please refer to the service's [README](https://github.com/rwth-acis/hye-youtube-recommendations/blob/master/README.md).

#### Configuration parameters
The following parameters are passed as environment variables either to the respective container in the docker-compose file or the respective kubernetes deployment file.

| Variable Name | Default Value | Description |
| ------------- | ------------- | ----------- |
| `LAS2PEER_PORT` | 9011 | Port connected to by las2peer |
| `BOOTSTRAP` | - | Domain and port of running las2peer node to connect to, instead of starting a new network |
| `LAS2PEER_ETH_HOST` | - | Denotes the location of the ethereum blockchain and should generally be set to `ethereum:8545` |
| `HYE_SERVICE_AGENT_NAME` | hyeAgent | Name of the las2peer service agent (has to be created beforehand) |
| `HYE_SERVICE_AGENT_PW` | changeme | Password associated with the service agent |
| `ML_LIB_URL` | http://localhost:8000/ | Location of Python MlLib server used to compute matrix factorization and word2vec embeddings |
| `MODEL_NAME` | HyE-MatrixFactorization | Name of the matrix factorization model created by the service |
| `MY_SQL_HOST` | localhost | Host domain of MySQL database |
| `MY_SQL_DATABASE` | hye | Database name in connected MySQL database |
| `MY_SQL_USER_NAME` | newuser | User name of MySQL database user used by service |
| `MY_SQL_USER_PW` | changeme | User password of MySQL database user used by service |
| `API_KEY` | - | Google API key required to retrieve YouTube video data |
| `CLIENT_ID` | - | Google API OAuth client ID required to retrieve personal access tokens for YouTube Data API |
| `CLIENT_SECRET` | - | Google API OAuth client secret associated with given ID |

### HyE - YouTube Frontend
A web frontend based on Google's [Lit Element](https://lit.dev/) providing access to the above described service's functionality.
The source code of the docker image can be found in the project's repository [here](https://github.com/rwth-acis/hye-youtube-frontend/blob/master/Dockerfile).
Make sure to set the `BASE_URL` environment variable to URL under which your deployment of the frontend will be available.
By default this value is set to *http://localhost:8081/*.

### Python MlLib
This small service performs the matrix factorization and word2vec embedding computations required by the HyE Recommendations service.
For this, a pre-trained word2vec model file with the proper format is required.
Details are provided in the service's [README](https://github.com/rwth-acis/hye-python-mllib/blob/master/README.md).
By default, the service is configured to use the model that can be downloaded [here](https://drive.google.com/file/d/0B7XkCwpI5KDYNlNUTTlSS21pQmM/edit?usp=sharing) which is then mounted into the directory as a volume.

By default, the server tries to bind to port `8000` which can be adjusted with the `SERVICE_PORT` environment variable.
Furthermore, the word2vec model location can be changed via the `MODEL_LOCATION` variable, and using the `MODEL_URL` variable, the docker container first tries to download the model from the provided URL and store it at the specified `MODEL_LOCATION`.

### Nginx
This container starts an nginx reverse proxy which manages the web paths to the respective service paths.
This is necessary as requests, sometimes authenticated through cookies, have to be made to the different sub-systems of the project which is only possible if these requests do not violate the [CORS policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS).
This containers configuration can be adjusted by editing the respective conf file for docker-compose or the configmap file for the kubernetes deployment.

## Kubernetes
The files stored in the kubernetes directory contain the configurations used by the service instance deployed to the tech4comp cluster available under https://hye.tech4comp.dbis.rwth-aachen.de/.
