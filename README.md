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

    A network daemon responsible for dynamic updates and maintenance of all network rules on the node. It abstracts details of `Pod`s networking and forwards connection requests to `Pod`s. It watches the API server on the master node.
    
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


### Accessing Cluster

`kubectl` is a command line tool (CLI) to client to manage cluster resources and applications. 

We can also access the web-based UI to interact with the cluster. To retrieve the endpoint, run the command `kubectl config view | grep https | cut -f 2- -d ":" | tr -d " "`.

This is what the Kubernetes HTTP API space looks like:

![](https://prod-edxapp.edx-cdn.org/assets/courseware/v1/7ebcb514203a3af89dc1599625779c1f/asset-v1:LinuxFoundationX+LFS158x+2T2019+type@asset+block/api-server-space_.jpg)

We can run `kubectl proxy` to bind the Kubernetes API server to the loopback address. We can then run `curl http://127.0.0.1:8001` to get a list of APIs:

```bash
[ec2-user:node1@~]  curl 127.0.0.1:8001/
# Response
{
  "paths": [
    "/api",
    "/api/v1",
    "/apis",
    "/apis/",
    ...
  ]
  ...
}

[ec2-user:node1@~]  curl 127.0.0.1:8001/healthz
# Response
ok
```

### Kubernetes Object Model

The object model represents different persistant entities in the Kubernetes cluster. The entities describe:

* What containerized application we're running and on which node.
* Application resource consumption limits
* Policies attached to the applications such as restart, fault tolerance, etc.

Examples of popular Kubernetes objects are:
* `Pod`
* `ReplicaSet`
* `Deployment`
* `Namespace`
* `Service`

Objects are created using a configuration file in `yaml` format (`json`-format can also be ued but for the most we use `yaml`). An example `Deployment` object for an `nginx` server:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.11
        ports:
        - containerPort: 80
```

We set the desired state under the `spec` section which is submitted to the Kubernetes API server. The Kubernetes system manages the `status` section (where it stores the actual state of the object). The Kubernetes Control Plane attempts to match the object's actual state to the desired state.

`apiVersion` field specifies the API endpoint on the API server which we wish to connect to.
`kind` field specifies the type of object we want to create. It can be of type `Deployment`, `Pod`, etc.
`metadata` object holds some basic information such as the name and label of the object.
`spec` object includes the desired state of the object. In this case, we want 3 `Pod` running at any given time (see `replicas` field). The `Pod`s are created using the template set in the `spec.template` field. The `Pod`s creates a single container using the `nginx` image. 

Once the object is created, Kubernetes attaches the `status` field to the object.

### **`Pod`s** 
Smallest and simplest Kubernetes object. It represents a single instance of an application. A `Pod` contains 1 or more containers which:
* Are scheduled together on the same host with the `Pod`
* Share the same network namespace
* Have access to mount the same external storage (volume)

`Pod`s do not have the capability to self-heal and are ephermal (last for a very short time). They are usually used with controllers (e.g. `Deployment`, `ReplicaSet`, `ReplicationControllers`) which handle `Pod` replication, fault-tolerance, self-healing. 

Here's an example of a `Pod` running one `nginx` container:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.15.11
    ports:
    - containerPort: 80
```

### Labels

Key-value pairs attached to Kubernetes objects (`Pod`s, `ReplicaSets`).
They are used to organize and select a subset of objects.

**Label Selectors** are used by Controllers to select a subset of objects. There are 2 types of Selectors:

* Equality-based Selectors - use `=`, `==`, `!=` to select objects. For example, `app==mymicroservicename`
* Set-Based Selectors - use `in`, `notin`, `exists`, `does not exist`. For example, `app in (dev,qa)`.


### `ReplicationController`

Ensures that the specified number of replicas of a `Pod` are running at any given time. It is recommended to instantiate `Pod`s using `ReplicationController`s because if a `Pod` without being created using a `ReplicationController`, the `Pod` would not recycle. The default controller is a `Deployment` which configures a `ReplicaSet` to manage the `Pod`'s lifecycle.

### `ReplicaSet`

Next generation `ReplicationController` which supports both equality and set-based Selectors. It is more popular to use `Deployment` to manage `Pod` lifecycle because the `Deployment` by default configures a `ReplicaSet` which manages the `Pod`.


### `Deployment`

The `DeploymentController` is part of the `master` node's controller manager and ensures that the current state always matches the desired state. It also allows `rollouts` and `rollbacks` for seamless application updates and downgrades and manages its `ReplicaSet` for application scaling.
An example of a `rolling update` would be if an application version is updated in the `Deployment` from 1.0 to 1.1 using:

```bash
kubectl set image deployment myapp myapp=myapp:1.1
```

The `DeploymentController` will scale down the `ReplicaSet` `Pod`s that use version 1.0 and will create a new `ReplicaSet` with `Pod`s running 1.1 scaled up 3. The fact that the `ReplicaSet` running version 1.0 doesn't get deleted means that there's an option to `rollback` in case version 1.1 has issues.

To check the rollout history, we can use the following command:

```bash
kubectl rollout history deploy myapp

deployment.extensions/myapp
REVISION CHANGE-CAUSE
1        <none>
```

To get more details about a specific revision:

```bash
kubectl rollout history deploy myapp --revision=1
```
```yaml
Labels:   app=myapp
      pod-template-hash=85737cc
Containers:
  myapp:
    Image: myapp:1.0
...
```


### Namespace

Partitioned virtual sub-clusters.

We can get the namespaces by running:

```bash
$ kubectl get namespaces
NAME              STATUS       AGE
default           Active       11h
kube-node-lease   Active       11h
kube-public       Active       11h
kube-system       Active       11h
```

Kubernetes creates some default namespaces:
* `kube-system` - contains objects created by the Kubernetes system, mostly the control plane agents.
* `kube-public` - unsecured used for exposing public information about the cluster.
* `kube-node-lease` - used for node heartbeat data.
* `default` - contains the objects and resources created by administrators and developers. By default, when not setting the `-n $NAMESPACE` flag, commands will use this namespace.

### Authentication

Every API request reaching the API server goes through 3 stages before being accepted:

1) **Authentication** - Logs in a user.

    Kubernetes has two types of users:
    - Normal users: Managed outside the cluster (e.g. Google accounts)
    - Service accounts: Managed internally and created by the API server. They are tied to a specific namespace and mount the credentials to communicate with the API server as `Secrets`.
    * It's also possible to set up anonymous requests and user impersonation.

    Kubernetes uses different authentication modules:

    - Client certificates: By passing `--client-ca-file=/path/to/cert` option to the API server.
    - Static token: pre-defined bearer tokens with `--token-aut-file=/path/to/token` passed to API server.
    - 

