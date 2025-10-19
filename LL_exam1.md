Task 1: Configuring a HA Cluster
* Configure a High Availability cluster with three control plane nodes and two worker nodes.
* Ensure that each control plane node can be used as a client as well.
* Use the scripts provided in the course Git repository at https://github.com/sandervanvugt/cka to install the CRI, kubetools andload balancer.

Task 2: Scheduling a Pod
Schedule a Pod with the name lab152pod that runs the Nginx and redis applications. 
It should also be able to run on the control plane. 
Do NOT change anything on the control plane nodes.

Task 3: Managing Application Initialization
Create a Deployment with the name lab153deploy which runs the Nginx
image, but waits 30 seconds before starting the actual Pods.

Task 4: Setting up Persistent Storage
Create a Persistent Volume with the name lab154 that uses HostPath on the
directory /lab154.

Task 5: Configuring Application Access
* Create a Deployment with the name lab155deploy, running 3 instances of
the Nginx image.
* Configure it such that it can be accessed by external users on port 32567
on each cluster node.

Task 6: Securing Network Traffic
Create a Namespace with the name “restricted”, and configure it such that
it only allows access to Pods exposing port 80 for Pods coming from the
Namespace access and only if these incoming Pods are using the label
access=true.
Create a Pod with the name “lab156web”, running the Nginx image in the
restricted Namespace, and a Pod with the name “lab156access”, running
the Busybox image in the access Namespace and verify that access works.

Task 7: Setting up Quota
* Create a Namespace with the name “limited”, and configure it such that
only 5 Pods can be started and all Pods can use no more than 1GiB
memory.
* Run a webserver Deployment with the name “lab157deploy” and using 3
Pods in this Namespace. Each Pod should request at least 32MiB RAM
while starting.
Task 8: Creating a Static Pod
* Configure a Pod with the metadata name set to lab158pod that will run the
Nginx image and be started by the kubelet on node worker2 as a static Pod.

Task 9: Troubleshooting Node Services
Assume that node worker2 is not currently available. Ensure that the
appropriate service is started on that node which will show the node as
running.
Task 10: Configuring Cluster Access
* Create a ServiceAccount "lab1510access" that has permissions to create
Pods, Deployments, DaemonSets, and StatefulSets in the Namespace
access, using the role "role1510".
* Inthe same Namespace, create the Pod "lab1510pod", which starts the
Busybox container image and the command sleep infinity, and uses this
ServiceAccount.

Task 11: Configuring Taints and Tolerations
* Configure node worker2 such that it will only allow Pods to run that have
been configured with the setting type:db
« After verifying this works, remove the node restriction to return to normal
operation.
