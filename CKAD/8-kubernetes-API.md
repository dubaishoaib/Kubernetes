# Exploring Kubernetes API

## Understanding the Role of Kubernetes API

The Kubernetes API serves as the foundation for managing and interacting with various resources within a Kubernetes cluster. It defines the available resources, their characteristics, and how they can be used.

- Resources: Kubernetes provides an array of resources, including pods, services, deployments, and more. These resources are managed and accessed through the API.

### Using `kubectl explain` to Explore API Definitions

- The `kubectl explain` command allows you to inspect the API definitions of specific resources.
- It reveals detailed information about the attributes, options, and configurations associated with each resource.
- This insight is crucial for effectively using and managing Kubernetes resources.

### Extending Kubernetes API

The core Kubernetes API is extendable through API groups and Custom Resource Definitions (CRDs).

#### API Groups for Extensibility

- The core API is further extended through API groups, allowing for modularity and organization.
- Different Kubernetes distributions might offer distinct API groups to accommodate their features and functionalities.

#### Custom Resource Definitions (CRDs) for Additional Resources

- CRDs empower users to introduce custom resources and APIs to Kubernetes.
- CRDs enable the creation of new resource types tailored to specific use cases and requirements.
- This extensibility enriches Kubernetes capabilities by incorporating domain-specific functionality.

### Operators and API Extensions

- Operators often leverage CRDs to extend Kubernetes' built-in capabilities.
- They provide a higher-level abstraction for managing complex applications.
- Operators encapsulate domain-specific knowledge, enabling the automation of intricate tasks.

## Accessing Kubernetes API

Kubernetes API access is a crucial aspect of cluster management and resource manipulation.

### Secure API Access with `kubectl proxy`

- The `kube-apiserver` is the core Kubernetes process that exposes cluster functionality.
- It ensures secure access through TLS certificate-based authentication and authorization.
- To interact with the API securely, `kubectl proxy` serves as a local intermediary.

### Using `kubectl proxy` for `cURL` Requests
Run the `kubectl proxy` command to start a local proxy server:
This displays available API paths and groups.

#### Secure Communication with kube-apiserver
- When using kubectl proxy, kube-apiserver ensures secure communication through TLS certificates.
- The local proxy forwards requests to kube-apiserver, maintaining the security of the communication.

```
$ kubectl api-resources | less
$ kubectl version
$ kubectl api-versions
$ kubectl --v=10 get pods
$ kubectl proxy --port=8001&
$ kubectl --v=10 get pods
```

#### Running `kubectl proxy` and Setting Up

- Begin by running the `kubectl proxy` command to establish a local proxy server that acts as an intermediary between your local machine and the Kubernetes API server:
- kubectl proxy --port=8001&
- The ampersand (&) allows the command to run in the background, freeing up your terminal for further actions.
- If you see that the proxy is already running on port 8001 (use kubectl get jobs), there's no need to start it again. You can continue with the subsequent steps.
- If the proxy is not running, initiate it with the following command:
- kubectl proxy --port=8001 &
#### Using cURL to Access API Resources
- Create a deployment to interact with using cURL:
  kubectl create deploy curl-demo --image=nginx --replicas=3
- Access Kubernetes version information using cURL:
  curl http://localhost:8001/version
- This cURL command retrieves and displays version information about the Kubernetes cluster. It's akin to running kubectl version.
- Fetch pod information from the default namespace using cURL:
  curl http://localhost:8001/api/v1/namespaces/default/pods
- Access pod-specific information using cURL:
  curl http://localhost:8001/api/v1/namespaces/default/pods/{pod_name}
- Delete a pod using cURL:
  curl -XDELETE http://localhost:8001/api/v1/namespaces/default/pods/{pod_name}

#### Authentication and Credentials
Authentication is automatically managed when using cURL with kubectl proxy. The credentials are derived from your kubeconfig file.

The kubeconfig file contains your certificates, which grant you the necessary permissions to perform administrative tasks on the Kubernetes cluster. It's the same set of credentials used when working with kubectl.

