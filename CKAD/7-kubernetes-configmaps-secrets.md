# Managing ConfigMaps and Secrets

## Managing Kubernetes Application Variables

- **No Direct Command-Line Option:** Kubernetes lacks a direct command-line option for passing variables during deployment creation.
- **Inconsistency:** Although the `kubectl run` or `kubectl create` commands provide such options for Pods, deployments require an alternate approach.

### Setting Environment Variables for Deployments

- **Two-Step Procedure:**
  1. Create the deployment: `kubectl create deploy mydb --image=mariadb`
  2. Set environment variable using: `kubectl set env deploy mydb MYSQL_ROOT_PASSWORD=password`
- **Working with YAML Files:**
  - Alternatively, use YAML files to define deployments and environment variables.
  - Create YAML file manually or generate it using `kubectl get deploy mydb -o yaml`.
  - Remove unnecessary auto-assigned metadata, status information, etc., from the generated YAML file.
  
### Working with Raw Pods (Not Recommended)

- **Direct Variable Provisioning for Pods:** Use the `kubectl run` command for deploying raw Pods.
- **Example:** `kubectl run mydb --image=mysql --env="MYSQL_ROOT_PASSWORD=password"`
- **Caution:** Direct use of naked Pods is discouraged in favor of Deployments.

#### Handling Initialization Errors

- **Understanding Errors:** Initialization errors may occur due to missing environment variables.
- **Investigation:** Use `kubectl describe pods mydb` to identify the specific error.
- **Example:** `database is uninitialized and password option is not specified.`

#### Dry Run and Environment Variables

- **Dry Run Considerations:** While `kubectl create` supports dry-run, dependency relations must be met.
- **Dry Run Example:** `kubectl create deploy newdb --image=mariadb --replicas=3 --dry-run=client -o yaml > mydb.yaml`
- **Dry Run Set Env Example:** `kubectl set env deploy newdb MYSQL_ROOT_PASS=password --dry-run=client -o yaml >> mydb.yaml`
- **Note:** Dry-run may lead to dependency issues with non-existent resources.

Effectively providing environment variables to Kubernetes applications requires careful consideration of the deployment process and the use of YAML files to ensure accurate configuration. While working with raw Pods is possible, employing Deployments is recommended for better management and scalability.

```
$ kubectl create deploy mydb --image=mariadb
deployment.apps/mydb created
$ kubectl get all
$ kubectl describe pod mydb-7978c6b495-vm8r4
$ kubectl logs mydb-7978c6b495-vm8r4
$ kubectl set env deploy mydb MARIADB_ROOT_PASSWORD=password
$ kubectl get deployments.apps mydb -o yaml > storage1-mydb.yaml
$ kubectl create deploy newdb --image=mariadb --replicas=3 --dry-run=client -o yaml > storage2-mydb.yaml
$ kubectl set env deploy newdb MARIADB_ROOT_PASSWORD=password --dry-run=client -o yaml >> storage2-mydb.yaml
Error from server (NotFound): deployments.apps "newdb" not found
$ kubectl create -f storage2-mydb.yaml
$ kubectl set env deploy newdb MARIADB_ROOT_PASSWORD=password --dry-run=client -o yaml >> storage2-mydb.yaml
```

## Decoupling Configuration with Kubernetes ConfigMaps

- **Importance of Decoupling:** Decoupling site-specific information from application code enhances portability and adaptability.
- **Static vs. Dynamic:** Code should be static for easy portability, while variables and configuration are site-specific and dynamic.
- **ConfigMaps:** Kubernetes provides ConfigMaps as a solution to store site-specific information separately from the application.

### Visualizing Decoupling with ConfigMaps

- **ConfigMap as a Bridge:** ConfigMap acts as a bridge between the application pod and cloud-native environment.
- **Cloud-Native Computing:** Cloud-native environments lack specific server relations, necessitating a centralized configuration storage.
- **ConfigMap Storage:** ConfigMaps are Kubernetes resources stored in the etcd database, encapsulating configuration data.

### How ConfigMaps Work

