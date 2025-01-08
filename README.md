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
