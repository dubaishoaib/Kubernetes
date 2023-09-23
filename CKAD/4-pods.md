## Pods in Kubernetes

- A pod is an abstraction of a server that provides everything necessary to run an application.
- Pods in Kubernetes are analogous to servers in traditional IT infrastructure.
- Pods allow running multiple containers within a single namespace, exposed by a single IP address.
- Kubernetes manages containers through pods, adding features for running containers in a cloud-native environment.

### Disadvantages of Naked Pods:

- Naked pods are not rescheduled in case of failure.
- Rolling updates don't apply to naked pods, requiring downtime for updates.
- Naked pods cannot be easily scaled or replaced automatically.
- Running naked pods can cause connectivity loss during restarts.

### Running Pods:

- Pods are typically managed through deployments for better resiliency and manageability.
- Pods can be created using the `kubectl run` command, specifying the pod name and image.
- The `kubectl get pods` command provides an overview of running pods.
- The `kubectl describe` command offers detailed information about a specific pod, including containers and their properties.
- The `-o yaml` flag can be used to view the YAML code behind the pod configuration.

### Key Points:

- Pods provide a higher-level abstraction for managing containers in Kubernetes.
- Naked pods have limitations compared to managed pods through deployments.



# YAML Overview

- A human-readable data-serialization language.
- YAML is perfect for DevOps to create input files for managing objects in different systems.
- It is commonly found in solutions like Kubernetes and Ansible.
- YAML uses indentation to identify relations between elements.
- The advantage of YAML is its clear and defined structure.
- It is often used in combination with GitHub to manage and update configurations in a structured way.
- YAML ensures consistency in larger environments and is essential for DevOps.

## YAML Basics

- YAML files define Kubernetes resources and follow a specific structure.
- The file starts with the API version, indicating the Kubernetes API version it corresponds to.
- The "kind" field specifies the type of resource (e.g., deployment, pod).
- The "metadata" section contains administrative information about the resource, such as its name and namespace.
- The "spec" section is crucial as it contains the specific details for the object.
- Use `kubectl explain` to get information about these basic properties.

## YAML for Pods

- Container definition is important when working with pods in YAML.
- The YAML file includes several parts related to containers:
  - **name**: Specifies the name of the container.
  - **image**: Defines the container image to be used.
  - **command**: Specifies the command that the container should run if the image doesn't have a usable entry point.
  - **arguments**: Used by the command.
  - **env**: Specifies environment variables for the container.
- Multiple containers can be included in a YAML file using a list structure.
- While running multiple containers in one pod is not common, YAML allows for it.

## Example YAML File

The following example shows a YAML file for a pod:

```yaml
apiVersion: v1
kind: pod
metadata:
  name: example-pod
  namespace: default
spec:
  containers:
    - name: busybox
      image: busybox
      command:
        -  sleep
        -  "3600"
    - name: nginx
      image: nginx
```

- This example YAML file defines a pod with two containers: busybox and nginx.
- The busybox container runs the command sleep 3600 to sleep for an hour.
- The nginx container runs the default command to start the nginx service.

## Exploring YAML with kubectl explain
- Use the kubectl explain command to explore the properties of YAML resources.
- Specify the resource and field to get detailed information about different properties.
- Use the kubectl explain --recursive option to display all parameters in a nicely indented YAML format.
- Pay attention to the case sensitivity of options when specifying YAML properties.

## Managing YAML Files with kubectl
- YAML files can be managed using various kubectl commands.
- To create a resource from a YAML file: kubectl create -f <filename.yaml>.
- Use kubectl get pods to retrieve information about running pods.
- To delete a resource specified in the YAML file: kubectl delete -f <filename.yaml>.
- Use kubectl apply -f <filename.yaml> to create or update a resource based on the YAML file. It applies updates if the resource already exists.
- Note that kubectl apply is particularly useful for deployments with modifiable properties.

# Multi-Container Pods: Use Cases and Scenarios

- Single container Pods are standard and easier to build and maintain.
- Applications consisting of multiple containers are typically defined as microservices.
- There are some scenarios where multi-container Pods can be used:

```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: multicontainer
spec:
  containers:
  - name: busybox
    image: busybox
    command:
      - sleep
      - "3600" 
  - name: nginx
    image: nginx
```

## Sidecar Container

- Enhances the primary application for logging or monitoring.
- Istio service mesh is an example where sidecar containers are injected into Pods for traffic management.
- The main container and sidecar container have access to shared resources and often use shared volumes.

## Ambassador Container

- Represents the primary container to the outside world.
- Used when the primary container should not be directly reachable.

## Adapter Container

- Adapts traffic or data patterns to match other applications in the cluster.
- Useful for transcribing foreign data patterns into compatible formats.

## Considerations for Multi-Container Pods

- Sidecar containers, ambassador containers, and adapter containers are not defined by Pod properties but are valid use cases for multi-container Pods.
- When considering a multi-container Pod, assess if it matches the sidecar, ambassador, or adapter traffic pattern.
- If the use case does not fit these patterns, consider separating the containers into separate Pods as microservices for better management.

## Example: Sidecar Scenario

