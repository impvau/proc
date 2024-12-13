
# Visual Studo Code (VSC) and Containers

The latest documentation on containers is the best source, while here provides concise details for the current versions.

Containers can be 
- Pre-built
- Built from Dockerfile

## Pre-Built Containers

The benefit of pre-built containers is that they are a static snapshot of the exact state of the enviroment. This include copies of the code and data used in specific experiements that (if seeded) facilitate 100% reproducibility and debug of the results. This may not be the case if you container runs scripts to download the data and repo's rather than copying it into the container as commands in the Dockerfile. 

The detriment of pre-built containers is that their size is large, although you can have a "development" container that is designed for inspecting the code, data and documentation, and a "production" container that has the bare minimum to execute the code. Consider a container that allows you to execute a C++ binary, than one that requires all the packages to build the binary from the source code. Generally, the size is not a problem and we care more about the full access to code given that research is not about high-scale operation like commercial products.

VSC is not necessary for pre-built contianers, as you can `docker pull` containers using your installation of Docker and simply run them if the Dockerfile uses to build the container has a default execution. In this case, you generally only need to specify the output directory when you run the contianer to have access to the persistent output of the container. The initial command will be run and the container will shutdown after execution. If the output directory is not mounted, the output is deleted with the container, in the same way that when you restart the container nothing that was previously done remains in the container.

When a default execution is not defined, or say you want to run with different paramaters, you can instead override the default execution and open a terminal to the container. This allows you a terminal to the OS and can undertake any actions you desire. When the container continues to run, you can attach to the container with VSC to inspect and debug should you wish to.

## Building from a Dockerfile

The benefit of building a Dockerfile from scratch is that you can easily change the Dockerfile and the state of the container for your own experiments. For instance, extending the source code or adding your algorithm into the suite of algorithms used within the source code. VSC is also designed to utilise Dockerfiles for developing within a "Dev Container" and launching pre-set configurations in the `.vscode/launch.json` directory such as 'Pre-process', 'Execute' and 'Post-process' which may all be in a different language but be configured for step-by-step debugging 

A detriment of building from a Dockerfile is possible differences in environments depending on how the Dockerfile is configured. If the Dockerfile does not specify the specific versions of different packages, then the source code may not run because packages are deprecated or incompatabile with new packages, etc. When providing a Dockerfile for reproduction in the long-term, research papers should specify a container uploaded to a Docker Registry that is simply pulled, or specify all versions for all packages in the Dockerfile commands. If you specify the versions, there may still be problems if they are removed from the package repos over time. If you create a pre-build container, you must ensure that you continue to host the container for as long as needed.

## Dockerfiles

Consider the example Dockerfile with comments

```
# Base image, specifying version 12.0.1 and Ubuntu 22.04
# We utilise an image that comes enabled for Cuda GPU integration
FROM nvidia/cuda:12.0.1-devel-ubuntu22.04

# Set non-interactive mode for APT
ENV DEBIAN_FRONTEND noninteractive

# We use RUN to run commands as if we would on the command line
# Here we do a standard apt-get update on the base image and then setup 
# the environment to use the latest R environment

# Note that when we run the container, all of these commands are executed into the 'default' image
# Once we build the container, when we start the container it is at the end of the execution of all these commands

RUN apt-get update && apt-get install -y --no-install-recommends software-properties-common wget && \
    wget -qO- https://cloud.r-project.org/bin/linux/ubuntu/marutter_pubkey.asc | gpg --dearmor -o /usr/share/keyrings/cran-archive-keyring.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/cran-archive-keyring.gpg] https://cloud.r-project.org/bin/linux/ubuntu $(lsb_release -cs)-cran40/" > /etc/apt/sources.list.d/cran-r.list

# R and R development tools
RUN apt-get install -y r-base
RUN apt-get install -y r-base-dev

# Some packages required as pre-reqs for the R packages
RUN apt-get install -y libcurl4-openssl-dev wget libxml2-dev libssl-dev

# Install required R packages
RUN R -e "install.packages('demography', dependencies=TRUE, repos='http://cran.rstudio.com/')"
RUN R -e "install.packages('StMoMo', dependencies=TRUE, repos='http://cran.rstudio.com/')"

# Install Java and R packages that utilise it
RUN apt-get install -y default-jdk && R CMD javareconf
RUN R -e "install.packages('rJava', dependencies=TRUE, repos='http://cran.rstudio.com/')"
RUN R -e "install.packages('xlsxjars', dependencies=TRUE, repos='http://cran.rstudio.com/')"
RUN R -e "install.packages('xlsx', dependencies=TRUE, repos='http://cran.rstudio.com/')"

# Set the default working directory, where we start when the run the container
WORKDIR /workspaces

# Note there is no default command which is run when the container is started
```

