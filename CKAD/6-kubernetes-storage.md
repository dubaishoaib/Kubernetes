# Kubernetes Storage

- **Container Layer:** Containers operate at the container layer, which is ephemeral and short-lived.
- **Container's Life:** A container's read-writeable layer is created during its lifecycle, and it's removed when the container is deleted.

### Using Volumes in Pods

- **Pod Volumes:** A pod can include volumes that provide storage beyond the container's lifecycle.
- **Volume Types:** Pod volumes can directly bind to specific storage types, such as NFS, for site-specific storage.

### Persistent Volume Claims (PVCs)

- **PVC Purpose:** To make pod specifications more portable and decouple them from site-specific storage.
- **PVC Definition:** A Persistent Volume Claim requests read-writeable or read-only storage of a specific size.
- **Administrator Role:** Site administrators ensure the existence of persistent volumes (PVs) based on PVC requirements.

### The Role of Persistent Volumes (PVs)

- **Persistent Volumes (PVs):** External storage resources available within a cluster.
- **PV Types:** A wide range of PV types can be defined based on the storage provider.
- **PVC-PV Matching:** When a PVC is created, it looks for an available PV that matches its storage requirements.
- **Dynamic Provisioning:** Storage classes and provisioners allow for automatic PV creation if a perfect match doesn't exist.

### Simplifying Storage Allocation with Storage Classes

- **Storage Classes:** Introduced to manage site-specific storage provisioning.
- **Automatic Provisioning:** Storage classes can dynamically create PVs based on PVC specifications.
- **Provisioners:** Provisioners associated with a storage class define how storage is allocated, such as AWS EBS or Azure Disk.

### Benefits of PVCs and Storage Classes

- **Decoupling Storage:** Using PVCs and storage classes decouples the pod specification from site-specific storage.
- **Portable Pod Manifests:** Developers can create generic pod manifests that don't include storage details.
- **Administrator Flexibility:** Site administrators can allocate suitable storage based on application requirements.

### Summarizing Storage Options

Here's a brief summary of the key storage options and concepts covered:

- Container storage is ephemeral, living only during a container's lifecycle.
- Pod volumes offer storage that outlives the container's lifespan.
- Volumes can directly bind to specific storage types.
- Persistent Volume Claims (PVCs) decouple pods from site-specific storage.
- Persistent Volumes (PVs) are external storage resources in a cluster.
- Storage classes automate dynamic PV provisioning based on PVC requirements.
- PVCs and storage classes enable portable pod manifests and administrator flexibility.

### Configuring Pod Local Volumes

- To configure pod local volumes, you define the volume within the pod specification.
- In the pod specification, a separate entry is dedicated to volumes, and the volume points to a specific volume type.
- Common volume types include `emptyDir` and `hostPath`, with `emptyDir` being temporary and dynamically created and `hostPath` being persistent and outlasting the pod's lifetime.
- A wide range of other storage types is available, such as cloud storage (e.g., AWS, Azure), local storage, and more.

### Mounting Volumes in Pods

- The volume is mounted within the `pod.spec.containers.volumeMounts` section of the pod specification.
- The container within the pod accesses the mounted volume, allowing access to site-specific storage.
- However, this scenario lacks portability since the storage type is defined in the pod specification, tying it to a specific site's storage.

#### Demonstrating Local Storage

- Use `kubectl explain` to explore various volume types available in the pod specification.
- Commonly used for testing purposes, `emptyDir` and `hostPath` are highlighted.
- `emptyDir`: A temporary directory created dynamically on the host running the pod.
- `hostPath`: A persistent directory defined, outlasting the pod's lifetime.
- Containers in a pod can access the same storage via these volume types.

### Demonstration YAML File
- A YAML file (`morevolumes.yaml`) is used to illustrate local storage usage.
- Two containers in a pod run CentOS images and access shared storage.
### Example YAML:
```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: morevol2
spec:
  containers:
  - name: centos1
    image: centos:7
    command:
      - sleep
      - "3600" 
    volumeMounts:
      - mountPath: /centos1
        name: test
  - name: centos2
    image: centos:7
    command:
      - sleep
      - "3600"
    volumeMounts:
      - mountPath: /centos2
        name: test
  volumes: 
    - name: test
      emptyDir: {}
```
#### Demonstrating the Process
- Create the pods using kubectl create -f morevolumes.yaml.
- Verify the pods' status with kubectl get pods more-vol.
- Describe the pod with kubectl describe pods more-vol to view the mounted volumes.
#### Accessing Local Storage
- Use kubectl exec -it more-vol -c centos1 -- touch /centos1/test to create a file on the emptyDir volume.
- Verify its existence by using kubectl exec -it more-vol -c centos2 -- ls /centos2.
Considerations and Limitations
- Pod local storage is useful but not portable across different sites due to site-specific storage dependencies.
- While emptyDir and hostPath are suitable for testing, they are not recommended for production environments due to their limitations.
```
$ kubectl explain pod.spec.volumes | less
$ nano morevolumes.yaml
$ kubectl create -f morevolumes.yaml
pod/morevol2 created

$ kubectl get pods morevol2
NAME       READY   STATUS              RESTARTS   AGE
morevol2   0/2     ContainerCreating   0          37s

$ kubectl describe pods morevol2 | less

$ kubectl exec -it morevol2 -c centos1 -- touch /centos1/test

$ kubectl exec -it morevol2 -c centos2 -- ls -l /centos2
total 0
-rw-r--r-- 1 root root 0 Aug 18 05:41 test
```