```
$ jobs
[1]-  Running                 kubectl proxy --port=8001 &
$ curl http://localhost:8001/version
{
  "major": "1",
  "minor": "26",
  "gitVersion": "v1.26.1",
  "gitCommit": "8f94681cd294aa8cfd3407b8191f6c70214973a4",
  "gitTreeState": "clean",
  "buildDate": "xxxxxxxxxxxx",
  "goVersion": "go1.19.5",
  "compiler": "gc",
  "platform": "linux/amd64"
}
$ curl http://localhost:8001/api/v1/namespaces/default/pods | less
$ curl http://localhost:8001/api/v1/namespaces/default/pods/curlginx-6cdc496cbc-cmbh4 | less
$ curl -XDELETE http://localhost:8001/api/v1/namespaces/default/pods/curlginx-6cdc496cbc-cmbh4
$ kubectl get pods
NAME                        READY   STATUS              RESTARTS   AGE
curlginx-6cdc496cbc-4kxlk   0/1     ContainerCreating   0          2s
curlginx-6cdc496cbc-hfvf9   1/1     Running             0          177m
curlginx-6cdc496cbc-wqpd7   1/1     Running             0          49s

~$ cat .kube/config
```
## API Deprecation and Impact

- Kubernetes releases occur approximately every three months, introducing new features and improvements.
- With each new release, some existing API versions may be marked as deprecated.
- Deprecation implies that the deprecated API version will be supported for a minimum of two more Kubernetes releases.
- Users need to be prepared for potential API changes and adapt their configurations and resources.

### Proactive Handling of Deprecations

- If a feature you are using gets deprecated, immediate action is necessary.
- Deprecation messages should not be ignored, as failing to address them might lead to issues later.
- Deprecations can have implications for your YAML files and resource configurations.

#### Handling API Deprecation - Demo

For demonstration, let's consider the scenario where an older API version is deprecated:

1. **Create a Deployment with Older API Version:**
   - Create a deployment using an older API version (e.g., `apps/v1beta1`).

2. **Observe Deprecation Error:**
   - Observe the error indicating that the API version is no longer supported.

3. **Verify Supported API Versions:**
   - Use the command: `kubectl api-versions` to verify supported API versions.

4. **Update Resource Configuration:**
   - Update your resource configuration to use a supported API version (e.g., `apps/v1`).
   - Consult the output of: `kubectl explain --recursive deploy` to understand property changes in the newer API version.

5. **Apply Changes:**
   - Apply the changes to your resource configuration to ensure compatibility.

#### Handling Deprecated Commands

1. **Use `kubectl exec` for Pod Access:**
   - Consider using the `kubectl exec` command for pod access.

2. **Notice Deprecation Message:**
   - Notice the deprecation message suggesting to use `kubectl exec pod -- command`.

3. **Adopt Recommended Approach:**
   - Adopt the recommended approach by using `kubectl exec pod -- /bin/sh`.

Understanding and managing API deprecations is vital to ensuring the longevity and stability of your Kubernetes deployments. Regularly review release notes, update your resource configurations to use supported API versions, and adapt to any deprecated commands to ensure seamless operations within the evolving Kubernetes landscape.

```
$ kubectl create -f redis-deploy.yaml
error: resource mapping not found for name: "redis" namespace: "" from "redis-deploy.yaml": no matches for kind "Deployment" in version "apps/v1beta1"
ensure CRDs are installed first

$ kubectl api-versions
$ kubectl explain --recursive deploy | less

$ kubectl exec -it curlginx-6cdc496cbc-4kxlk /bin/sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
```

### Authentication and Its Importance

- **Authentication**: Determines where Kubernetes users come from.
- By default, a local Kubernetes admin account is used.
- More advanced setups allow you to create custom user accounts.
- The `kubectl config` specifies the cluster for authentication.
- Use `kubectl config view` to view the current authentication settings.

## Role-Based Access Control (RBAC)

