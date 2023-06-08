
...Under construction...

# Intoduction

The prerequisites section covers the motivation and context around the advocated development environment. This includes the use of
- Containers - for encapsulating all program artifacts and environment configurations for exact reproduction
- Container Registries - for storing a built containers as a persistent artifacts 
- Visual Studio Code IDE - for its compatability with containers, container registries and tools for great development practices

# Containers

Containers are used for encapsulating all code, data, software libraries, and operating systems of the environment required to run the program, analogous to a light-weight virtual machine.

This is not limited to a single, physical machine but could be a pod of multiple containers that interact with each other (e.g. a database container, a queue container, and the program container). We currently assume a pod with a single container that encapsulates all software as a pod requires.

The use of containers faciltiates:
- Encapsulation of all program requirements, removing the need to download anything other than a single container
- Zero configuration of the environment, rather the downloaded container is run with zero chance of missing libraries, incompatible versions, or environmental differences
- Very fast transfer of an individuals program for debugging or use by peers (we build and push the container to a registry, where a peer can download and mirror our environment in near-real-time)

The container software recommended is [Docker Desktop](https://www.docker.com/) which facilitates command line interface, but can also be used via the interface to download and run another researchers experiment.

Containers can be build with a `Dockerfile`, or a built container can be pushed to a `Container Registry` for download and execution as a persistent artifact. The `Dockerfile` specifies:
- The base image, e.g. Ubuntu, Windows, Debian etc.
- Installation or configuration commands to run on the base image
- Files and folders that are copied from the build location into the container

A `Dockerfile` can be used to build a container, but if the `Dockerfile` uses the 'latest' software versions and does not define the specific software versions, there is the chance over time that the software versions become incompatable with the program code. A `Container Registry` on the other hand is already built and as long as the researcher has tested the container, can always be used to replicate the experiemntal results.

When a built container is run, the initial state of the container is instanciated and prior experiements or changes made in the container are lost, ensuring that the environment always remainers persistent. This requires us to mount a directory outside of the container such that we can export information or output to our host machine on which the container is run.

Containers have near-native performance on the host machine and limited resources can be used, such as running 10 containers in parellel each with 1 CPU and 1 GB of RAM to consume a totale of 10 CPUs and 10 GB of RAM on the host machine. Similarly, the GPU can also be shared, however, not all domestic GPUs (such RTX series) can be used in parallel with containers as of yet.

In summary, useful applications of containers include:
- Sharing development environments between multiple people in the team, for project handover, or allowing seniors debug issues in anothers environment

- Archiving key software that is helpful but uses old technology, or technology that is not primarily used by the team. For instance, if R is not commonly used in the team, but is very helpful for plotting, you can create a container and a specific set of instructions that allows users to re-create plots without needing to work through setup issues. 

- Allow peer researchers to download a container, execute it, and corroborate results without any code

- As an extension of the last point, you can provide the integration with VS Code such that they can step-by-step debug your software with zero setup and difficulty outside of installing Docker Desktop and downloading the container.

- Storing static key versions of research projects as they evolve, for instance, version 1 which is used in publication with no special data structures, then version 2 which uses matrices, then version 3 which uses GPU integration etc.

# Container Registry

Container Registries are a local where built containers are stored, and can be downloaded and run. They can be used as a persistent and static archive for published results, and internal teams can share restricted-access containers.

Cloud providers (AWS, GC, Azure etc.) offer container registry services and web interfaces for access configuration.

# Kubernetes 

Kubernetes is a container orchestration software, which can be used to execute a container at scale and in parallel. The same containers that are used in development, can be provided to a Kubernetes HPC and executed at scale.

Kubernetes can be enabled in Docker Desktop, where the "cluster" exists of your local machine, however it can still be used to split the resources of your machine for many parallel executions. Say a machine with 24 cores, 64 GB RAM could be used to run 20 parallel issues, each with a 1 core, 2GB RAM and still have 4 cores, 24 GB of ram to run the local environment.

For a multi-node Kubernetes cluster, we use microk8s and an NAS storage, as defined in the [grid](GRID.md) documentation.

# Visual Studio Code (VS Code)

[VSCode](https://code.visualstudio.com/) is a modern IDE that facilitates development in containers and many community extensions that on balance, we feel marks this IDE as the leading tool.

## VS Code Extensions

- Dev Containers (id: ms-vscode-remote.remote-containers). Facilitates use of containers within VS Code

- Docker (id: ms-azuretools.vscode-docker). Facilates connection to Azure registries

- Azure Account (id: ms-vscode.azure-account). Connect to Azure container registry 

- C/C++ (id: ms-vscode.cpptools). Tools for C++ developments and debugging

- Python (id: ms-python.python). Tools for Python development and debugging

