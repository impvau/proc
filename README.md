
...Under construction...

# Introduction

This repository provides an opinioned recommendation for development in academic research.

| Process | Description |
| ------------- | ------------- |
| [Prerequisites](PREREQUISITES.md)  | Introduction for software to perform local and at-scale execution, debug and share reproducible code, and  |
| [Kubernetes: Docker Desktop](KUBERNETES.md)  | Local Execution of multiple containers on a single host |
| [Kubernetes: Microk8s](MICROK8S.md) | Scaled Execution of multple containers on multiple hosts |
| [Reproduction](REPRODUCE.md)  | Process for ensuring complete reproducibility of experiements |


# Cheatsheet

Build Container
```
docker build -t <reop>/230830-pbf-sk-intp -f .devcontainer/mlab-hpc.Dockerfile .
```

Run container priviledged, overriding entry point
```
docker run -it --privileged --entrypoint /bin/bash <repo>/230830-pbf-sk-intp
```

Push to repository
```
docker push <repo>/230830-pbf-sk-intp
```

Access Kubernetes Dashboard
```
// Save dashboard podname
export POD_NAME=$(microk8s kubectl get pods -n kubernetes-dashboard -l "app.kubernetes.io/name=kubernetes-dashboard,app.kubernetes.io/instance=kubernetes-dashboard" -o jsonpath="{.items[0].metadata.name}")

// Port forward pod
microk8s kubectl -n kubernetes-dashboard port-forward $POD_NAME 8443:8443

// Generate token to login
microk8s kubectl -n kubernetes-dashboard create token admin-user

// Access and copy paste the token at 
// https://127.0.0.1:8443/#/login
```

Trigger a job for HPC execution given `impv.yaml`
```
microk8s kubectl apply -f impv.yaml
```