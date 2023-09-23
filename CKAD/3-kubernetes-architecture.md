# Kubernetes Architecture Overview

## Nodes
A Kubernetes cluster consists of different nodes. The control node is the most important node in the cluster, and there are multiple worker nodes.

## Components

### Container Runtime

A container runtime, such as Containerd, is required on both the control nodes and the worker nodes. Kubernetes is focused on containerization, and its core components also run as containers.

### Kubelet

The kubelet is a Kubernetes component that interfaces with the container runtime. It is present on all nodes and ensures that containers are scheduled properly.

### Controller Manager

The controller manager, referred to as c-manager, handles generic cluster operations. It runs on the control node.

### Kubernetes Database (etcd)

The Kubernetes database, called etcd, is responsible for storing cluster configuration and state information. It is also located on the control node.

## Interfacing with Kubernetes

To interact with Kubernetes, a client is needed. Typically, the kubelet serves as the client and communicates with the API service. Commands like kubectl are used to interact with Kubernetes. When a command is executed, the controller manager is involved in directing the action. The scheduler communicates with kubelets to determine the worker node for scheduling tasks. The etcd is updated with the port definition and status information.

```
                                 +------------------------+
                                 |                        |
                                 |     Control Node       |
                                 |                        |
                                 +------------------------+
                                 |    Container Runtime   |
                                 |          etcd          |
                                 |    kubelet             |
                                 |    Controller Manager  |
                                 +------------------------+
                                            ^
                                            |
                                            |
                                  +---------+---------+
                                  |                   |
                                  |   Worker Node     |
                                  |                   |
                                  +-------------------+
                                  |  Container Runtime|
                                  |  kubelet          |
                                  +-------------------+

```

# API Resources in Kubernetes

## Understanding API Resources

API resources in Kubernetes are essential for running applications. When using commands like `kubectl create deployment myapp --image=nginx --replicas=3`, you start the `myapp` application in Kubernetes. Each API resource adds functionality necessary for running applications in a cloud-native environment.

## Importance of API Resources

Cloud-native environments fundamentally change application behavior. In a standalone server setup, an application can fetch information from the local hard disk. However, in a cloud environment, accessing specific hard disks becomes impractical. API resources address this and other challenges, forming a critical part of Kubernetes understanding.

## Available API Resources

The Kubernetes API offers various resources. The main resources include:

- **Deployment**: Represents the application itself and provides key functionality.
- **ReplicaSet**: Manages scalability by handling application replicas (instances).
- **Pods**: Essential for running application containers in the cloud, offering features like pod priority, resource usage, and affinity.

Apart from Deployment, ReplicaSet, and Pods, Kubernetes provides numerous other resources. You can use `kubectl api-resources` to get an overview of available resources. The core Kubernetes API is versioned as V1, but Kubernetes API is extensible, enabling the availability of additional APIs.

## Essential API Resources

When running a container, Kubernetes manages it within a Pod. The Pod is the managed entity, housing related properties such as volumes for storage. Pods are typically replicated, and ReplicaSet manages their replication. However, Deployment is usually used to manage ReplicaSets, making it the application's representative entity.

Other components play important roles as well, including:

- **Service**: Exposes the application to users, allowing them to access it using an externally represented IP address.
- **ConfigMap**: Connects site-specific information, such as variables or secrets, to the application.
- **Persistent Volumes**: Handles storage for the application.

These API resources enable running applications in a cloud-native environment, where applications are not tied to specific servers. Instead, they retrieve information from the cloud, requiring suitable storage solutions.

```
  +------------------+     +-----------------------+     +--------------+
  |                  |     |                       |     |              |
  |     Container    |     |          Pod          |     |    Service   |
  |                  |     |                       |     |              |
  +------------------+     +-----------------------+     +--------------+
  |                  |     |                       |     |              |
  |     ReplicaSet   |-----|       Deployment      |-----|  ConfigMap   |
  |                  |     |                       |     |              |
  +------------------+     +-----------------------+     +--------------+

```