- Demonstrates how a sidecar container enhances the main application's functionality.
- In the example, the main application generates data, and a web server container presents the data.
- The main application writes data to a shared volume, which the web server container can access.

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: sidecar-pod
spec:
  volumes:
  - name: logs
    emptyDir: {}

  containers:
  - name: app
    image: busybox
    command: ["/bin/sh"]
    args: ["-c", "while true; do date >> /var/log/date.txt; sleep
10;done"]
    volumeMounts:
    - name: logs
      mountPath: /var/log

  - name: sidecar
    image: centos/httpd
    ports:
    - containerPort: 80
    volumeMounts:
    - name: logs
      mountPath: /var/www/html
```

```
$ kubectl create -f sidecar.yaml
$ kubectl exec -it sidecar-pod -c sidecar -- /bin/bash
$ yum install -y curl
$ curl http://localhost/date.txt
```
## Init Container

**https://kubernetes.io/docs/concepts/workloads/pods/init-containers/**

**https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-initialization/**

```
$ kubectl apply -f myapp-init-container.yaml
$ kubectl get -f myapp-init-container.yaml
$ kubectl describe -f myapp-init-container.yaml
$ kubectl logs myapp-pod -c init-myservice # Inspect the first init container
$ kubectl logs myapp-pod -c init-mydb      # Inspect the second init container
$ kubectl apply -f myapp-init-services.yaml
$ kubectl get -f myapp-init-services.yaml
```

```
$ kubectl apply -f pods-init-containers.yaml
$ kubectl get pods

NAME        READY     STATUS    RESTARTS   AGE
init-demo   1/1       Running   0          1m

$ kubectl exec -it init-demo -- /bin/bash
Defaulted container "nginx" out of: nginx, install (init)
root@init-demo:/# apt-get update
root@init-demo:/# apt-get install curl
root@init-demo:/# curl localhost
```

## Using NameSpaces

```
$ kubectl get all -A
$ kubectl create ns secret
$ kubectl run secretnginx --image=nginx -n secret
$ kubectl config view
$ kubectl get pods -n secret

NAME          READY   STATUS    RESTARTS   AGE
secretnginx   1/1     Running   0          3m39s

$ kubectl create -f busybox-ns.yaml

$ kubectl get pods -n secret
NAME          READY   STATUS    RESTARTS   AGE
busybox3      1/1     Running   0          16s
secretnginx   1/1     Running   0          22m

$ kubectl describe ns secret
Name:         secret
Labels:       kubernetes.io/metadata.name=secret
Annotations:  <none>
Status:       Active

No resource quota.

No LimitRange resource.

$ kubectl create ns -h | less
$ kubectl create ns production --dry-run=client -o yaml > ns-test-lab.yaml
$ kubectl create -f ns-test-lab.yaml
```

## Exploring Pod
### Summary: Exploring Kubernetes Resources with `kubectl describe` and `kubectl exec`

### Introduction
- `kubectl describe` and `kubectl exec` commands are powerful tools in Kubernetes for exploring and inspecting resources.

### `kubectl describe`
- Provides a human-readable view of information stored in the etcd database for a specific resource.
- Use `kubectl describe pod <pod-name>` to see all parameters and current settings for a Pod.

### `kubectl exec`
- Allows interaction with a running Pod by executing commands inside it.
- Connect to a Pod with `kubectl exec -it <pod-name> -- sh` to open an interactive shell.

### Viewing Pod Information in Different Formats
- Output of `kubectl describe` and `kubectl exec` can be displayed in JSON or YAML format.
- Use `kubectl get pod <pod-name> -o json` for JSON format, and `kubectl get pod <pod-name> -o yaml` for YAML format.
- JSON format provides information as it's stored in the etcd database but might not be the most readable.
- Use `kubectl explain pods.spec.enableServiceLinks` to get details about the `enableServiceLinks` option.
- This will provide details about enableServiceLinks, which is a boolean indicating whether information about services should be injected in pod environment variables.

```
$ kubectl get pods mynginx -o json | less
$ kubectl get pods mynginx -o yaml | less
$ kubectl explain pods.spec.enableServiceLinks
$ kubectl describe pods mynginx
```

### Troubleshooting with `kubectl describe` and `kubectl exec`
- `kubectl describe pod <pod-name>` provides a concise summary of the Pod's important information, including Events for troubleshooting.
- Use `kubectl exec -it <pod-name> -- sh` to open a shell inside the Pod and investigate issues.
- `kubectl describe` and `kubectl exec` are valuable tools for managing and understanding Kubernetes resources effectively.

## Troubleshooting Kubernetes Pods using `kubectl logs` and `kubectl describe`

- `kubectl logs` is used to access application output and troubleshoot Pods in Kubernetes.
- Pod main applications don't connect to standard output; hence, logs are sent to the Kubernetes cluster for inspection.
- Start troubleshooting failed applications by using `kubectl get pods` to check the status of the Pods.
- Use `kubectl describe pod <pod-name>` to get more information about a specific Pod's status.
- Look for the current state and last state in `kubectl describe` output for insights into any issues.
- If the main application generates a non-zero exit code, use `kubectl logs` to investigate further.
- Demonstration: Troubleshooting a failing application using 
  `kubectl run mydb --image=mariadb`.
- Observe Pod status with `kubectl get pods`, and if issues persist, use `kubectl describe pod <pod-name>`.
- Examine the "waiting" state and "CrashLoopBackOff" reason in the `kubectl describe` output.
- Pay attention to the "last state Terminated" information, specifically the "Error" reason and "exit code one."
- Use `kubectl logs <pod-name>` to access detailed information about the application errors.
- Resolve the issue by providing required environment variables, e.g., `--env MYSQL_ROOT_PASSWORD`.
- For production, use ConfigMaps and Secrets to manage environment variables securely.
- Finally, verify that the Pod is running and use `kubectl logs` to check for successful initialization.

```
$ kubectl run mydb --image=mariadb

$ kubectl get pods
NAME      READY   STATUS             RESTARTS      AGE
mydb      0/1     CrashLoopBackOff   4 (16s ago)   3m30s

$ kubectl describe pod mydb

  mydb:
    Container ID:   docker://df06404d759771b32823756b6b68541ce86afeafb32a3f77923371f5389a2d6d
    Image:          mariadb
    Image ID:       docker-pullable://mariadb@sha256:f94bb4868d953fed5220c9d3cdc8449f4c314efb07d3a18eefa6010b383f2ab8
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Error
      Exit Code:    1

$ kubectl logs mydb
[ERROR] [Entrypoint]: Database is uninitialized and password option is not specified
        You need to specify one of MARIADB_ROOT_PASSWORD, MARIADB_ROOT_PASSWORD_HASH, MARIADB_ALLOW_EMPTY_ROOT_PASSWORD and MARIADB_RANDOM_ROOT_PASSWORD

$ kubectl delete pods mydb
$ kubectl run --help | less
$ kubectl run mydb --image=mariadb --env MYSQL_ROOT_PASSWORD=password
$ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
mydb      1/1     Running   0          5s

```

## Summary: Pod Troubleshooting and Port Forwarding in Kubernetes

- Port forwarding is a troubleshooting method to access Pods in Kubernetes.
- It exposes a Pod's port on the local computer where `kubectl` is running.
- Port forwarding is only for troubleshooting purposes, not for regular user access.
- Regular user access to applications in Pods is achieved through Services and Ingress.
- To perform port forwarding, use the command:

```
$ kubectl port-forward <pod-name> 80:80
```

- The Pod's IP address is not directly accessible from outside the cluster.
- Port forwarding allows testing port accessibility using tools like `curl`.
- Remember to use port forwarding as a foreground process and terminate it with `Ctrl + C`.

```
$ kubectl get pods -o wide
$ kubectl run fwnginx --image=nginx
$ kubectl port-forward fwnginx 8080:80 &
curl localhost:8080
fg
Ctrl+C
```

## Understanding Kubernetes SecurityContext

### Introduction
SecurityContext in Kubernetes, a feature used to define privilege and access control settings for Pods and containers. It encompasses various options such as Discretionary Access Control, Security Enhanced Linux settings, defining privileged/unprivileged users, Linux capabilities, and AppArmor settings.

### Applying SecurityContext
- SecurityContext can be set at the Pod level or applied to individual containers within a Pod.
- Use `kubectl explain` on Pod specs to get a complete overview of available SecurityContext options.

### Troubleshooting with SecurityContext
- When a Pod fails to run due to SecurityContext settings, use `kubectl describe` to gather additional information from the events.
- Further analyze the issue by using `kubectl logs` on the failing Pod.

### Demonstrations
### Demo 1: Setting SecurityContext on a Pod and Container
- Illustrates the use of `runAsUser`, `runAsGroup`, and `fsGroup` on a Pod.
- Demonstrates the effect of `allowPrivilegeEscalation` setting on a container.

```
$ kubectl apply -f securitycontextdemo2.yaml
$ kubectl get all
$ kubectl exec -it security-context-demo -- sh
~ $ ps
PID   USER     TIME  COMMAND
    1 1000      0:00 sh -c sleep 1h
    7 1000      0:00 sh
   13 1000      0:00 ps
~ $ cd data
/data $ ls -l
total 4
drwxrwsrwx    2 root     2000          4096 Jul 30 13:51 demo
/data $ touch hh
touch: hh: Permission denied
/data $ cd demo
/data/demo $ echo hello > testfile
/data/demo $ ls -l
total 4
-rw-r--r--    1 1000     2000             6 Jul 30 13:56 testfile
/data/demo $ id
uid=1000 gid=3000 groups=2000,3000
```

### Demo 2: Handling Conflicts with SecurityContext
- Shows an example of setting `runAsNonRoot` on a Pod.
- Highlights a case where conflicts with the container image configuration lead to a `CreateContainerConfigError`.

```
$ kubectl apply -f securitycontextdemo.yaml

$ kubectl get all
NAME                        READY   STATUS                       RESTARTS   AGE
pod/nginxsecure             0/1     CreateContainerConfigError   0          27s

$ kubectl describe pods nginxsecure
Events:
  Type     Reason     Age                 From               Message
  ----     ------     ----                ----               -------
  Warning  Failed     15s (x8 over 108s)  kubelet            Error: container has runAsNonRoot and image will run as root (pod: "nginxsecure_default(43430a85-9bce-44fb-8eb9-98f43b202c94)", container: nginx)
  
```

### Cautionary Notes
- Be cautious when using SecurityContext to avoid conflicts with hard-coded settings in the container image.
- Some SecurityContext settings might not be compatible with the configuration of the container image.


## Understanding Kubernetes Jobs

### Introduction to Jobs
- Jobs in Kubernetes are designed for one-shot tasks that need to run to completion, unlike Pods that run continuously and get restarted on failure.
- The nature of Pods is to run forever, while Jobs are intended for finite tasks that do not require ongoing execution.

### Use Cases for Jobs
- Jobs are particularly useful for one-time operations, such as backup tasks, batch processing, or any task that needs to be executed once and marked as completed.
- When you need to perform a task that doesn't require continuous running, a Job is a suitable choice.

### Managing Jobs with `spec.ttlSecondsAfterFinished`
- The `spec.ttlSecondsAfterFinished` parameter allows you to set a time limit after which completed Jobs are automatically cleaned up.
- This feature prevents the accumulation of redundant information about completed Jobs and helps keep the cluster tidy.
- Jobs are generally short-lived, so cleaning up completed Jobs helps manage cluster resources efficiently.

### Types of Jobs
There are three types of Jobs based on the completions and parallelism parameters:

1. Non-parallel Job:
   - In a non-parallel Job, only one Pod is started, unless the Pod fails.
   - The default values for completions and parallelism are set to 1, which means one Job, one Pod.
   - Use `completions: 1` and `parallelism: 1` to explicitly define the non-parallel Job.

2. Parallel Job with Fixed Completion Count:
   - This type of Job runs multiple Pods in parallel, and the Job is considered complete after successfully running as many times as specified in `jobs.spec.completions`.
   - Use `completions` to specify the desired completion count and `parallelism` to define how many Pods can run concurrently.

3. Parallel Job with Work Queue:
   - Multiple Pods are started simultaneously, and the Job is marked as complete when any one of them finishes successfully.
   - Use `completions: 1` and `parallelism` set to the number of Nodes you want the Job to run on.

## Managing Pods in Jobs
- Each Job runs a Pod using the template specification provided in the Job spec.
- The template section in the Job spec includes metadata and spec sections that define the properties of the Pod managed by the Job.

## Demonstrations
- Use `kubectl create job` to create a Job and `kubectl delete jobs` to delete it along with associated Pods.
- Demonstrate `spec.ttlSecondsAfterFinished` to automatically clean up completed Jobs.

```
$ kubectl create job onejob --image=busybox -- date

$ kubectl get jobs
NAME     COMPLETIONS   DURATION   AGE
onejob   1/1           8s         15s

$ kubectl get jobs,pods
$ kubectl get jobs -o yaml | less

$ kubectl get jobs -o yaml | grep restartPolicy
        restartPolicy: Never
$ kubectl get jobs -o yaml | grep -B 5 restartPolicy
          name: onejob
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
        dnsPolicy: ClusterFirst
        restartPolicy: Never

$ kubectl delete jobs.batch onejob
job.batch "onejob" deleted        
```

```
$ kubectl create job mynewjob --image=busybox --dry-run=client -o yaml -- sleep 5 > mynewjob.yaml

$ kubectl create -f mynewjob.yaml

$ kubectl get pods,jobs
NAME                        READY   STATUS                       RESTARTS      AGE
pod/mynewjob-89v87          1/1     Running                      0             5s

NAME                 COMPLETIONS   DURATION   AGE
job.batch/mynewjob   0/3           5s         5s
```

## Understanding Kubernetes Cronjobs

### Introduction to Cronjobs
- Cronjobs are used for tasks that need to run on a regular schedule or at specified intervals.
- Unlike one-shot tasks handled by Jobs, Cronjobs are intended for recurring operations.

### Scheduling Cronjobs
- When you create a Cronjob, it schedules a Job that, in turn, starts a Pod to execute the task.
- To test a Cronjob without waiting for the scheduled time, you can use `kubectl create job myjob --from=cronjob/mycronjob`.
- This option triggers the Job associated with the Cronjob, allowing you to observe the task execution instantly.

### Demo: Creating and Testing a Cronjob
1. Create a Cronjob with the following command:

```
$ kubectl create cronjob -h | less
$ kubectl create cronjob runme --image=busybox --schedule="*/2 * * * *" -- echo greetings from the cluster
```

The above schedule represents "every second minute on every hour, every day of the month, every month, every day of the week."

2. Specify the command you want to run inside the Cronjob using the normal syntax, like `--echo "greetings from the cluster"`.

3. Check the created Cronjob using `kubectl get cronjobs`.

4. To manually trigger the Cronjob and test it, run:

5. Observe the created Job and the Pod it starts using `kubectl get jobs,pods`.

```
$ kubectl create job runme --from=cronjob/runme

$ kubectl get cronjob
NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
runme   */2 * * * *   False     0        52s             67s

$ kubectl get cronjobs,jobs,pod

$ kubectl get pods
NAME                    READY   STATUS                       RESTARTS      AGE
runme-28179038-kdrqq    0/1     Completed                    0             84s
```

6. To view the output of the executed command within the Pod, use:

```
$ kubectl logs runme-28179038-kdrqq
greetings from the cluster

$ kubectl delete cronjobs.batch runme
```

7. Once you are satisfied with the test results, delete the Cronjob using:

**Note:** Cronjobs are ideal for running recurring tasks, such as periodic backups or regular batch processing.

## Resource Limitations and Quota in Kubernetes

### Introduction
- limitations and quota in Kubernetes, which are essential for efficient resource management and workload scheduling.
- Resource requests and limits can be set as application properties to control CPU and memory usage for pods.
- By default, a pod will use as much CPU and memory as necessary, but you can limit it using `pod.spec.containers.resources`.

### Resource Requests and Limits
- **Requests**: Initial requests for resources that Kubernetes uses to schedule pods on suitable nodes.
  - Helps Kubernetes find a node with enough available resources to match the request.
  - Specifies the minimum resources required for a pod to run.
- **Limits**: Upper thresholds of resources that a pod can use.
  - Prevents pods from using more resources than specified, even if more resources are available.
  - Specifies the maximum resources allowed for a pod.

### Understanding CPU Limits
- CPU limits are expressed in millicores (millicpu), where 500 millicore is equal to 0.5 CPU core.
- Kubernetes kube-scheduler ensures nodes have the requested resources available to schedule pods.
- If a pod with resource limits cannot be scheduled, it will show a status of "Pending."
- CPU limits can be helpful in scenarios where you want to control the CPU consumption of specific pods.

### Memory Limits and Requests
- Memory limits and requests are expressed in bytes but can be specified with memory units (e.g., MB, GB).
- Memory requests define the amount of memory a pod initially requests to run.
- Memory limits define the maximum memory a pod is allowed to consume.
- Memory limits and requests are useful for managing memory-intensive workloads and preventing memory overconsumption.

### Demo 1: Setting Resource Limits
- Create a YAML file, `frontend-resources.yaml`, with CPU and memory resource limits for two containers: DB and WP.
- Test the YAML using `kubectl apply -f frontend-resources.yaml`.
- Observe the "OOMKilled" status due to insufficient memory (128MB limit) for the frontend container.
- Adjust the limits to a reasonable value (128MB request, 2GB limit) and re-run the demo to ensure the pod runs successfully.

```
$ kubectl apply -f frontend-resources.yaml
$ kubectl describe pods frontend
$ kubectl delete -f frontend-resources.yaml
$ kubectl apply -f frontend-resources-2.yaml

$ kubectl describe pods frontend
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  31s   default-scheduler  0/1 nodes are available: 1 Insufficient memory. preemption: 0/1 nodes are available: 1 No preemption victims found for incoming pod..

$ kubectl delete -f frontend-resources-2.yaml

$ kubectl apply -f frontend-resources-3.yaml

$ kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
frontend   2/2     Running   0          13s

$ kubectl delete -f frontend-resources-3.yaml
```

## Quota and Namespace Restrictions
- Quota is a restriction applied to namespaces, requiring applications to have resource requests and limits set.
- Create quota in a namespace using `kubectl create quota -n mynamespace`.
- Quota helps manage resources at the namespace level and ensures fair resource distribution among applications.

## Demo 2: Quota and Resource Limits
- Create a namespace `kubectl create ns restricted` to demonstrate quota and resource limits.
- Set quota on the namespace using `kubectl create quota mylimits -n restricted --hard=cpu=2,memory=1G,pods=3`.
- Attempt to run a pod in the restricted namespace without specifying resource limits (failed due to quota restrictions).
- Use deployments for better resource management and set resource limits using `kubectl set resources`.
- Resolve issues with limits within the quota boundaries to ensure successful pod scheduling.

```
$ kubectl create ns restricted
$ kubectl create quota -h | less

$ kubectl create quota mylimits -n restricted --hard=cpu=2,memory=1G,pods=3
resourcequota/mylimits created

$ kubectl describe ns restricted
Name:         restricted
Labels:       kubernetes.io/metadata.name=restricted
Annotations:  <none>
Status:       Active

Resource Quotas
  Name:     mylimits
  Resource  Used  Hard
  --------  ---   ---
  cpu       0     2
  memory    0     1G
  pods      0     3

No LimitRange resource.

$ kubectl run pods restrictedpod --image=nginx -n restricted
Error from server (Forbidden): pods "pods" is forbidden: failed quota: mylimits: must specify cpu for: pods; memory for: pods

$ kubectl create deploy restricteddeploy --image=nginx -n restricted
deployment.apps/restricteddeploy created

$ kubectl get all -n restricted
NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/restricteddeploy   0/1     0            0           30s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/restricteddeploy-74d5f7b997   1         0         0       30s

$ kubectl set resources deploy -h | less

$ kubectl set resources deploy restricteddeploy --limits=cpu=200m,memory=2G -n restricted
deployment.apps/restricteddeploy resource requirements updated

$ kubectl get all -n restricted
NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/restricteddeploy   0/1     0            0           17m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/restricteddeploy-74d5f7b997   1         0         0       17m
replicaset.apps/restricteddeploy-869775475b   1         0         0       47s

$ kubectl describe -n restricted deployments.apps
Conditions:
  Type             Status  Reason
  ----             ------  ------
  Available        False   MinimumReplicasUnavailable
  ReplicaFailure   True    FailedCreate
  Progressing      True    NewReplicaSetCreated
OldReplicaSets:    restricteddeploy-74d5f7b997 (0/1 replicas created)
NewReplicaSet:     restricteddeploy-869775475b (0/1 replicas created)

$ kubectl describe -n restricted replicaset restricteddeploy-869775475b
Events:
  Type     Reason        Age                  From                   Message
  ----     ------        ----                 ----                   -------
  Warning  FailedCreate  5m54s                replicaset-controller  Error creating: pods "restricteddeploy-869775475b-264wg" is forbidden: exceeded quota: mylimits, requested: memory=2G, used: memory=0, limited: memory=1G

$ kubectl set resources -n restricted deploy restricteddeploy --limits=cpu=200m,memory=128M --requests=cpu=100m,memory=64M
deployment.apps/restricteddeploy resource requirements updated

$ kubectl describe ns restricted
Name:         restricted
Labels:       kubernetes.io/metadata.name=restricted
Annotations:  <none>
Status:       Active

Resource Quotas
  Name:     mylimits
  Resource  Used  Hard
  --------  ---   ---
  cpu       100m  2
  memory    64M   1G
  pods      1     3

No LimitRange resource.

$ kubectl get all -n restricted
NAME                                    READY   STATUS    RESTARTS   AGE
pod/restricteddeploy-7f76887d97-h6hrg   1/1     Running   0          2m12s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/restricteddeploy   1/1     1            1           39m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/restricteddeploy-74d5f7b997   0         0         0       39m
replicaset.apps/restricteddeploy-7f76887d97   1         1         1       2m12s
replicaset.apps/restricteddeploy-869775475b   0         0         0       22m
```

**Note:** Understanding resource limitations and quota is crucial for efficient resource management and workload scheduling in Kubernetes. Properly setting resource requests and limits helps ensure stability and performance while avoiding resource exhaustion. Additionally, utilizing quota at the namespace level provides an effective way to manage resources across multiple applications in a Kubernetes cluster.

## Cleaning Up Resources in Kubernetes

### Introduction
- The importance of cleaning up resources in Kubernetes to ensure efficient resource utilization and prevent memory saturation in busy clusters.
- Resources in Kubernetes are not automatically cleaned up, necessitating periodic manual cleanup or using options like cron jobs for automatic cleanup.

### Considerations for Cleanup
- **Job Resources**: Some resources, like the Job, have options for automatic cleanup when they are no longer used.
- **Manual Cleanup**: For other resources, manual cleanup may be necessary to avoid resource saturation and potential cluster issues.
- **Managed Resources**: Be cautious while cleaning up resources managed by other components like deployments. Deleting the deployment should remove associated pods.

### Handling Resource Deletion
- **Force Option**: Use `kubectl delete` with the `--force` option to forcibly delete resources. Be extremely careful with this option as it can cause unmanageable states.
- **Grace Period Option**: The `--grace-period` option specifies the time to wait for graceful termination before forcefully deleting a resource.
- **Avoid --force --grace-period -1**: This powerful and dangerous combination forces deletion without waiting, leading to potential cluster disruption.

### Performing Cleanup
- Use `kubectl get pods -A` to list all pods in all namespaces.
- Use `kubectl delete all --all` to delete all resources within the current namespace.
- **Note**: Deleting the service "kubernetes" (part of the Kubernetes cluster itself) will be recreated automatically.

### Caution with Cleanup Commands
- Be cautious with forceful and immediate deletions, as it may cause resource/database issues.
- Avoid using `-A` option with cleanup commands, as it can lead to severe cluster problems.

**Important**: Properly cleaning up resources is crucial to maintain a stable and well-functioning Kubernetes cluster. Carefully use cleanup commands, avoid unnecessary forceful deletions, and follow best practices to ensure the stability and reliability of your Kubernetes environment.

```
$ kubectl delete all
error: resource(s) were provided, but no name was specified

$ kubectl delete all -all

$ kubectl get all -A

$ kubectl delete all --all --force --grace-period=-1
Warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
service "kubernetes" force deleted
```

```
$ kubectl create ns secret --dry-run=client -o yaml > lesson6lab.yaml

$ kubectl run secret-app --image=busybox --dry-run=client -o yaml -- sleep 3600 >> lesson6lab.yaml

$ kubectl explain pod.spec | less

$ kubectl create -f lesson6lab.yaml
namespace/secret created
pod/secret-app created

$ kubectl get pods -n secret
No resources found in secret namespace.

$ kubectl create -f lesson6lab.yaml
namespace/secret created
pod/secret-app created

$ kubectl get pods -n secret
NAME         READY   STATUS    RESTARTS   AGE
secret-app   1/1     Running   0          17s
```

**lesson6lab.yaml**
```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: secret
spec: {}
status: {}
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secret-app
  name: secret-app
  namespace: secret
spec:
  containers:
  - args:
    - sleep
    - "3600"
    image: busybox
    resources:
      requests:
        memory: "64Mi"
      limits:
        memory: "128Mi"
    name: secret-app
  dnsPolicy: ClusterFirst
  restartPolicy: OnFailure
status: {}
```

## Deployments in Kubernetes

### Introduction
- Deployments are the preferred way to run applications in Kubernetes due to their scalability and reliability features.
- Running naked pods is discouraged, as deployments offer various advantages.

### Advantages of Deployments
- **Scalability**: Deployments allow scaling the number of application instances to meet demand.
- **Updates**: Deployments facilitate zero downtime application upgrades by changing parameters, variables, and images.
- **Pod Protection**: Deployments automatically restart pods if any issues occur.

### Creating a Deployment
- To create a deployment, use `kubectl create deploy` followed by the deployment name and mandatory image argument.
- The `--replicas` option specifies the desired number of replicas for the application.
- Check the deployment using `kubectl describe deploy <deployment-name>`, and observe the pod template with its labels.

### Managed Pods in Deployments
- Deployments automatically manage and maintain pod instances.
- Replica Sets assist deployments in ensuring the desired number of pods is running.
- Deleting a pod within a deployment triggers the deployment to recreate it automatically.
- Running naked pods without a deployment lacks automatic management and won't recreate pods on deletion.

**Important**: Deployments provide essential features like scalability, zero-downtime updates, and automatic pod management. Always use deployments to ensure efficient and reliable application deployment and management in Kubernetes.

```
$ kubectl create deploy myweb --image=nginx --replicas=3
deployment.apps/myweb created

$ kubectl describe deployments.apps myweb
Name:                   myweb
Namespace:              default
CreationTimestamp:      Thu, **************
Labels:                 app=myweb
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=myweb
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=myweb
  Containers:
   nginx:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   myweb-5b7f574f7f (3/3 replicas created)
Events:          <none>

$ kubectl get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/myweb-5b7f574f7f-674w9   1/1     Running   0          114m
pod/myweb-5b7f574f7f-cv8ph   1/1     Running   0          114m
pod/myweb-5b7f574f7f-n8fs9   1/1     Running   0          114m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   3h18m

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/myweb   3/3     3            3           114m

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/myweb-5b7f574f7f   3         3         3       114m
```

## Deployment Scalability in Kubernetes

### Introduction
- Deployments manage scalability in Kubernetes, replacing the need for direct manipulation of ReplicaSets.
- In the original Kubernetes API, Replication Controllers were used for managing scalability.

### Evolution of Scalability Management
- With the introduction of the apps API extension, Deployments became the standard for scalability management.
- The Deployment resource creates a ReplicaSet under the hood, allowing for more advanced scalability features.
- Managed resources like ReplicaSets should be manipulated through their parent Deployments.

### Scalability Management Methods
- **Manual Scaling**:
  - Use `kubectl scale` followed by deployment name and desired number of replicas.
  - Alternatively, use `kubectl edit deployment <deployment-name>` to manually modify the number of replicas.

- **Horizontal Pod Autoscaler**:
  - Automatically adjusts the number of replicas based on CPU or custom metrics.
  - Ensures optimal resource utilization based on workload demands.

### Key Points to Note
- While `kubectl edit` seems powerful, it has limitations on which options can be modified.
- Deployments are managed through the apps API extension with versions like `apps/v1`.

### Practical Demonstration
- Create a deployment using a YAML file like `kubectl create -f redis-deploy.yaml`.
- Labels in metadata help selectors match pods and their related resources.
- Use `kubectl get all --selector app=redis` to focus on resources with the specified label.
- Deleting a ReplicaSet managed by a Deployment triggers automatic recreation of pods.

**Note**: Deployments provide controlled and flexible scalability. Combine manual scaling or the Horizontal Pod Autoscaler based on workload requirements.

```
kubectl api-resources | less

$ kubectl create -f redis-deploy.yaml
error: resource mapping not found for name: "redis" namespace: "" from "redis-deploy.yaml": no matches for kind "Deployment" in version "apps/v1beta1"
ensure CRDs are installed first

$ kubectl api-versions | less

$ kubectl create -f redis-deploy-2.yaml
deployment.apps/redis created

$ kubectl get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/redis-6dc855767d-r274x   1/1     Running   0          46s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   5h48m

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/redis   1/1     1            1           46s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/redis-6dc855767d   1         1         1       46s

$ kubectl get all --selector app=redis

$ kubectl delete rs redis-6dc855767d

$ kubectl get all --selector app=redis
```

## Application Updates and Deployments

### Introduction
- Application updates through deployments ensure zero downtime updates.
- Deployments enable changing image versions and other parameters like environment variables.

### Zero Downtime Application Updates
- Deployment updates involve creating a new ReplicaSet while keeping the old one.
- Pods with new properties are started in the new ReplicaSet.
- After a successful update, the old ReplicaSet can be deleted or kept for reversion.
- `revisionHistoryLimit` in deployment spec stores a set number of previous ReplicaSet versions.

### Update Strategy and Types
- Update strategy in deployments manages how updates are handled.
- **Rolling Update**:
  - Standard strategy for zero downtime updates.
  - Gradually replaces old pods with new ones.
- **Replace**:
  - Halts entire application and deploys a new version.
  - Used when application can't run multiple versions simultaneously.

### Practical Demonstration
- Create a new deployment using `kubectl create deploy nginexup --image=nginex:1.14`.
- Use `kubectl describe deploy nginexup` to verify image version in the container property.
- Use `kubectl get all --selector app=nginexup` to observe deployment and ReplicaSet status.

### Updating and Rolling Updates
- Use `kubectl set` for updates:
  - `kubectl set image deploy nginexup nginex=nginex:latest`.
- Observe the rolling update process:
  - Old and new ReplicaSets are managed simultaneously.
  - Pods are gradually replaced in the new ReplicaSet.
  - `kubectl get all` reveals progress of updates.
- Rolling updates ensure continuous application functionality during updates.

### Reverting to Previous Version
- The old ReplicaSet remains after a successful rolling update.
- This allows easy reversion to the previous application version for testing.
- Practical benefit in maintaining application stability and recoverability.

```
$ kubectl create deployment nginxup --image=nginx:1.14

$ kubectl describe deploy nginxup

$ kubectl get all --selector app=nginxup
NAME                           READY   STATUS              RESTARTS   AGE
pod/nginxup-859fb7489d-pwpm2   0/1     ContainerCreating   0          27s

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginxup   0/1     1            0           27s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/nginxup-859fb7489d   1         1         0       27s

$ kubectl set image deploy nginxup nginx=nginx:latest

$ kubectl get all --selector app=nginxup
NAME                           READY   STATUS    RESTARTS   AGE
pod/nginxup-575d7875dc-mcdb9   1/1     Running   0          15s

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginxup   1/1     1            1           3m34s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/nginxup-575d7875dc   1         1         1       15s
replicaset.apps/nginxup-859fb7489d   0         0         0       3m34s
```

## Labels, Selectors, and Annotations in Kubernetes

### Introduction to Labels, Selectors, and Annotations
- Labels: Key-value pairs providing additional info about Kubernetes resources.
- Used for grouping resources, connecting related resources (like pods, services), and more.
- Can be set manually or automatically.

### Labels and Selectors
- Labels connect resources and are used by selectors to find related resources.
- Deployments use labels and selectors to manage related pods.
- Services use labels and selectors to locate endpoints.
- Labels are crucial for identifying and managing resources.

### Working with Labels
- Labels allow grouping applications and are used in network policies.
- To find resources matching a specific label: `kubectl get <resource> --selector key=value`.
- Labels like `app=appname` and `run=podname` are automatically generated for deployments and pods.
- Labels are essential for tracking and managing pods within deployments.

### Annotations
- Annotations provide detailed metadata in objects.
- Originally had no real function, used for non-essential information.
- Recent Kubernetes resources use annotations for functional information as well.

### Practical Demonstration
- Create a deployment: `kubectl create deploy bluelabel --image=nginx`.
- Manually set a label: `kubectl label deployment bluelabel state=demo`.
- Check deployments with labels: `kubectl get deployments --show-labels`.
- Use labels with selectors: `kubectl get all --selector state=demo`.
- Labels set on deployment aren't inherited by resources it creates.
- Auto-created labels are crucial for connecting resources.

### Removing Labels
- To remove a label: `kubectl label pod bluelabel app-`.
- Removing labels can affect deployment's ability to manage pods.
- Removing a label led to a new pod being created due to the label mismatch.

### Conclusion
- Labels, selectors, and annotations play key roles in resource management.
- Labels facilitate grouping, selection, and connection of resources.
- Auto-created labels are crucial for the functioning of deployments.
- Labels and selectors are essential concepts for efficient Kubernetes resource management.

```
$ kubectl create deploy bluelabel --image=nginx

$ kubectl label deployment bluelabel state=demo

$ kubectl get deployments --show-labels
NAME        READY   UP-TO-DATE   AVAILABLE   AGE   LABELS
bluelabel   1/1     1            1           89s   app=bluelabel,state=demo
nginxup     1/1     1            1           46m   app=nginxup
redis       1/1     1            1           9h    app=redis

$ kubectl get deployments --selector state=demo
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
bluelabel   1/1     1            1           4m31s

$ kubectl get all --show-labels

$ kubectl get all --show-labels --selector state=demo
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE     LABELS
deployment.apps/bluelabel   1/1     1            1           8m13s   app=bluelabel,state=demo

$ kubectl describe deployments.apps bluelabel

$ kubectl label pod bluelabel-5cdb7645b5-fjdkk app-
pod/bluelabel-5cdb7645b5-fjdkk unlabeled

$ kubectl get all
NAME                             READY   STATUS    RESTARTS   AGE
pod/bluelabel-5cdb7645b5-fjdkk   1/1     Running   0          12m
pod/bluelabel-5cdb7645b5-l22lz   1/1     Running   0          29s

$ kubectl get all --show-labels
NAME                             READY   STATUS    RESTARTS   AGE     LABELS
pod/bluelabel-5cdb7645b5-fjdkk   1/1     Running   0          14m     pod-template-hash=5cdb7645b5
pod/bluelabel-5cdb7645b5-l22lz   1/1     Running   0          2m37s   app=bluelabel,pod-template-hash=5cdb7645b5
```

## Managing Deployment Update Strategy

### Introduction
- Deployment updates are crucial for maintaining application availability.
- Two strategies: Recreate and RollingUpdate.
- Recreate: Kills all Pods and creates new ones, causing temporary unavailability.
- RollingUpdate: Updates Pods one at a time, ensuring application availability.

### RollingUpdate Strategy
- RollingUpdate preferred for application availability during updates.
- Deployment ensures enough Pods are running at all times.
- Changed version deployed in a new ReplicaSet.
- After successful update, old ReplicaSet scaled to zero.
- Easy rollback by scaling newer ReplicaSet to zero and restoring older ReplicaSet.

## Managing Deployment Updates
- Use `kubectl rollout history` for details about recent updates.
- Use `kubectl rollout undo` to undo a previous change and restore old ReplicaSet.

## Fine-Tuning RollingUpdate Behavior
- Options to control availability during updates.
  - `maxUnavailable`: Max Pods unavailable during update.
  - `maxSurge`: Max Pods running beyond desired replicas for minimal availability.

## Practical Demonstration
1. Create a deployment: `kubectl create deploy bluelabel --image=nginx`.
2. Investigate strategy options: `kubectl get deploy bluelabel -o yaml`.
3. Edit options with `kubectl edit deploy bluelabel`.
4. Trigger an update using `kubectl set`.
5. Observe behavior: MaxUnavailable and MaxSurge options in action.
6. Check status using `kubectl get all --selector app=bluelabel`.

## Conclusion
- Deployment update strategy crucial for maintaining application availability.
- RollingUpdate strategy preferred, with options for fine-tuning.
- Use `kubectl rollout` commands for managing updates and rollbacks.
- Managing updates effectively ensures minimal disruption and reliable application performance.

```
$ kubectl get deployments.apps bluelabel -o yaml | less
..
.
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: bluelabel
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate

$ kubectl edit deploy bluelabel
  strategy:
    rollingUpdate:
      maxSurge: 4
      maxUnavailable: 2

$ kubectl scale deploy bluelabel --replicas=4
deployment.apps/bluelabel scaled

$ kubectl get all --selector app=bluelabel

$ kubectl set env deploy bluelabel type=blended
deployment.apps/bluelabel env updated

$ kubectl get all --selector app=bluelabel
```

## Managing Deployment History and Rollback

### Introduction
- Investigating how to manage deployment history and perform rollbacks.
- Deployment updates involve creating new ReplicaSets while keeping old ones with zero pods.
- `kubectl rollout history` command provides a history of deployment updates.
- Rollback can be performed using `kubectl rollout undo`.

### Using `kubectl rollout history`
- Use `kubectl rollout history deployment deployment-name` to see history.
- Shows revision details and changes for each update.
- No change logs shown if update doesn't use `--record` option.

### Performing Rollback
- Use `kubectl rollout undo` to revert to a previous version.
- Provides easy rollback in case of issues with updates.
- Useful options include checking history, restarting pods, resuming rollouts, and showing status.

### Practical Demonstration
1. Create a deployment from a YAML file, e.g., `kubectl create -f rolling.yaml`.
2. Use `kubectl rollout history` to view the history of the created deployment.
3. Use `kubectl edit deploy deployment-name` to make changes for an update.
4. Observe the changes using `kubectl describe deploy deployment-name`.
5. Perform a rollback using `kubectl rollout undo deployment deployment-name --to-revision=revision-number`.
6. Clean up by scaling down a deployment using `kubectl scale deploy deployment-name --replicas=0`.

### Understanding Deployment History
- Changes reflected in `kubectl rollout history` include actual changes in the application.
- Replicas changes (scaling) do not appear in the history as they are managed at a different level.

### Conclusion
- Managing deployment history crucial for tracking changes and performing rollbacks.
- Use `kubectl rollout history` to monitor updates and changes.
- Perform rollbacks using `kubectl rollout undo` in case of issues.
- Deployment history provides insights into changes affecting the application.

```
$ kubectl create -f rolling.yaml
deployment.apps/rolling-nginx created

$ kubectl get deploy
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
bluelabel       4/4     4            4           2d13h
nginxup         1/1     1            1           2d14h
redis           1/1     1            1           2d23h
rolling-nginx   3/4     4            3           49s

$ kubectl rollout history deployment
deployment.apps/bluelabel
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

deployment.apps/nginxup
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

deployment.apps/redis
REVISION  CHANGE-CAUSE
1         <none>

deployment.apps/rolling-nginx
REVISION  CHANGE-CAUSE
1         <none>

$ kubectl edit deployments.apps rolling-nginx
deployment.apps/rolling-nginx edited

$ kubectl rollout history deployment rolling-nginx
deployment.apps/rolling-nginx
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

$ kubectl describe deployments.apps rolling-nginx

$ kubectl rollout history deployment rolling-nginx --revision=2
deployment.apps/rolling-nginx with revision #2
Pod Template:
  Labels:       app=nginx
        pod-template-hash=7bd6589bc4
  Containers:
   nginx:
    Image:      nginx:1.15
    Port:       <none>
    Host Port:  <none>
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>

$ kubectl rollout history deployment rolling-nginx --revision=1

$ kubectl rollout -h | less

$ kubectl rollout undo deployment rolling-nginx --to-revision=1
deployment.apps/rolling-nginx rolled back

$ kubectl rollout history deployment bluelabel
deployment.apps/bluelabel
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

$ kubectl rollout history deployment bluelabel --revision=2
```

## Understanding DaemonSets in Kubernetes

### Introduction to DaemonSets
- DaemonSets are a type of deployment used to run a single pod instance on every node in the cluster.
- Designed for cases where specific software needs to be present on all cluster nodes, like network agents.
- Automatically adjusts the number of pods as nodes are added or removed.

### Use Cases for DaemonSets
- DaemonSets are ideal for multi-node clusters where software needs to run on all nodes.
- Ensures at least one pod instance is running on each node.

### Creating DaemonSets
- DaemonSets are created using YAML configurations, unlike regular deployments.
- No direct usage of `kubectl create` for DaemonSets.
- DaemonSet YAML configuration defines the pod template and matching labels.

### DaemonSet Configuration Example
- A DaemonSet configuration example is provided in the course Git Repository.
- YAML file named `daemon.yaml` demonstrates a simple DaemonSet.
- Configuration is similar to a deployment with differences in certain properties.

### Key Properties of DaemonSets
- DaemonSets have a `matchLabel` in the spec that matches the label in the template.
- Unlike deployments, DaemonSets do not specify a number of replicas.
- The unique aspect of a DaemonSet is that it ensures one pod per node.

### Demonstrating DaemonSets
1. Create the DaemonSet using `kubectl create -f daemon.yaml`.
2. Verify DaemonSet and associated pod using `kubectl get ds,pods`.
3. In the kube-system namespace, DaemonSets ensure system components on all nodes.

## Note on Spectacular DaemonSets
- DaemonSets' real impact shines in multi-node clusters.
- Their significance becomes evident when nodes are added or removed from the cluster.
- However, this demonstration might not be visible in a Minikube environment.

## Conclusion
- DaemonSets are crucial for ensuring specific software runs on every node.
- Use cases include network agents and other cluster-wide necessities.
- Configuration involves YAML files defining pod templates and labels.
- DaemonSets automatically adjust the number of pods as nodes change.


```
$ kubectl create -f daemon.yaml
daemonset.apps/nginxdaemon created

$ kubectl get ds,pods
NAME                         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/nginxdaemon   1         1         1       1            1           <none>          45s

NAME                                 READY   STATUS    RESTARTS   AGE
pod/nginxdaemon-6rkkr                1/1     Running   0          45s

$ kubectl get ds,pods -n kube-system
NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   30d
```

## Exploring Horizontal Pod Autoscaler in Kubernetes

### Introduction to Horizontal Pod Autoscaler (HPA)
- HPA is a resource in Kubernetes that automatically scales the number of pod replicas.
- It's based on resource metrics collected by the Metric Server.
- Essential for managing application scalability and resource efficiency.

### Using Horizontal Pod Autoscaler (HPA)
- While manual scaling with `kubectl scale` is important for CKAD, HPA is crucial for real clusters.
- HPA monitors resource usage and adds replicas when a threshold is crossed.

### Demo Setup
- The demonstration involves creating an application that performs heavy calculations.
- The application's Docker image is based on a custom PHP file running on Apache web server.
- The demo directory "AutoScaling" in the Git repo contains necessary files.

### Running the Demo
1. Create the deployment using `kubectl apply -f hpa.yaml`.
2. Set up HPA using `kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10`.
3. HPA will automatically scale the pods based on CPU usage exceeding 50%.
4. Verify HPA using `kubectl get hpa`.

### Metric Server Requirement
- HPA relies on Metric Server for collecting resource usage data.
- Metric Server setup is important for HPA to work effectively.

### Understanding HPA Behavior
- HPA is an API resource under autoscaling/v1.
- It adjusts the number of replicas based on defined thresholds.
- HPA targets are specified as a percentage of resource utilization.

### Running the Workload
1. Simulate workload using `kubectl run -it load-generator --rm --image=busybox -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"`.
2. Observe HPA's behavior as the application receives the load.
3. HPA will increase the number of replicas if resource utilization crosses the threshold.

### Cleaning Up
- Delete the load generator pod using `kubectl delete pod load-generator`.
- The HPA will adjust the replica count based on reduced load.
- The demo provides a basic understanding of how HPA operates.

## Conclusion
- Horizontal Pod Autoscaler (HPA) is essential for dynamic scaling of applications.
- It uses resource metrics collected by Metric Server to determine scaling needs.
- HPA helps maintain application performance while optimizing resource usage.

```
$ cd autoscaling/
$ docker build -t php-apache .

$ kubectl apply -f hpa.yaml
deployment.apps/php-apache created
service/php-apache created

$ kubectl autoscale -h | less

$ kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
horizontalpodautoscaler.autoscaling/php-apache autoscaled

$ kubectl api-resources | grep auto
horizontalpodautoscalers          hpa          autoscaling/v2         true         HorizontalPodAutoscaler

$ kubectl get hpa
NAME         REFERENCE               TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/50%   1         10        1          2m21s

$ kubectl describe hpa

$ minikube addons enable metrics-server

$ kubectl run -it load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"

$ kubectl get hpa
NAME         REFERENCE               TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   248%/50%   1         10        7          22h

$ kubectl get deployments.apps php-apache
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
php-apache   7/7     7            7           22h

$ kubectl delete pods load-generator
```

```
$ kubectl create deploy lab7nginx --image=nginx:1.9 --replicas=5 --dry-run=client -o yaml > lab7.yaml

$ kubectl explain --recursive deployment.spec | less

$ nano lab7.yaml
...
  strategy: 
    rollingUpdate:
      maxUnavailable: 2

$ kubectl create -f lab7.yaml

$ kubectl get all --selector app=lab7nginx
NAME                            READY   STATUS    RESTARTS   AGE
pod/lab7nginx-854bf75f5-62h9g   1/1     Running   0          10m
pod/lab7nginx-854bf75f5-6lsfh   1/1     Running   0          10m
pod/lab7nginx-854bf75f5-m2xkl   1/1     Running   0          10m
pod/lab7nginx-854bf75f5-qfr2b   1/1     Running   0          10m
pod/lab7nginx-854bf75f5-znrhb   1/1     Running   0          10m

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/lab7nginx   5/5     5            5           10m

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/lab7nginx-854bf75f5   5         5         5       10m

$ kubectl set image deploy lab7nginx nginx=nginx:latest
deployment.apps/lab7nginx image updated

$ kubectl get all --selector app=lab7nginx
NAME                            READY   STATUS              RESTARTS   AGE
pod/lab7nginx-57c87dd5d-287ll   0/1     ContainerCreating   0          3s
pod/lab7nginx-57c87dd5d-6qf5k   0/1     Pending             0          0s
pod/lab7nginx-57c87dd5d-f4mbv   1/1     Running             0          3s
pod/lab7nginx-57c87dd5d-fv8z6   0/1     ContainerCreating   0          3s
pod/lab7nginx-57c87dd5d-kzvkp   0/1     ContainerCreating   0          3s
pod/lab7nginx-854bf75f5-m2xkl   1/1     Running             0          12m
pod/lab7nginx-854bf75f5-qfr2b   1/1     Running             0          12m
pod/lab7nginx-854bf75f5-znrhb   1/1     Terminating         0          12m

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/lab7nginx   3/5     5            3           12m

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/lab7nginx-57c87dd5d   5         5         1       3s
replicaset.apps/lab7nginx-854bf75f5   2         2         2       12m
```