- **Authorization**: Defines what authenticated users can do.
- Role-Based Access Control (RBAC) manages permissions.
- RBAC uses three main elements:
  - **Role**: Defines permissions for specific resources.
  - **User / ServiceAccount**: Represents an entity interacting with the API.
  - **RoleBinding**: Connects a user or ServiceAccount to a Role.

## Demonstrating Current Authentication and Authorization

- Use the `kubectl config view` command to see the current configuration.
- The configuration includes clusters, contexts, and users.
- The `~/.kube/config` file holds this configuration.
- Authentication involves users, clusters, and contexts brought together.
- `kubectl auth can-i` command checks authorization status.
- For example: `kubectl auth can-i get pods` to verify permission.
- The `--as` option allows checking authorization as a specific user.
- Troubleshoot authentication-related issues using these commands.

```
$ kubectl config view
$ less ~/.kube/config
$ kubectl auth can-i get pods
yes
$ kubectl auth can-i get pods --as shoaib@MSI
```
### ServiceAccounts

- **ServiceAccount**: Addresses the need for authenticated and authorized actions in a Kubernetes Cluster.
- Used for basic authentication within the Kubernetes Cluster.
- Essential when a Pod requires information about other Pods in the same cluster.

#### Role-Based Access Control (RBAC) and ServiceAccounts

- **Role-Based Access Control (RBAC)**: Connects a ServiceAccount to a specific Role.
- RBAC and ServiceAccounts work in tandem to control authorization.

#### Default ServiceAccount and API Access

- By default, every Pod uses the `default` ServiceAccount to contact the API server.
- A `default` ServiceAccount exists in every namespace.
- Allows Pods to retrieve API server information.
- Limited permissions, even listing Pods isn't possible with the `default` ServiceAccount.

#### ServiceAccount Secrets

- **ServiceAccount Secrets**: Contain API credentials for ServiceAccounts.
- When specifying a ServiceAccount for a Pod, the corresponding Secret is auto-mounted.
- Provides API access credentials based on ServiceAccount RBAC connections.

#### Exploring ServiceAccounts

- View ServiceAccount information of a Pod using `kubectl get pods -o yaml | less`.
- `ServiceAccount` and `ServiceAccountName` refer to the ServiceAccount in use.
- `kubectl get sa` shows available ServiceAccounts.
- Many ServiceAccounts present in the `kube-system` environment for core components.
```
$ kubectl get pods
$ kubectl get pods curlginx-6cdc496cbc-4kxlk -o yaml | less
$ kubectl get sa
$ kubectl get sa -o yaml
$ kubectl describe pod curlginx-6cdc496cbc-4kxlk
$ kubect get sa -A
```

### Introduction to RBAC

- **Role-Based Access Control (RBAC)**: Configured by Cluster Administrators to specify access permissions.

#### Enhancing Access with Custom ServiceAccounts

- Custom **ServiceAccounts** and RBAC enhance access from Pods to cluster resources.
- Common use case: Application in the cluster needing information about other cluster applications.

#### Role and ClusterRole

- **Role** (for specific Namespaces) or **ClusterRole** (for the entire cluster) define access permissions to API resources.
- API groups, resources, and verbs define what actions can be taken on which resources.
- Example: A role allowing listing of pods can be defined.

#### RoleBindings and ClusterRoleBindings

- **RoleBinding**: Connects a ServiceAccount to a specific role.
- **ClusterRoleBinding**: Configures access to cluster resources.
- RoleBindings make sure the right ServiceAccount has the right permissions.

#### Example of Role and RoleBinding

- two example files: `list-pods.yaml` and `list-pods-mysa-binding.yaml`.

#### list-pods.yaml

- Role definition example.
- Specifies the Namespace, API groups, resources, and verbs for access.
- Lists the resources the role provides access to.
- For example, allows listing of pods.

#### list-pods-mysa-binding.yaml

