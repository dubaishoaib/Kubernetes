## Resource Creation Flow

1. **kubectl Command Execution**:
   - Begin by using `kubectl` to run commands to create or manage resources.

2. **Request Forwarded to API Server**:
   - Your request is forwarded by the `kubectl` client to the Kubernetes API server.
   - The API server is the entry point for all operations in the cluster.

3. **API Server Writes to etcd**:
   - The API server writes the requested resources to the etcd database.
   - `etcd` is a distributed key-value store used as Kubernetes' primary datastore.

4. **Kube Scheduler Selection**:
   - Once the resources are successfully stored in `etcd`, the Kubernetes scheduler comes into play.
   - The scheduler selects an appropriate node (Kubernetes worker) to run your resource.

5. **Communication with Kubelet**:
   - The selected kubelet, a component running on the Kubernetes node, communicates with the container engine.
   - The container engine is responsible for running the containerized application.

6. **Container Execution**:
   - The container engine starts and runs the application within a pod.

7. **Troubleshooting**:
   - Troubleshooting can be done at different stages:
     - If issues occur during resource creation, `kubectl describe` provides insights into resource status.
     - For problems related to scheduling, use `kubectl describe` as well.
     - When the kubelet is running your application, its status can be checked.
     - If the application status is "off," use `kubectl logs` to inspect container logs.
       - Look for non-zero exit codes indicating issues.
     - `kubectl describe` is your primary troubleshooting tool.

#### Resource Creation Overview

- To summarize the resource creation process in Kubernetes:
   1. Use `kubectl create` or `kubectl run` to initiate resource creation.
   2. Resources are stored in the etcd database.
   3. The scheduler selects a suitable node for the resource.
   4. The container image is fetched.
   5. The container runs its entrypoint application.
   6. Based on the application's success or failure, the Pod Restart Policy is applied to decide further actions.


# Analyzing Failing Applications

In this section, we'll discuss how to analyze failing applications in Kubernetes and troubleshoot common issues.

- **Check Pod States:**
  - Use `kubectl get pods` to view the state of your pods.
  - Pods can be in different states:
    - **Pending:** Prerequisite conditions haven't been met; resource requests exceed available nodes.
    - **Running:** The pod is successfully running.
    - **Completed:** The pod has run to completion (usually seen with jobs) with a restart policy of "Never."
    - **Failed:** The pod has finished, but an issue occurred.
    - **CrashLoopBackOff:** The pod has failed, and Kubernetes restarted it.
    - **Unknown:** The pod status couldn't be obtained.

- **Use `kubectl describe`:**
  - After identifying an issue, use `kubectl describe` to investigate the application state.
  - Look at the events and application state.
  - Check the "last state" and exit code in the describe output.
    - If the exit code is zero, the application completed successfully.
    - If the exit code is non-zero, investigate using `kubectl logs`.

#### Scenario 1 - BusyBox Container

1. Create a pod using BusyBox: `kubectl create deploy failure1 --image busybox`.
2. Check the pod status: `kubectl get pods`.
3. Use `kubectl describe` on the pod: `kubectl describe pod failure1`.
4. Observe the "last state" and exit code. Exit code 0 indicates successful completion.
5. If exit code is non-zero, check logs using `kubectl logs`.


```
$ kubectl create deployment failure1 --image=busybox
$ kubectl get pods
NAME                        READY   STATUS             RESTARTS      AGE
failure1-6fd5c54fcd-6m95c   0/1     CrashLoopBackOff   3 (15s ago)   71s
$ kubectl describe pod failure1
....
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
....
$ kubectl delete deployments failure1
$ kubectl create deployment failure1 --image=busybox -- sleep 3600
$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
failure1-7c8f99fb77-9zlnw   1/1     Running   0          5s
```

#### Scenario 2 - MariaDB Container

1. Create a pod using MariaDB: `kubectl create deploy failure2 --image mariadb`.
2. Check the pod status: `kubectl get pods`.
3. Use `kubectl describe` on the pod: `kubectl describe pod failure2`.
4. Observe the "last state" and exit code. Exit code 1 indicates an issue.
5. Use `kubectl logs` to investigate the logs.
6. Fix issues (e.g., set environment variables) and verify using `kubectl get all`.

