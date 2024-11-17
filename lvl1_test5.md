k8s lvl1 test5

5 / 10
Weight: 10
The Nautilus DevOps team is actively creating jobs within the Kubernetes cluster. While they are in the process of finalizing actual scripts/commands, they are presently structuring templates and testing the jobs using placeholder commands. Below are the specifications for creating a job template:


Create a job named countdown-devops-t3q2.

The spec template should be named as countdown-devops-t3q2 (under metadata), and the container should be named as container-countdown-devops-t3q2.

Use image fedora with latest tag only and remember to mention tag i.e fedora:latest, and restart policy should be Never.

Use command sleep 5.

Note: The kubectl utility on jump_host has been configured to work with the kubernetes cluster.

Job 'countdown-devops-t3q2' exists

Spec template is named as 'countdown-devops-t3q2'

Image used is 'fedora:latest'

Container name is 'container-countdown-devops-t3q2'

restart policy is not set to 'Never'

```
apiVersion: batch/v1
kind: Job
metadata:
  name: countdown-devops-t3q2
spec:
  template:
    metadata:
      name: countdown-devops-t3q2
    spec:
      containers:
      - name: container-countdown-devops-t3q2
        image: fedora:latest
        command: ["sleep", "5"]
      restartPolicy: Never
```