2) **Authorization** - Authorizes the API requests added by logged-in user.

    Some authorization modules:
    - **Node Authorizer**: authorizes API requests made by `kubelet`s. It authorizes `kubelet`'s I/O operations for services, endpoints, nodes, etc.
    - **Webhook Authorizer**: Gives third-party services authorization permission. To enable this type of authorization, the API server needs to be started with `--authorization-webhook-config-file=/path/to/webhook/config`.
    - **Role-Based Access Control (RBAC) Authorizer**: Authorization based on user roles which restrict them to perform specific operations (`create`, `get`, etc.). There are two types of roles:
      - `Role`: Grant access to resources within a specific namespace.
      - `ClusterRole`: Same as `Role` but cluster-wide.

    For example, this creates a `Role` named `pod-reader` for a specific namespace with only some permissible operations:
    ```yaml
    kind: Role
    apiVersion: rbac.authorization.k8s.io/v1
    metdata:
      namespace: sisense
      name: pod-reader
    rules:
    - apiGroups: ["] # "" indicates the core API group
      resources: ["pods"]
      verbs: ["get", "watch", "list"]
    ```

    After the `Role` is created, we can associate users to roles using `RoleBinding`.

    There are two types of `RoleBindings`, `RoleBinding` and `ClusterRoleBinding`.

    In this example, we're associating a user with a role:
    ```yaml
    kind: RoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: pod-read-access
      namespace: sisense
    subjects:
    - kind: User
      name: sisense_guest
      apiGroup: rbac.authorization.k8s.io
    roleRef:
      kind: Role
      name: pod-reader
      apiGroup: rbac.authorization.k8s.io
    ```

    To enable this type of authorization, we need to run the API server with `--authorziation-mode=RBAC`.

    - **Attribute-Based Access Control (ABAC) Authorizer**: Kubernetes grants access to specific attributes. For example:


```json
{
  "apiVersion": "abac.authorization.kubernetes.io/v1beta1",
  "kind": "Policy",
  "spec": {
    "user": "sisense_guest",
    "namespace": "sisense",
    "resource": "pods",
    "readonly": true
  }
}
```
This `Policy` would allow the `sisense_guest` user to only read `Pod`s in the `sisense` namespace.

  To enable this type of authorization, the API server needs to be started with the following argument `--authorization-mode=ABAC`.