## Dev Containers and Building a Dockerfile

VSC keeps the Dev Container configurations in the .devcontainer directory.
You can also keep your Dockerfiles in here.

The key configuration is the `.devcontainer/devcontainer.json` file which is considered when you run 'Dev Containers: Rebuild and Reopen in Container' (ensuring you have the Dev Containers extension downloaded in VSC).

An example `devcontainer.json` file.
```
{
    "name": "Python Container",
    "dockerFile": "Dockerfile",     // Specify that .devcontainer/Dockerfile is used to build the dev container
    "forwardPorts": [8787],         // Configure the port forwarding on the container
    "customizations": {             // Specify what extensions need to be installed in the devcontainer after building and 
                                    // running, for instance the VSC extensions for debugging
                                    // Note that this is installed in the container OS, and has nothing to do with 
                                    // the local extensions or packages installed
            "vscode": {
                    "extensions": [
                        "REditorSupport.r",
                        "RDebugger.r-debugger",
                        "Mikhail-Arkhipov.r",
                        "ms-python.python"
                    ]
            }
    }
}
```

## VSC and Launch Configurations

VSC allows you to define launch configurations in the .vscode directory that configure the executable, script etc. that is executed and what parameters are passed in to it. This can allow someone to execute the exact command, or run a series of executions that are exactly the same as another using the container. These configurations also allow you to specify how they are run, for instance with a debugger, that allows you to put break-points in the source code and inspect all the live variables and go line-by-line through the code.

An example `.vscode/launch.json`
```
{
    "version": "0.2.0",

    // Here we list the possible configurations.
    // These show in the 'Run' table which you can select and run and execute the pre-defined codes
    "configurations": [
        {
            // The run option will show the 'name' field
            // and runs the R-Debugger on the src/compute_cfr.R file
            "type": "R-Debugger",
            "name": "Debug compute_cfr.R",
            "request": "launch",
            "debugMode": "file",
            "workingDirectory": "${workspaceFolder}",
            "file": "${workspaceFolder}/src/compute_cfr.R"
        },
        {
            // This runs the R-debugger on a different R file
            "type": "R-Debugger",
            "name": "Point Forecasts",
            "request": "launch",
            "debugMode": "file",
            "workingDirectory": "${workspaceFolder}",
            "file": "${workspaceFolder}/src/benchmark/point_forecast.R"
        },
        {
            // Run the python debugger on the pre-processing code
            "name": "Pre Process",
            "type": "debugpy",
            "request": "launch",
            "program": "${workspaceFolder}/src/preproc.py",
            "console": "integratedTerminal"
        },
        {
            // Run the python debugger on the post-processing code
            "name": "Post Process",
            "type": "debugpy",
            "request": "launch",
            "program": "${workspaceFolder}/src/postproc.py",
            "console": "integratedTerminal"
        },
        {
            // Run the CFR C++ code with a large set of specific argments passed into the executable
            // It utilises the cpp debugger so the source code can be stepped through

            "name": "CFR Launch",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/bin/main",
            "args": [
                
                "-s","1433120488"
                ,"-t", "${workspaceFolder}/data/ellipse_hp_t.csv"
                ,"-T", "${workspaceFolder}/data/ellipse_hp_t.csv"
                ,"-lt", "${workspaceFolder}/out/"
                ,"-f", "5"
                ,"-g", "-1"
                ,"-ld", "0.2"
                ,"-dd", "adp-rnd"
                ,"-dc", "5"
                ,"-dm", "stale-ext"
                ,"-lt", "${workspaceFolder}/out"
                ,"-mt", "3600"
                ,"-e", "1e-8"
            ],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],

            // !!! Note this is a thing executed before the executable is run and 
            // defined in .vscode/tasks.json
            // e.g. we run the make file to clean and build all the source code
            // This ensures we run the executable that has any changes done to the source
            //
            //  {
            //      "version": "2.0.0",
            //      "tasks": [
            //          {
            //              "label": "Build Main",
            //              "type": "shell",
            //              "command": "clear; make clean; make",
            //              "group": {
            //                  "kind": "build",
            //                  "isDefault": true
            //              }
            //          },
            //      ]
            //  }
            //
            "preLaunchTask": "Build Main"
        },
    ]
}
```

## Gotchyas

- C++ can have optimization levels set. Sometimes the debugger does not align with the source code due to this optimization and the debugger will not stop on specific lines of code, or report some variable values as 'Optimized Out'. Should this occur, set the optimization to 0 to ensure access to all debug capabilities.