- **Pod Integration:** Pods integrate ConfigMaps using the `envFrom` directive to access environment variables.
- **Separation of Static and Dynamic:** ConfigMaps separate static application code from dynamic site-specific information.
- **One Application, Multiple ConfigMaps:** Each ConfigMap represents site-specific information, promoting a clean separation.

### Understanding ConfigMap Types

- **Three ConfigMap Types:** ConfigMaps are used for variables, configuration files, and command line arguments.
- **Variable and Configuration Use:** Variables and configuration files are the most common uses of ConfigMaps.
- **Choose the Right Type:** Depending on the application's requirements, select the appropriate type of ConfigMap.

### ConfigMap Deployment Considerations

- **Order of Creation:** Ensure the ConfigMap exists before deploying applications that require it.
- **Failure on Absence:** Unlike some resources, applications referring to missing ConfigMaps result in immediate failures.

Decoupling configuration from application code is a fundamental principle in cloud-native computing. Kubernetes ConfigMaps provide a structured and efficient way to achieve this separation, enhancing portability, scalability, and maintainability of your applications.

## Using Variables in Kubernetes ConfigMaps

ConfigMaps allow us to decouple configuration information from our applications, enhancing flexibility and portability.

- **Variable Decoupling:** Separating configuration variables from application code.
- **ConfigMaps:** Kubernetes ConfigMaps offer a way to store and manage configuration data.
- **Importance of Separation:** Keeping configuration separate ensures easier portability and adaptability.

### Creating ConfigMaps with Variables

- **Two Ways to Provide Variables:**
  - Use `--from-env-file` option to parse variables from a file.
  - Use `--from-literal` option for a more straightforward approach.
  - `--from-literal` can be used multiple times, while `--from-env-file` cannot.

#### Updating Applications with ConfigMap Variables

- **Using `kubectl set env`:**
  - Update application environment with ConfigMap variables.
  - Syntax can be slightly confusing, so use `kubectl set env -h` for help.
  - Use `kubectl set env` with the `--from` flag to reference the ConfigMap.

#### Generating YAML Files with ConfigMap Variables

- **Generating YAML Files:**
  - Use `--dry-run=client` and command line redirection.
  - Create YAML files with ConfigMap variables.
  - Ensure you've created the deployment before using `kubectl set env`.

#### Demonstration

- Create a variables file (e.g., `config-varsfile`) containing configuration variables.
- Use `kubectl create cm` to create a ConfigMap using variables from the file.
- Create a deployment (`mydb`) with `kubectl create deploy` specifying the image and replicas.
- Observe the failing deployment due to missing ConfigMap variables.
- Use `kubectl set env` to provide the ConfigMap variables to the deployment.
- Witness the rolling update and container creation process.
- Understand the YAML structure generated by `kubectl set env`.

Using variables within Kubernetes ConfigMaps is an essential practice to enhance the portability and adaptability of applications. By effectively decoupling configuration from application code, you ensure your applications are more versatile and easier to manage across different environments.

## Using ConfigMaps for Configuration Files in Kubernetes

ConfigMaps are an effective way to provide site-specific information and configuration files to applications while maintaining decoupling.

- **Configuration Files:** Used to supply site-specific information to applications.
- **Decoupling:** Configuration files should not reside within the application.
- **ConfigMap Solution:** Store configuration files in the cloud using ConfigMaps.

#### Creating ConfigMaps for Configuration Files

- **Creating from Files:**
  - Use `kubectl create cm myconf --from-file=name_of_file` to create a ConfigMap from a single file.
  - All files in the specified directory are included in the ConfigMap.
  - Ideal for including multiple configuration files.

#### Mounting ConfigMaps in Applications

- **Configuration Files as Volumes:**
  - Mount ConfigMaps in applications as volumes.
  - Behavior is similar to volumes.
  - Imperative way to mount ConfigMaps within applications is not available in Kubernetes.

#### Demonstration