## Introduction to Persistent Volumes (PVs)

- A **persistent volume (PV)** is an independent resource in Kubernetes that connects to external storage.
- It operates separately from any pod in the cluster, providing decoupling and flexibility in storage management.
- PVs support various storage types similar to pod volumes, excluding `emptyDir`.

### Using Persistent Volumes

- To utilize a PV, you require a **persistent volume claim (PVC)**.
- The PVC binds to a PV, allowing pods to use the allocated storage.
- The PVC, in turn, dynamically maps to backend storage providers.

### Binding Process of PVC and PV

- When a PVC is created, it searches for an available PV matching the storage request, access modes, and capacity.
- Once a suitable PV is found, the PVC binds to it, establishing the connection to the external storage.

### Demonstrating the Creation of a PV

- The following YAML (`pv.yaml`) demonstrates how to create a persistent volume.

```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-volume
  labels:
      type: local
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mydata"
```
#### Exploring the YAML:
- capacity: Defines the available storage capacity (e.g., 1Gi).
- accessModes: Specifies access modes, including ReadWriteOnce, ReadOnlyMany, and ReadWriteMany.
- hostPath: Points to a directory created on the host (e.g., "/mydata").
#### Verifying PV Creation
- Use kubectl create -f pv.yaml to create the PV.
- Check its status with kubectl get pv.
- Detailed information can be obtained with kubectl describe pv pv-volume.

Upon PV creation, the host path directory (/mydata) is not immediately created on the host. It only gets created once the PV is accessed and utilized by a pod.
Remember that persistent volumes offer enhanced storage management in a cloud environment, ensuring the separation of storage resources from individual pods for improved flexibility and maintainability.

```
$ nano pv.yaml
$ kubectl create -f pv.yaml
persistentvolume/pv-volume created

$ kubectl get pv
NAME        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
pv-volume   2Gi        RWO            Retain           Available                                   3m43s

$ kubectl describe pv pv-volume
Name:            pv-volume
Labels:          type=local
Annotations:     <none>
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:
Status:          Available
Claim:
Reclaim Policy:  Retain
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        2Gi
Node Affinity:   <none>
Message:
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /mydata
    HostPathType:
Events:            <none>

$ minikube ssh
docker@minikube:~$ ls /
```

## Introduction to Persistent Volume Claims

- A **Persistent Volume Claim (PVC)** is used to request access to a Persistent Volume (PV) in Kubernetes.
- PVCs specify properties such as access modes (ReadWriteOnce, ReadOnlyMany, ReadWriteMany) and storage resources.
- The requested properties determine the PV that will be bound to the PVC.

### Binding Process of PVC and PV

- When a PVC is created, it binds to an available PV based on matching access modes and capacity.
- Once bound, the PVC establishes a one-to-one connection with the PV.
- The bound status indicates successful PVC-PV pairing.

### Efficient Resource Utilization

- A PVC's resource request should match the available capacity of a PV.
- An inefficient match can result in unused storage or wasted resources.
- A PVC will bind exclusively to a PV, preventing other PVCs from utilizing the same PV.

### Dynamic Provisioning with Storage Class

- If an exact match PV is not available, **Storage Class** may be used.
- Storage Class dynamically provisions PVs to match PVC requests.
- This allows for efficient resource allocation and avoids wastage.

#### Demonstrating PVC Creation and Binding

- The YAML file `pvc.yaml` demonstrates creating a PVC without connecting to a specific PV.

```yaml
```

#### Exploring the YAML:
- accessModes: Specifies access modes, similar to PVs (e.g., ReadWriteOnce).
- resources.requests.storage: Requests 1 gigabyte of storage.
#### Verifying PVC Behavior
- Use kubectl create -f pvc.yaml to create the PVC.
- Check its status with kubectl get pvc.
- Detailed information can be obtained with kubectl describe pvc pv-claim.
- Dynamic Creation of PV by Storage Class
- Depending on the environment, a matching PV may not exist for a PVC.
- Storage Class can dynamically create a PV based on PVC requirements.

This prevents PVCs from binding to PVs that do not efficiently match their storage needs.
By leveraging Persistent Volume Claims, Kubernetes workloads gain access to requested storage resources, enhancing portability and resource management.

```
$ nano pvc.yaml
$ kubectl create -f pvc.yaml
persistentvolumeclaim/pv-claim created

$ kubectl get pvc
NAME       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pv-claim   Bound    pvc-21dd3293-79d8-41d6-9b90-64d47e52d542   1Gi        RWO            standard       22s

$ kubectl get pv
$ kubectl describe pvc pv-claim
$ kubectl describe pv pvc-21dd3293-79d8-41d6-9b90-64d47e52d542
```
### Connecting Pods to Persistent Volumes using Persistent Volume Claims (PVCs)