```
$ kubectl create deployment failure2 --image=mariadb
$ kubectl get pods
$ kubectl describe pod failure2
...
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Error
      Exit Code:    1
...
$ kubectl logs failure2-64c5c64d55-wgcfm
$ kubectl set env deploy/failure2 MYSQL_ROOT_PASSWORD=password
$ kubectl get pods
```

### Service Troubleshooting

- **Service Basics:**
  - Services are essential for accessing pods in Kubernetes.
  - Services load balance traffic between available pods.
  - Without a service, direct access to pods is not possible.

- **Service Selector Labels:**
  - Services use selector labels to match with the pods they target.
  - When services don't work as expected, check the selector labels.
  - Use `kubectl get endpoints` to check services and corresponding pod endpoints.

- **Service Nature:**
  - Understand the nature of your services.
  - ClusterIP services can't be accessed from outside the cluster.
  - Ensure you are on a node within the cluster to access ClusterIP services.

### Ingress Troubleshooting

- **Ingress Components:**
  - Ingress forwards traffic to pods and relies on service selector labels.
  - Ingress requires a configured and running Ingress controller.

### NetworkPolicy Troubleshooting

- **NetworkPolicies:**
  - NetworkPolicies restrict traffic to and from pods, namespaces, and IP ranges.
  - Check for existing NetworkPolicies: `kubectl get netpol -A`.
  - Use `kubectl describe` on NetworkPolicies for details.
  - Delete NetworkPolicies if they are causing issues.

#### Network Add-Ons
- Different network add-ons are available for Kubernetes.
- Changing a network add-on can fix or break network-related problems.
- For rich feature support, consider using the Calico add-on.

## Demo Example

1. Create a deployment using nginx: `kubectl create deploy trouble --image nginx`.
2. Expose the deployment as a NodePort service: `kubectl expose deploy trouble --port 80 --type NodePort`.
3. Verify that the service is working as expected.
4. Edit the service selector label to simulate an issue: `kubectl edit svc trouble`.
5. Observe that the service stops working.
6. Check the endpoints using `kubectl get endpoints` to confirm the issue.
7. Correct the label and verify that the service is accessible again.

```
$ kubectl create deploy trouble --image=nginx
$ kubectl expose deploy trouble --port=80 --type=NodePort
$ kubectl get endpoints
$ kubectl get svc
$ curl $(minikube ip):32354
$ kubectl edit svc trouble
$ curl $(minikube ip):32354
$ kubectl get endpoints
```
### Cluster Event Log and Node Troubleshooting

### Cluster Event Log

- **Cluster Event Overview:**
  - The cluster event log provides a general overview of cluster-wide events.
  - It's a valuable tool when you're unsure where to look for problems.
  - Use `kubectl get events` to view cluster-wide events.
  - For detailed cluster-wide event information, use `kubectl get events -o wide`.

- **Event Types:**
  - Events are categorized into types like "normal" or "warning."
  - Warnings indicate potential issues that need investigation.
  - Examples of warnings might include back-off restarting failed containers.

- **Event Details:**
  - Events contain information such as the type, reason, object, and message.
  - Filtering events by type (e.g., warnings) can help focus troubleshooting efforts.

#### Node Troubleshooting

- **Node Descriptions:**
  - Nodes in your cluster can play a significant role in troubleshooting.
  - Use `kubectl describe node <node-name>` to get a comprehensive overview of a specific node.
  - Node descriptions include critical conditions, status, and event history.

- **Node Conditions:**
  - Check the conditions section in node descriptions.
  - Conditions like memory pressure or disk pressure can indicate node-related issues.
  - Understanding node conditions is crucial for advanced node troubleshooting.

#### Demo Example

Let's explore a practical example of using `kubectl get events` and `kubectl describe node` to troubleshoot cluster-wide events and node conditions.

1. Use `kubectl get events` to view cluster-wide events and identify any warnings or issues.
2. Filter events to focus on warnings using `kubectl get events | grep warning`.
3. Use `kubectl describe node <node-name>` to examine a specific node's conditions, status, and event history.

```
$ kubectl get events
$ kubectl get events | grep warning
$ kubectl describe nodes minikube

```

### Authentication Configuration

- **Kubeconfig File:**
  - Access to a Kubernetes cluster is configured through a `~/.kube/config` file.
  - This file is commonly located at `~/.kube/config` on your local machine.
  - On a control node in the cluster, it's stored as `/etc/kubernetes/admin.conf`.

