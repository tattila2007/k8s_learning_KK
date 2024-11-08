In certain cases, applications deployed on the Kubernetes cluster require specific configurations or setup changes before launching the app container. The Nautilus DevOps team has devised a solution using init containers to fulfill these prerequisites during deployment. Below is an initial test scenario:


a. Create a Pod named red-devops-t1q5. It must have an init container named red-init-devops-t1q5, it should utilise image fedora (preferably with the latest tag). The command '/bin/bash', '-c' should be used with arguments echo "Welcome!"

b. The main container name should be red-main-devops-t1q5 and it should utilise image fedora (preferably with the latest tag). The Command: '/bin/bash', '-c' should be used with arguments sleep 1000

This scenario demonstrates the use of init containers to fulfil pre-requisites before deploying the main application container in the Kubernetes Pod.

Note: The kubectl utility on jump_host has been configured to work with the kubernetes cluster.

Pod 'red-devops-t1q5' exits

Init container 'red-init-devops-t1q5' exists

Init container is utilising image 'fedora'

Main container 'red-main-devops-t1q5' exists

Main container is using image 'fedora'

Pod is 'Running'

Init container is configured as expected

Main container is configured as expected


apiVersion: v1
kind: Pod
metadata:
  name: red-devops-t1q5
spec:
  containers:
  - name: red-main-devops-t1q5
    image: fedora:latest
    command: ['/bin/bash', '-c', 'sleep 1000']
  initContainers:
  - name: red-init-devops-t1q5
    image: fedora:latest
    command: ['/bin/bash', '-c', 'echo "Welcome!"']

apiVersion: v1
kind: Pod
metadata:
  name: red-devops-t1q5
spec:
  containers:
  - name: red-main-devops-t1q5
    image: fedora:latest
    command: ['/bin/bash', '-c', 'sleep 1000']
  initContainers:
  - name: red-init-devops-t1q5
    image: fedora:latest
    command: ['/bin/bash', '-c', 'echo "Welcome!"']