- RoleBinding example.
- Establishes the connection between the role and the subject (ServiceAccount).
- `roleRef` points to the role, and `subject` points to the ServiceAccount.

#### Usage Demonstration

- Role and RoleBinding need to be created for actual usage.
- Run commands like `kubectl create -f list-pods.yaml` and `kubectl create -f list-pods-mysa-binding.yaml`.
- Create the associated ServiceAccount if necessary using `kubectl create sa mysa`.

```
$ kubectl create -f list-pods.yaml
role.rbac.authorization.k8s.io/list-pods created

$ kubectl create -f list-pods-mysa-binding.yaml
rolebinding.rbac.authorization.k8s.io/list-pods-mysa-binding created

$ kubectl get sa

$ kubectl create sa mysa
serviceaccount/mysa created
```

#### Demonstrate

- Demonstrate a comprehensive service account configuration.

#### Managing Service Accounts

- **Default Service Account**: Each namespace has a default service account named "default".
- **Additional Service Accounts**: Can be created for more resource access.
- Not advisable to enhance default service account privileges.
- Additional service accounts need permissions through RBAC (Role-Based Access Control).

#### Creating Additional Service Account

- Creating new service account: `kubectl create serviceaccount mysa`.
- Service account properties are largely defined through roles and role bindings.

#### Service Account Secret

- Upon creating a service account, an associated **Secret** is automatically generated.
- Secret connects the service account to the API, used as an access token.
- The secret is mounted in pods that use the respective service account.

#### Accessing Kubernetes API with Default Service Account

- Using `curl` to access the Kubernetes API.
- Unauthorized access attempt, user identified as anonymous.
- Default service account token needed for identification.

#### Enabling Access through RBAC

- RBAC involves defining roles and their bindings.
- Example: `list-pods.yaml` defines a role allowing pod listing.
- Role binding `list-pods-mysa-binding.yaml` associates the role with a custom service account.

#### Using the Custom Service Account

- Creating a pod with custom service account: `kubectl create -f mysa-pod.yaml`.
- Demonstrates proper authorization process from the pod.

```
$ kubectl apply -f mypod.yaml
$ kubectl get pods mypod -o yaml
$ kubectl exec -it mypod -- sh
/ # apk add --update curl
/ # curl https://kubernetes/api/v1 --insecure
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "forbidden: User \"system:anonymous\" cannot get path \"/api/v1\"",
  "reason": "Forbidden",
  "details": {},
  "code": 403
}
/ # TOKEN=$(cat /run/secrets/kubernetes.io/serviceaccount/token)
/ # echo $TOKEN
/ # curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/ --insecure
/ # curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/namespaces/default/pods/ --insecure
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "pods is forbidden: User \"system:serviceaccount:default:default\" cannot list resource \"pods\" in API group \"\" in the namespace \"default\"",
  "reason": "Forbidden",
  "details": {
    "kind": "pods"
  },
  "code": 403
}
/ # exit

$ kubectl create sa mysa
$ kubectl describe roles list-pods
Name:         list-pods
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  pods       []                 []              [list]

$ kubectl describe rolebindings.rbac.authorization.k8s.io list-pods-mysa-binding
Name:         list-pods-mysa-binding
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  list-pods
Subjects:
  Kind            Name  Namespace
  ----            ----  ---------
  ServiceAccount  mysa  default

$ kubectl create -f mysapod.yaml
pod/mysapod created

$ kubectl exec -it mysapod -- sh
/ # apk add --update curl
/ # TOKEN=$(cat /run/secrets/kubernetes.io/serviceaccount/token)
/ # curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1 --insecure
/ # curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/namespaces/default/pods/ --insecure
```

```
$ kubectl create sa testsalab
serviceaccount/testsalab created
$ kubectl get sa
$ kubectl run --help | less
$ kubectl run busysa --image=busybox -o yaml --dry-run=client -- sleep 3600 > bysysa.yaml
$ kubectl create -f bysysa.yaml
pod/busysa created
$ kubectl get pods busysa -o yaml | less
```

