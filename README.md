## Kubernetes
### Orchestration for containers

Kubernetes has the following main components:
* One or more **master nodes**
* One or more **worker nodes**
* Distributed key-value store, such as [`etcd`](https://github.com/etcd-io/etcd)

![alt text](https://prod-edxapp.edx-cdn.org/assets/courseware/v1/8f441b27101be805bc286e67adc671a2/asset-v1:LinuxFoundationX+LFS158x+2T2019+type@asset+block/Kubernetes_Architecture1.png "Kubernetes Architecture")

A **master node** is the brain behind all operations inside the cluster. It includes the **Control Plane**, which maintains a record of all the Kubernetes objects in the system and runs continuous control loops to manage those objects' state. These loops ensure that the state of the system will match what was configured by the cluster administrator. The cluster's state, all cluster configuration data is saved to `etcd`.

A master node has the following components:

* API server

    All administrative tasks are coordinated by the `kube-apiserver`. It receives the requests sent by users, validates and processes them. During processing, it reads the current state from `etcd` and updates `etcd` after the request finishes execution.

* Scheduler

    `kube-scheduler` assigns new objects (such as pods) to nodes. The scheduler obtains from `etcd` (through the API server) the resource usage data for each worker node in the cluster. It also obtains the new object's requirements from the API server and uses this information to assign objects to nodes. The scheduler also considers QoS requirements, data locality, affinity, anti-affinity, taints, etc.

* Controller managers

    Controller managers are control plane components running controllers to regulate the state of the Kubernetes cluster. Controllers watch-loop comparing the cluster's current state with the desired (configured) state.

    `kube-controller-manager` runs controllers responsible to act when nodes become unavailable. It ensures the pod counts are as expected, creates endpoints, service accoounts and API access tokens.

    `cloud-controller-manager` runs controllers responsible to interact with the cloud provider infrastructure when nodes become unavailable. It manages the cloud provider storage volumes, load balancing and routing.


* `etcd`

    Distributed key-value data store used to persist Kubernetes cluster state. New data is always appended to the store. Only the API server can communicate with `etcd`. Beside the cluster state, `etcd` also stores configuration details such as subnets, `ConfigMap`s, `Secrets`, etc.


A **worker node** has the following components:

* Container runtime

    Handles the containers. The most widely used runtime is `docker`.

* `kubelet`

    An agent that runs on each node and communicates with the Control Plane components on the master node. It receives Pod definitions from the API server and interacts with the container runtime on the node to run containers associated with the Pod. It also monitors the health of the Pod's running containers.
    It connects to the container runtime using the Container Runtime Interface (CRI).
    
    ![](https://prod-edxapp.edx-cdn.org/assets/courseware/v1/ab209f7c32ceb17ed43dcf6b66056cea/asset-v1:LinuxFoundationX+LFS158x+2T2019+type@asset+block/CRI.png)

    `kubelet` connects to the CRI shim to perform container and image operations. The CRI shim implements two services:

    * `ImageService` - responsible for all image-related operations.
    * `RuntimeService` - responsible for all Pod and container-related operations.

    An example of a CRI shim that uses `dockershim` to create and manage containers:
    ![](https://prod-edxapp.edx-cdn.org/assets/courseware/v1/aa11f8d767939eb27a989d12423e5ae6/asset-v1:LinuxFoundationX+LFS158x+2T2019+type@asset+block/dockershim.png)

* `kube-proxy`

    A network agent responsible for dynamic updates and maintenance of all network rules on the node. It abstracts details of Pods networking and forwards connection requests to Pods.
    
* Addons for DNS, Dashboard, cluster-level monitoring and logging
  
Networking has a critical role in containerized applications. The main challenges are:


* Container-to-container communication inside Pods.

    A container runtime creates an isolated network space for each container it starts. This isolated network space is called **network namespace** and is shared across containers or host operating system.
    
    When a Pod is started, a network namespace is created inside the Pod and all containers running inside the Pod will share that network namespace so they can talk to each other via `localhost`.

* Pod-to-Pod communication on the same node and across cluster nodes.

    Pods communicate without a need of implmenting NAT. Pods are treated as VMs on a network where each Pod receives an IP address (model called **IP-per-Pod**.)
    
    Containers inside the Pods share the network namespace and communicate with each other through `localhost`. Containers are integrated with the Kubernetes networking model through the use of the **Container Network Interface** (CNI) - supported by CNI plugins. The CNI enables plugins to configure networking for containers. A few popular plugins are `weave`, `calico`.

    ![](https://prod-edxapp.edx-cdn.org/assets/courseware/v1/e7f4f4c4b79ffd11fb57659d8558943b/asset-v1:LinuxFoundationX+LFS158x+2T2019+type@asset+block/Container_Network_Interface_CNI.png)

    
    The container runtime offloads the IP assignment to CNI => The CNI connects to the underlying configured plugin to get the IP address => CNI forwards IP address to requested container runtime.
    

* Pod-to-Service communication within the same namespace and across cluster namespaces.
* External-to-Service communication for clients to access applications in a cluster.

    Kubernetes enables external accessibility to the application over a virtual IP using **services** which are exposed through `kube-proxy`.


### Deploying Kubernetes

There are 4 types of deployments:

* All-in-One Single-Node Installation
* Single-Node `etcd`, Single-Master and Multi-Worker Installation
* Single-Node `etcd`, Multi-Master and Multi-Worker Installation
* Multi-Node `etcd`, Multi-Master and Multi-Worker Installation

We can use the following popular tools to deploy Kubernetes on `localhost`:
* `minikube`
* Docker Desktop

Here are a few popular tools to deploy a Kubernetes cluster on cloud environments and VMs:
* `kubeadm`
* `kubespray` - Based on `ansible`
* `kube-aws` - Deploy on AWS




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
`helm create myChart`