1. **PV Parameters:**
   - Access Mode: ReadWriteOnce, ReadOnlyMany, etc.
   - Size: Specifies the capacity, e.g., 2GB.
   - Type: Specifies the storage type (usually local specifics).
   
2. **PersistentVolumeClaim (PVC):**
   - PVC requests access based on specific access mode and size.
   - A bound relationship is established if matching PV is available.
   
3. **Pod Specification:**
   - The pod defines the storage type as PVC.
   - Refers to the name of the PVC (e.g., mypvc).

4. **Mount Path:**
   - Specifies where the PVC will be mounted within the pod.
   - This information is defined in the pod's specification.

5. **Decoupling Site-Specific Information:**
   - Using PVCs decouples storage specifics from the pod specification.
   - Developers focus on pod and PVC manifests.
   - Site administrators handle storage, ensuring the right PV matches the PVC's requirements.
   
## Demo Steps

1. Create a PVC and a Pod with Matching PVC
   - YAML file: `pvc-pod.yaml`
   - PVC (`nginx-pvc`): Requesting 2GB, ReadWriteMany access mode.
   - Pod: Refers to the PVC and mounts it to `usr/share/nginx/html`.

2. Verify PVC Binding and PV Dynamic Creation
   - Run `kubectl create -f pvc-pod.yaml`.
   - Use `kubectl get pvc` to check PVC status.
   - Observe PVC `bound` to a dynamically generated PV.
   
3. Exploration and Verification of PV and PVC Relationship
   - Investigate PV with `kubectl describe pv pvc-<generated-id>`.
   - Demonstrate interaction: 
     - Use `kubectl exec` to create a test file inside the mounted PVC directory.
     - Access the host machine using `minikube ssh` to verify the file's presence.
     
By utilizing Persistent Volume Claims, Kubernetes developers can maintain portability and separation of duties. Pod specifications remain generic, while PVCs connect to site-specific storage, ensuring efficient and flexible resource allocation.

```
$ nano pvc-pod.yaml

$ kubectl create -f pvc-pod.yaml
persistentvolumeclaim/nginx-pvc created
pod/nginx-pvc-pod created

$ kubectl get pvc,pv

$ kubectl describe pv pvc-b744a098-8462-4bd0-bb47-212a3742b863

$ kubectl exec nginx-pvc-pod -- touch /usr/share/nginx/html/testfile
$ minikube ssh
docker@minikube:~$ ls -l /tmp/hostpath-provisioner/default/nginx-pvc/
total 0
-rw-r--r-- 1 root root 0 Aug 19 02:56 testfile
```

## What is StorageClass?

- **Cluster-Specific Storage Type:** Cluster environments often have specific storage types.
- **Automatic Provisioning:** Kubernetes StorageClass enables automatic provisioning of persistent volumes when a claim request comes in.
- **Storage Provisioner:** To use StorageClass, a storage provisioner in the backend is needed.
- **Internal and External Provisioners:** Kubernetes offers internal provisioners; external ones can be installed via operators.
- **Flexibility:** Provisioners make the mechanism flexible, enabling support for various storage types.
- **Minikube Example:** Minikube provides a built-in provisioner for local development.

### StorageClass as a Selector

- **Label and Selector Concept:** Similar to labels and selectors in deployments and services.
- **Binding Logic:** PVCs usually bind to PVs based on best match.
- **StorageClass as a Match:** When StorageClass is used in PV and PVC:
  - PVC only binds to a PV with the same StorageClass setting.
  - If matching PV is unavailable, the PVC remains in a pending state.
  
#### Demo Steps
1. **Using StorageClass as a Selector Label:**
   - Examine `pvc.yaml` configuration file.
   - Create PVC using `kubectl create -f pvc.yaml`.
   - Use `kubectl get pvc` to observe the PVC's binding to an auto-created PV.
   - Investigate the automatically created PV using `kubectl describe pv`.
2. **Automating Storage Creation and Binding:**
   - Review `pvc-pod.yaml` YAML file.
   - Create PV, PVC, and Pod using `kubectl create -f pv-pvc-pod.yaml`.
   - Verify the binding of the local PV claim to the local PV volume.
   - Examine the StorageClass assignment using `kubectl describe pv`.

StorageClass simplifies the provisioning and binding of persistent volumes by automating the process and providing a selector mechanism. This enhances flexibility and allows seamless allocation of storage resources to match specific requirements.
```
$ cat pvc.yaml
$ kubectl create -f pvc.yaml
$ kubectl get pvc
$ kubectl get pv
$ kubectl get storageclass
$ kubectl describe pv pvc-21dd3293-79d8-41d6-9b90-64d47e52d542
$ nano pv-pvc-pod.yaml
$ kubectl create -f pv-pvc-pod.yaml
$ kubectl get pvc
$ kubectl get pv
```

```
$ kubectl create -f pv-lab.yaml
persistentvolume/task-pv-volume created
persistentvolumeclaim/task-pv-claim created
pod/task-pv-pod created

$ kubectl get pods,pvc,pv

$ kubectl describe pod task-pv-pod

$ kubectl delete all --all
```