- Create a configuration file (e.g., `index.html`) with necessary content.
- Use `kubectl create cm` to create a ConfigMap using the configuration file.
- Inspect the ConfigMap's details using `kubectl describe cm`.
- Create a deployment (`myweb`) using an image (e.g., nginx).
- Observe the running deployment and note the absence of specific configuration.
- Use `kubectl edit` to directly modify the running deployment.
- Add the ConfigMap mount information to the deployment YAML.
- Verify the successful update by checking the pod logs.
  
The Kubernetes documentation provides insights into mounting ConfigMaps as files.

- Create ConfigMaps from directories.
- Create ConfigMaps from files.
- Focus on adding ConfigMap data to a volume for mounting.
- Ensure you understand the distinctions between different ConfigMap uses.

Leveraging ConfigMaps for configuration files in Kubernetes applications provides a flexible and cloud-native solution to manage site-specific information. By mounting ConfigMaps as volumes, you ensure your applications remain adaptable and configurable while maintaining a separation between code and configuration.

```
$ nano config-varsfile
MYSQL_ROOT_PASSWORD=password
MYSQL_USER=test
$ kubectl create cm -h | less
$ kubectl create cm myconfig --from-env-file=config-varsfile
configmap/myconfig created
$ kubectl get all --selector app=mydb
$ kubectl set env deploy mydb --from=configmap/myconfig
$ kubectl get all --selector app=mydb
$ kubectl get deployments.apps mydb -o yaml | less
```

```
$ echo "hello world" > index.html
$ kubectl create cm myindex --from-file=index.html
$ kubectl describe cm myindex
$ kubectl create deploy myweb --image=nginx
$ kubectl get deploy --selector app=myweb
$ kubectl get all --selector app=myweb
$ kubectl edit deploy myweb
$ kubectl get all --selector app=myweb
```

```yaml
...
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: cmvol
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      volumes:
      - configMap:
          name: myindex
        name: cmvol
...

```

```
$ kubectl describe pods myweb-589445db67-p6rqr
$ kubectl exec myweb-589445db67-p6rqr -- cat /usr/share/nginx/html/index.html
hello world
```

## Kubernetes Secrets: Safeguarding Sensitive Data

- **Specialized ConfigMaps:** Secrets are a specific type of ConfigMap with added encoding.
- **Storage of Sensitive Data:** Designed for storing sensitive information like passwords, authentication tokens, TLS keys, SSH keys, etc.
- **Reduced Exposure Risk:** Secrets prevent the need to store sensitive data in clear text, reducing the risk of accidental exposure.
- **System-Created Secrets:** Kubernetes automatically generates some secrets crucial for its functioning, such as service account-related secrets.

### Types of Secrets

Kubernetes supports three main types of secrets:

1. **docker-registry Secret:**
   - Used for authenticating with Docker registries.
   - Also applicable to other container registries requiring authentication.

2. **TLS Secret:**
   - Utilized to store TLS key material, such as certificates and private keys.

3. **generic Secret:**
   - Equivalent to encoded ConfigMap.
   - Used for various scenarios, including encoded local files, directories, or literal values.

#### Creating Secrets

- **Defining Secret Type:** When creating a secret, explicitly specify the secret type using `kubectl create secret secret_type`.
- **Use the Correct Type:** Specify the appropriate secret type to avoid errors and ensure Kubernetes understands your intent.

#### Importance of Secrets

- **System Functionality:** System-created secrets are vital for Kubernetes system functionality.
- **Service Accounts:** Secrets are crucial for authenticating service accounts.

Kubernetes Secrets are a fundamental aspect of maintaining security and confidentiality in a Kubernetes environment. By providing an encoded and secure way to store sensitive data, they ensure that valuable information remains protected and inaccessible to unauthorized users.

## Kubernetes Secrets Internals

- **Internal Usage:** Kubernetes Secrets are essential for the proper functioning of Kubernetes components.
- **TLS Key Requirement:** All Kubernetes resources require access to TLS keys to communicate within the cluster.
- **Authorization and Communication:** TLS keys authorize interactions between Kubernetes components, pods, backend services, and more.