3) **Admission Control** - Software modules that can modify or reject the requests. They are used to specify granular access control policies. Examples include `ResourceQuota`, `AlwaysPullImages`, etc. To enable this option, we need to run the API server using `--enable-admission-plugins=NamespaceLifecycle,ResourceQuota,PodSecurityPolicy,DefaultStorageClass`. Kubernetes has some of these enabled by default.


### `Service`s

Services are used to group `Pod`s to provide common access points from the exterior to thje containerized application. This grouping is achieved using `Label`s and `Selector`s.

![Grouping of Pods using the Service object](https://prod-edxapp.edx-cdn.org/assets/courseware/v1/43deaf159772d06b10039d683640c244/asset-v1:LinuxFoundationX+LFS158x+2T2019+type@asset+block/Services2.png)

`Services` can expose:
- single `Pod`s, 
- `ReplicaSet`s, 
- `Deployment`s, 
- `DaemonSet`s
- `StatefulSet`s.

Here's an example of how the `Service` object definition looks like:

```yaml
kind: Service
apiVersion: v1
metadata:
  name: frontend-svc
spec:
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 80 # Service receives requests on this port
    targetPort: 5000 # Service routes requests to Pods on this port. If undefined, will use port instead.
```

In this example, we're grouping all `Pod`s with the `app==frontend` `Label` under a `Service` called `frontend-svc`. By default, an IP address (referred to as `ClusterIP`) is assigned to the service and is only routable inside the cluster. A client connects to the `Service` using the `ClusterIP` which forwards traffic to one of the `Pod`s associated with it. A `Service` includes a load balancer by default. The `Pod` IP address and `targetPort` are referred to as the `Service endpoint` (e.g. `10.0.1.4:5000`)

 `kube-proxy` watches the API server on the master node for the addition and removal of `Service`s and endpoints. For each `Service`, on each `node`, `kube-proxy` configures `iptables` rules to capture the traffic for the `ClusterIP` and forwards it to the `Service`'s endpoint. 

 #### `Service` Discovery

 Kubernetes supports two methods of discovering `Services`:
 
 * **Environmental Variables**: As soon as a `Pod` starts on any worker node, `kubelet` running on that node adds a set of environmental variables in the `Pod` for all actibe services. e.g., if we have a service called `gateway`, which exposes port 30999 and its `ClusterIP` is 172.17.0.1, then on any newly-created `Pod`, we will see the environmental variables:

```s
GATEWAY_SERVICE_HOST=172.17.0.1
GATEWAY_SERVICE_PORT=30999
GATEWAY_PORT=tcp://172.17.0.1:30999
GATEWAY_PORT_6379_TCP=tcp://172.17.0.1:30999
GATEWAY_PORT_6379_TCP_PROTO=tcp
GATEWAY_PORT_6379_TCP_PORT=30999
GATEWAY_PORT_6379_TCP_ADDR=172.17.0.1
```
* **DNS**: Kubernetes has a plugin for DNS which creates a DNS record for each Service and its format is `my-svc.my-namespace.svc.cluster.local`. Services within the same namespace find other `Service`s just by their name. 


#### `ServiceType`

We can define the access scope of a `Service` using `ServiceType`. The default is `ClusterIP` and is only accessible within the cluster.

* `NodePort`

  In addition to `ClusterIP`, another port (ranging from 30000-32767) is mapped to the `Service` from all worker nodes. 

  So if we map port 30845 to a `Service` and connect to any node on port 30845, the node would route all traffic to the assigned `ClusterIP`. 

  **We use `NodePort` when we want to make `Service`s accessible from the external world**.

* `LoadBalancer`:

  `NodePort` and `ClusterIP` are automatically created and the external load balancer will route to them.

  The `Service` is exposed as a static port on each worker node.

  The `Service` is exposed externally using the underlying cloud provider's load balancer configuration.

* `ExternalIP` 
  
  Not managed by Kubernetes.

  A `Service` can be mapped to an `ExternalIP` if it can route to one or more worker nodes. 

  Traffic that is ingressed into the cluster with the `ExternalIP` (as the destination IP) on the `Service` port gets routed to one of the `Service` endpoints. This requires an external cloud provider (Google Cloud Platform, AWS, etc.)


* `ExternalName`

  Has no `Selectors` and does not define any endpoints. 

  Used primarily to make externally-configured `Services` available to application inside the cluster.

