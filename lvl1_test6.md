k8s lvl1 test6
6 / 10
Weight: 10
The Nautilus DevOps team plans to deploy applications on a Kubernetes cluster for the migration of some existing applications. Recently, a team member has been tasked with creating below components:


a. Create a ReplicaSet named httpd-replicaset-t3q4 using httpd image with latest tag only (remember to mention tag i.e httpd:latest).

b. Label app should be httpd_app_t3q4, label type should be front-end-t3q4.

c. The container should be named as httpd-container-t3q4, also make sure replicas counts are 4.

Note: The kubectl utility on jump_host has been configured to work with the kubernetes cluster.

Replicaset name is 'httpd-replicaset-t3q4'

Labels 'app' is set to 'httpd_app_t3q4'

Labels 'type' is set to 'front-end-t3q4'

Image used is 'httpd:latest'

Container name is 'httpd-container-t3q4'

Replicas count is '4'

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: httpd-replicaset-t3q4
  labels:
    front-end-t3q4: httpd_app_t3q4
spec:
  # modify replicas according to your case
  replicas: 4
  selector:
    matchLabels:
      front-end-t3q4: httpd_app_t3q4
  template:
    metadata:
      labels:
        front-end-t3q4: httpd_app_t3q4
    spec:
      containers:
      - name: httpd-container-t3q4
        image: httpd:latest
```