### Role of Secrets and ServiceAccounts

- **TLS Key Provision:** Secrets are responsible for providing TLS key material.
- **ServiceAccounts:** ServiceAccounts provide an identity to pods and other resources.
- **Authorization:** Through ServiceAccounts, pods can access their associated Secrets.

#### Investigating CoreDNS's Usage of Secrets

1. Use `kubectl get pods -n kube-system` to list CoreDNS pods in the kube-system namespace.
2. Inspect the CoreDNS pod's YAML format to find the associated ServiceAccount.
3. The ServiceAccount ("CoreDNS") acts as a user account with specific credentials for fetching Kubernetes information.
4. Run `kubectl get sa -n kube-system coredns -o yaml` to retrieve details about the CoreDNS ServiceAccount.
5. The ServiceAccount is associated with the Secret "coredns-token-gqblc."
6. Execute `kubectl get secret -n kube-system coredns-token-gqblc -o yaml` to view the contents of the Secret.
7. The Secret contains fields like `ca.crt`, `namespace`, and `token`.
8. The Secret's encoded nature can be decoded using tools like `base64 -d`, revealing its actual content.

#### Security Considerations

- **Encoded Format:** Secrets are encoded for security but can be decoded with proper access.
- **Real-World Security:** In actual secure Kubernetes setups:
  - Access to Secrets requires administrator login.
  - Secrets are stored in etcd with restricted access.
  - Access to etcd necessitates Linux root privileges.

Understanding how Kubernetes leverages Secrets internally to manage sensitive data and TLS key material is crucial for maintaining secure communication and authentication within the Kubernetes ecosystem.

```
$ kubectl get pods -n kube-system
$ kubectl get pods -n kube-system coredns-787d4945fb-fmjmv -o yaml | less
$ kubectl get sa -n kube-system coredns -o yaml | less
$ kubectl get secret -n kube-system coredns-token-gqblc
```

## Using Kubernetes Secrets in Applications

#### Secret Use Cases

- **TLS Key Provision:** For providing TLS keys to applications.
- **Password Security:** For storing sensitive passwords securely.
- **SSH Key Management:** Offering secure access to SSH private keys.
- **Docker Registry Access:** Supporting authentication for Docker registries.
- **Sensitive File Access:** Allowing applications access to sensitive files.

#### Configuration Steps for Different Use Cases

#### TLS Key Provision

1. Use `kubectl create secret tls` with:
   - Name of the Secret
   - `--cert` flag for the certificate
   - `--key` flag for the private key

#### Password Security (Generic Secret)

1. Use `kubectl create secret generic` with:
   - `--from-literal` flag for providing password data.

#### SSH Key Management

1. Use `kubectl create secret generic` with:
   - `--from-file` flag for specifying the SSH private key file.

#### Sensitive File Access

1. Use `kubectl create secret generic` with:
   - `--from-file` flag for specifying the sensitive file.

#### Usage Similarity to ConfigMaps

- Secrets are used similarly to ConfigMaps within applications.
- For variables, use `kubectl set env` and consider the `--prefix` option.
- For files, manually mount the Secret, as demonstrated for ConfigMaps.
- Use `defaultMode` to set proper permissions while mounting Secrets.

#### Automatic Updates and Security Considerations

- Mounted Secrets are updated automatically in the application upon Secret updates.
- While Secrets provide security while stored in the cloud, they are plain readable files inside Pods.
- Secrets protect values in transit but become plain text within Pods.

#### Walkthrough

1. Create a generic Secret named `dbpw` using `kubectl create secret generic` with `--from-literal`.
2. Inspect the Secret using `kubectl describe secret` and `kubectl get secret -o yaml`.
3. Create a MariaDB deployment named `mynewdb` using `kubectl create deploy` with the MariaDB image.
4. Use `kubectl set env` to set environment variables from the Secret, using the `--prefix` option.
5. Verify the successful deployment using `kubectl get all` and `kubectl exec` to inspect the environment.