- **Viewing Kubeconfig:**
  - Use `kubectl config view` to check the contents of the kubeconfig file.
  - This command provides insights into the configuration of clusters, users, and contexts.

- **Authorization Verification:**
  - For additional authorization-related troubleshooting:
    - Use `kubectl auth can-i` followed by a verb and resource (e.g., `kubectl auth can-i create pods`).
    - This command provides a simple "yes" or "no" answer regarding your authorization to perform a specific action.

## Demo Example

Here's a practical example of troubleshooting an authentication issue:

1. Start in the `~/.kube` directory, where your kubeconfig file is located.
2. Move the `config` file to `~/.kube/config` (if it's not already there).
3. When experiencing an authentication problem (e.g., "The connection to the server was refused"), access the Kubernetes control host.
4. Use `minikube ssh` to SSH into the Minikube machine with a root shell (`sudo -i`).
5. Copy the `/etc/kubernetes/admin.conf` file to the `/tmp` directory and set permissions (`chmod 644`).
6. Exit the root shell and then exit the SSH session.
7. Use SCP to securely copy the `admin.conf` file from the Minikube machine to your local `~/.kube/config`.

By following these steps, you can resolve authentication issues and regain access to your Kubernetes cluster.

```
$ cd ~
~$ cd .kube/
$ mv config .config
$ kubectl get all
E0918 22:59:27.619540 1503432 memcache.go:265] couldn't get current server API group list: Get "http://localhost:8080/api?timeout=32s": dial tcp 127.0.0.1:8080: connect: connection refuseds

~/.kube$ minikube ssh
docker@minikube:~$ sudo -i
root@minikube:~# cp /etc/kubernetes/admin.conf /tmp
root@minikube:~# chmod 664 /tmp/admin.conf
root@minikube:~# exit
$ scp -i $(minikube ssh-key) docker@$(minikube ip):/tmp/admin.conf /home/shoaib/.kube/
$ kubectl get all
```

### Understanding Probes

- **What Are Probes?**
  - Probes are small tests that determine if a Pod is running correctly.
  - They are essential for preemptively identifying problems in your application.

- **Types of Probes:**
  - **ReadinessProbe:**
    - Ensures a Pod is not marked as available until it passes the readinessProbe.
  - **LivenessProbe:**
    - Continuously checks the availability of a Pod; if it fails, the Pod is restarted.
  - **StartupProbe:**
    - Used for legacy applications requiring additional startup time on initialization.

- **Probe Methods:**
  - Probes often use commands, but other methods include:
    - **Exec:** Executes a command in the container and expects a zero exit value.
    - **HttpGet:** Performs an HTTP request, expecting a response status between 200 and 399.
    - **TcpSocket:** Checks connectivity to a specified TCP port.

#### Practical Demonstrations

#### Demo 1: ReadinessProbe Example

1. Check the Kubernetes documentation or resources for in-depth knowledge about probes.
2. Create a readinessProbe example YAML file (e.g., `busybox-ready.yaml`) that includes a container with a readinessProbe.
3. Use `kubectl create -f busybox-ready.yaml` to create a Pod with the readinessProbe.
4. Observe the Pod status, which should show "NotReady" due to the failed readinessProbe.
5. Try to edit the Pod configuration to fix the issue (usually not possible).
6. Use `kubectl exec` to correct the problem within the running container (e.g., `kubectl exec -it busybox-ready -- touch /tmp/nothing`).
7. Observe the Pod returning to the "Ready" state.

#### Demo 2: ReadinessProbe and LivenessProbe for Nginx

1. Create an example YAML file (`nginx-probes.yaml`) with both readinessProbe and livenessProbe configurations targeting port 80.
2. Deploy the configuration with `kubectl create -f nginx-probes.yaml`.
3. Observe the Pod transitioning to the "Running" state as both probes succeed.
4. Understand that probes are essential for ensuring your Pods are not only running but also performing their expected functions.

By using probes effectively, you can improve the reliability and availability of your applications running in Kubernetes.

```
$ kubectl create -f busybox-ready.yaml
$ kubectl describe pods busybox-ready
$ kubectl exec busybox-ready -- touch /tmp/nothing
$ kubectl get pods
$ kubectl create -f nginx-probes.yaml
```