# k8s_learning

1 / 7
Weight: 15
Upgrade the current version of kubernetes from 1.30.0 to 1.31.0 exactly using the kubeadm utility. Make sure that the upgrade is carried out one node at a time starting with the controlplane node. To minimize downtime, the deployment gold-nginx should be rescheduled on an alternate node before upgrading each node.


Upgrade controlplane node first and drain node node01 before upgrading it. Pods for gold-nginx should run on the controlplane node subsequently.

Cluster Upgraded?

pods 'gold-nginx' running on controlplane?

2 / 7
Weight: 15
Print the names of all deployments in the admin2406 namespace in the following format:

DEPLOYMENT   CONTAINER_IMAGE   READY_REPLICAS   NAMESPACE

<deployment name>   <container image used>   <ready replica count>   <Namespace>
. The data should be sorted by the increasing order of the deployment name.


Example:

DEPLOYMENT   CONTAINER_IMAGE   READY_REPLICAS   NAMESPACE
deploy0   nginx:alpine   1   admin2406
Write the result to the file /opt/admin2406_data.


Task completed?

kubectl -n admin2406 get deployment -o custom-columns=DEPLOYMENT:.metadata.name,CONTAINER_IMAGE:.spec.template.spec.containers[].image,READY_REPLICAS:.status.readyReplicas,NAMESPACE:.metadata.namespace --sort-by=.metadata.name > /opt/admin2406_data

3 / 7
Weight: 8
A kubeconfig file called admin.kubeconfig has been created in /root/CKA. There is something wrong with the configuration. Troubleshoot and fix it.


Fix /root/CKA/admin.kubeconfig
```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJZVFWVGl3RURaNEV3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TlRBeE1UQXhNak0wTkRCYUZ3MHpOVEF4TURneE1qTTVOREJhTUJVeApFekFSQmdOVkJBTVRDbXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLCkFvSUJBUURDcTRrbkVOZUNMcWdmM1lFNDQ5aHJwMEs5QVB2U2ZZYmc5YUM0d1J5ai95SlBUbkY1d0Ezc21ITWEKTnBaNmVxWk51T1pCeFYzT1BBUDNqTXRGcXdOekRzeGVwYlFGVmw3R2lnMlVWOGR3UHFNc2NGMkxTbXIvMHYwcQppMGQxRWF4R1V1T29tQ0k2cTZKWGhEQ0czUE53VUszR0lDdjRTSWxDQktiNFlRSjFDMU44ZDN6VzQyeG9tL1dJCkR2bk9wc09rRFBMdWtYRXYrVGF1TS92em9XdWE5QWJtbjdHS1Y3N0hCQVRIdVMzbWpjeGd1S1YwZXMzR3ZQR0gKNGRmazUwaWVtUCt6QWhYRXBsNjdkQmlBbkVRWU5JYXErc0g2Nkd2QmR0UTlXRXRFN1hjeXIxN2FteVpCTWxIZQpHY2FtUjY5Rjh2ZnhSZWg0V05TM1hVRXdkbUEzQWdNQkFBR2pXVEJYTUE0R0ExVWREd0VCL3dRRUF3SUNwREFQCkJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJUWTIxcUgrZnRscHJWTXM0NEhvUmF5elNydllqQVYKQmdOVkhSRUVEakFNZ2dwcmRXSmxjbTVsZEdWek1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQVRRdXNHM3NORApiaVIyOHdEWTk5OTh3bHZXTEFPV1N6d2xLQ2E3U2RjT2c2c3RDUnVxTTZhNzFJbkdBNkhMQ3FRYWxYMTdqeTFuCkd1NW5LUnBSZFJHUlVrSDBVelVEVTEydTUwblRUY09Lc3I4V0FNM2dDTDhZWFQ5dnZLa3d6YXRuMWxIdmFlYnIKUy9RVk4rcHIrbFUrZThSdHhaeWl2VFVCcC96UmtKY2RnV2hxVU1SK0lpK0gxSDdhVVVGU1E5eW4zQnA0Tml2ZgpOZDFxMkw1TUFKTHFhck14cGpCRGJhQ0ljT0ZUVnF0RTJXM0NuR0xUWlRVdXpvdWUrbEpWTysrTEJtUlR1OStaCjd2MlZyREJocXJsNlhadDJRYjd0bnZZcmxCVWlMKzNCNFJrUS9IeFRCcDUvVFdQNHl3MFRkOHVYSXZteFBrOXcKa29pL3ZQY1RyOGJqCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    server: https://controlplane:4380
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
```