```
$ kubectl create secret generic -h | less
$ kubectl create secret generic dbpw --from-literal=ROOT_PASSWORD=password
secret/dbpw created
$ kubectl describe secret dbpw
$ kubectl get secret dbpw -o yaml
apiVersion: v1
data:
  ROOT_PASSWORD: cGFzc3dvcmQ=

$ echo cGFzc3dvcmQ= | base64 -d
password

$ kubectl create deployment mynewdb --image=mariadb

$ kubectl set env --help | less

$ kubectl set env deploy mynewdb --from=secret/dbpw --prefix=MYSQL_
deployment.apps/mynewdb env updated

$ kubectl get all

$ kubectl exec mynewdb-64547f8d6-gw57r -- env
```

## Configuring Docker Registry Access Secret in Kubernetes

- For accessing container registries, authentication is required.
- Anonymous access to registries often comes with limitations.
- Docker credentials need to be configured in Kubernetes for cloud-native environments.

#### Docker Registry Secret: Purpose and Usage

- **Purpose:** To store Docker authentication credentials securely in Kubernetes.
- Use it to decouple authentication information from local nodes and store it in the cloud.

### Example Parameters for Creating Docker Registry Secret

- `kubectl create secret docker-registry` command parameters:
  - `--docker-username`: Your Docker username
  - `--docker-password`: Your Docker password
  - `--docker-email`: Your Docker email
  - `--docker-server`: The Docker registry server URL (e.g., `docker.io`)

#### Creating Docker Registry Access Secret

1. Use the `kubectl create secret docker-registry` command with necessary parameters.
   kubectl create secret docker-registry my-docker-credentials --docker-username=$DOCKER_USERNAME --docker-password=$DOCKER_PASSWORD --docker-email=$DOCKER_EMAIL --docker-server=docker.io

- Verify the secret creation using kubectl get secrets and kubectl describe secrets.
- Understand that sensitive values are stored in a .dockerconfigjson file.
- Ensure the secure management of Docker credentials to avoid unauthorized access.

```sh
$ kubectl create secret docker-registry -h | less

$ kubectl create secret docker-registry my-docker-credentials --docker-username=xxxxx --docker-password=xxxxxx --docker-email=xxxx@gmail.com --docker-server=docker.io
secret/my-docker-credentials created

$ kubectl get secrets
NAME                    TYPE                             DATA   AGE
dbpw                    Opaque                           1      46m
my-docker-credentials   kubernetes.io/dockerconfigjson   1      77s

$ kubectl describe secrets

$ kubectl get secret my-docker-credentials -o yaml
```

```sh
$ echo hello world > index.html

$ kubectl create cm secretlab --from-file=index.html
configmap/secretlab created

$ kubectl create secret generic secretlabsecret --from-literal=MYPASSWORD=verysecret
secret/secretlabsecret created

$ kubectl get secrets secretlabsecret -o yaml

$ echo dmVyeXNlY3JldA== | base64 -d

$ kubectl create deployment secretlabdeploy --image=nginx
deployment.apps/secretlabdeploy created

$ kubectl set env deploy secretlabdeploy --from=secret/secretlabsecret
deployment.apps/secretlabdeploy env updated

$ kubectl edit deployments.apps secretlabdeploy
 ## do this 
     spec:
      volumes:
      - name: secretlab
        configMap:
          name: secretlab
      containers:
      - env:
        - name: MYPASSWORD
          valueFrom:
            secretKeyRef:
              key: MYPASSWORD
              name: secretlabsecret
        image: nginx
        imagePullPolicy: Always
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - name: secretlab
          mountPath: /usr/share/nginx/html
    ###

$ kubectl get deploy secretlabdeploy -o yaml > secretlabdeploy.yaml

$ kubectl delete deployments.apps secretlabdeploy

$ kubectl create -f secretlabdeploy.yaml
deployment.apps/secretlabdeploy created

$ kubectl exec secretlabdeploy-dd9dcf7bb-p4447 -- cat /usr/share/nginx/html/in
dex.html
hello world

$ kubectl exec secretlabdeploy-dd9dcf7bb-p4447 -- env
```