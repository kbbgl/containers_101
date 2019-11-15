### Docker Compose v1

Following is an example of a multiservice deployment of 2 containers:

```yaml
# docker-compose.yml
web:
    build: . # context of the build. Will use the Dockerfile in current directory
    ports:
        - 5000:5000
    links:
        - mongo
mongo:
    image: mongo
```

If we then run `docker-compose build`,  `docker` will find the default `docker-compose.yml` file and use it to build the containers according to the instructions.

After building, we can run `docker-compose up` to start the application.

`docker-compose stop` will stop all running images.

`docker-compose rm` will delete all running images.


### Docker Compose v2
#### Docker Compose is a tool to define and run multi-container Docker applications

Has three layers:
1) Services
2) Networks
3) Volumes

```yaml
# docker-compose.yml
version: '2'
services:
    web:
        build: 
            context: .
            dockerfile: Dockerfile
        image: demoapp:0.1
    mongo:
        image: mongo
```

The network layer can help map out which networks the services will be connecting to:

```yaml
version: '2'
services:
    service1:
        image: ubuntu
        command: sleep 3600
        networks:
            - internal1
            - default
    service2:
        image: ubuntu
        command: sleep 3600
        networks:
            - internal1
    service3:
        image: ubuntu
        command: sleep 3600
        networks:
            - default
networks:
    internal1:
        driver: bridge 
```

So what happens in this case is that a network named `internal1` will be created which will based on the host machine network (`bridge` option) . `service1` will be able to ping a hostname called `service2` and `service3` while `service3` will not be able to reach `service2`.

The volumes layer helps us share volumes between containers:

```yaml
version: '2'
services:
    service1:
        image: ubuntu
        command: sleep 3600
        volumes:
            - data: /data
    service2:
        image: ubuntu
        command: sleep 3600
        volumes:
            - data: /data
volumes:
    data:
        driver: local
```

So in this set up, if we access `service1` and create a file `test.txt` in `/data`, this file will also appear in `service2` folder `/data/test.txt`.


## Kubernetes
### Orchestration for containers

Kubernetes has the following main components:
* One or more **master nodes**
* One or more **worker nodes**
* Distributed key-value store, such as [`etcd`](https://github.com/etcd-io/etcd)

![alt text](https://prod-edxapp.edx-cdn.org/assets/courseware/v1/8f441b27101be805bc286e67adc671a2/asset-v1:LinuxFoundationX+LFS158x+2T2019+type@asset+block/Kubernetes_Architecture1.png "Kubernetes Architecture")

A **master node** is the brain behind all operations inside the cluster. It includes the Control Plane, which maintains a record of all the Kubernetes objects in the system and runs continuous control loops to manage those objects' state. These loops ensure that the state of the system will match what was configured by the cluster administrator. The cluster's state, all cluster configuration data is saved to `etcd`.

A master node has the following components:

* API server
* Scheduler
* Controller managers
* `etcd`



`ConfigMap` - A container for storing configuration data. Other components (such as Pods) can access the data in the `ConfigMap`.




## Helm
### A package manager for Kubernetes

**Chart** - Packaged Kubernetes resource.

A chart contains two models:
* Template
* Values

**Chart repository** - Dockerhub but for `helm` charts.
**Release** - Deployed instance of a chart. Even if the source code wasn't modified, the release

#### Helm Components

Client- `helm client`  - Command line templating engine.

Tiller - Lives inside the cluster, manages releases, history and introspection.

To create a chart:
`heml create myChart`