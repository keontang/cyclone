## Workflow
![flow](flow.png)
- Cyclone provides abundant [APIs](http://118.193.142.27:7099/apidocs/) for web applications.
- After registering the code repository in VCS with Cyclone via API, commiting and releasing to VCS will notify Cyclone-Server by webhook.
- Cyclone-Server will run a Cyclone-Worker container which uses the “Docker in Docker” technique. The Cyclone-Worker container will checkout code from VCS, then execute steps according to the configrations of caicloud.yml in the code repository as follows:
 - PreBuild: compile the source code from VCS and generate the executable file in the specified system environment
 - Build: copy the executable file to the specified system environment, package the environment to a docker image and push the image to the specified docker registry
 - Integration: run the newly built image as a container, and bring up its dependencies (as other containers specified in the configuration) to perform integration testing.
 - PostBuild: run a container to execute some shells or commads which aim to do some related operations after the images is published in the registry
 - Deploy: deploy the containerized application into a containerized platform like Kubernetes.
- The logs durning the entire workflow can be pulled from Cyclone-Server via websocket
- Cyclone-Server will send the results and the complete logs of CI & CD workflow to users by email when the progress has finished


## Architecture
![architecture](architecture.png)


Each cube represent a container
- The API-Server component in Cyclone-Server provides the restful API service. If the task created by calling the API needs long time to handle, Cyclone will generate a pending event and write it into etcd
- The EventManger component loads pending events from etcd, watches the changes of events, and sends new pending events to WorkerManager
- WorkerManager calls the docker API to run a Cyclone-Worker container, and sends information to it via ENVs
- Cyclone-Worker uses event ID as a token to call the API server and gets event information, and then runs containers to execute integretion, prebuild, build and post build steps. Meanwhile, the workflow logs are pushed to the Log-Server and saved to kafka. 
- Log-Server component pulls logs from kafka and pushes the logs to users
- The data which need to be persisted are saved into  into mongo. 