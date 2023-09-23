# Container Overview

- A container is a self-contained ready-to-run application package that contains everything needed to run an application.
- Containers run on top of the local host kernel and communicate with the host platform through a container runtime.
- Containers rely on the host operating system's kernel and use a container runtime (e.g., containerd, CRI-O) to run container images.
- Container images include the application, its dependencies (libraries), and glibc, which provides the container with the identity of a specific operating system.
- Containers offer strict isolation using Linux Kernel NameSpaces for networking, files, users, processes, and IPCs, ensuring different components run independently.
- CGroups in Linux enable resource allocation and limitation for containers.

## Components of Containers

### Images

- Read-only environments containing the runtime environment, including the application and required libraries.
- Stored in registries such as Docker Hub or private registries.

### Containers

- Isolated runtime environments where the application runs.
- Offer strict isolation using name spaces.

### Registries

- Used to store container images.
- Examples: Docker Hub, quay.io, private registries.
- Docker Hub is the default registry in Kubernetes.

## Container Runtimes

- Container runtimes enable starting and running containers on top of the host operating system.
- Examples of container runtimes include Docker, lxc, runc, and CRI-O (containerd).
- The Open Containers Initiative (OCI) provides standardization for container packaging and running.

## Docker and Podman

- Docker is a popular container solution that provides an image format, dockerfile, and tools for managing and running containers.
- Podman is an alternative used in Red Hat environments and runs containers without a daemon on the cryo container runtime.
- Red Hat dropped support for Docker in RHEL 8 in favor of Podman due to high compatibility.

## Diagrams
```

                  +----------------+
                  |     Images     |
                  +----------------+
                  |                |
                  | Runtime        |
                  | Environment    |
                  | (Application   |
                  |  + Libraries)  |
                  |                |
                  +----------------+
                  |                |
                  |   Containers   |
                  |                |
                  +----------------+
                  |                |
                  |   Registries   |
                  |                |
                  +----------------+
```

```
   Host Operating System (e.g., RHEL 9)
   +----------------------------------------+
   | Kernel                                 |
   | Linux Kernel                           |
   | Version: 5.x.x                         |
   +----------------------------------------+
   | Container Runtime (e.g., containerd)   |
   +----------------------------------------+
   | Container Image                        |
   | +------------------------------------+ |
   | | Default Application                | |
   | +------------------------------------+ |
   | | Libraries (Dependencies)           | |
   | +------------------------------------+ |
   | | glibc (OS Identity)                | |
   | +------------------------------------+ |
   +----------------------------------------+

```

```
   Containers                                   Virtual Machines
   +-------------------------+                  +-------------------------+
   | Container Runtime       |                  | Hypervisor              |
   | (e.g., Docker, CRI-O)   |                  | (e.g., KVM, VMware)     |
   +-------------------------+                  +-------------------------+
   | Host Operating System   |                  | Host Operating System   |
   | Kernel                  |                  | Kernel                  |
   +-------------------------+                  +-------------------------+

```

