
...Under construction...

# Intoduction

The simpliest manner to have a Kubernetes cluser on your local machine to divide your resources across the cores and RAM is by using the Kubernetes plugin directly in [Docker Desktop](https://www.docker.com/).

At version 4.18.0, you can enable Kubernetes in Docker Desktop by
1. Go to Settings at the top right corner
2. Select 'Kubernetes' from the left menu
3. Select 'Enable Kubernetes'

The `kubectl` command will be availble in Powershell, WSL, or the terminal in your operating system.

Now you can generate a Job yaml file specifying the nature of execution for the cluser, and submit it to the cluster with `kubectl apply -f job.yaml` 

Example configuration file call `job.yaml`
``` 

apiVersion: batch/v1
kind: Job
metadata:
  name: YYMMDD-unique-name    # Set the unique job id for kubernetes
spec:
  
  # Number of parallel jobs to run, current IMPVGRID total capacity is 35 at 4000Mi RAM, 1000m CPU
  # Number of completions required e.g. have 20 in parallel but require 40 completions - as resources
  # are freed, more containers will begin
  parallelism: 20             
  completions: 20            
  
  template:
    spec:
      containers:
      - name: mtbtm           # Name of the container, easiest to copy job name

        # Location of container image to download
        # Format <registry-server>/<container-name>:<version>
        # Best practice is to group a set of experiements into a <container-name>
        # then have specific versions undernearth e.g. for 'cfr-knuth-bins-penn20' we have
        # versions '220414-f1', '220414-f2', '220414-f2-m0.1' that indicate what
        # critical parameters/info
        image: dsm.impv.au/mlab-testarea:YYMMDD-unique-name   

        # Always pull container before executing on the container. If the image has not 
        # changed there is no overhead
        imagePullPolicy: Always     

        # Minimum hardware requested and required to be free for a Pod to start on a Node
        # At the moment, if the resources aren't available the job fails, so we must always
        # ensure we have the number of free resources below times the 'parallelism' amount
        # that will be executing at the same time. The CPU is in 'milliCPUs' and is the number
        # of threads (not physical CPUs)
        resources:
          requests:
            cpu: "750m"
            memory: "1000Mi"
          limits:
            cpu: "750m"
            memory: "1000Mi"
            
        # We avoid painful volume mounting by enabling privleged access to the container
        # such that we can use cips to mount a local network share
        securityContext:
          privileged: true
          capabilities:
            add: ["NET_ADMIN", "SYS_TIME"]

        # Initial command to trigger
        # When not specified, it uses the ENTRYPOINT or CMD defined in the dockerfile
        # This is generally preferred, as we setup connection to a NAS share and define
        # the folders that we write in to
        # command: ["python execute.py"]
        
      # The configured secret for accessing private container registry. Crednetial name always set to 'regcred'
      # Created by kubectl create secret docker-registry regcred --docker-server=<> --docker-username=<> --docker-password=<>
      #imagePullSecrets:
      #  - name: regcred

      # Defining behaviour that occurs when container crashes. We loose logging information when we restart so
      # we always set to Never. May need to reconsider when we move mlab to a 'service' over a batch 'job' style
      restartPolicy: Never
```

To stop or remove a job logs, you can use the following command, assuming that `job.yaml` still contains the same name you used to apply the job
```
kubectl delete -f job.yaml
``` 