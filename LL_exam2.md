Task 1: Creating a Cluster
* Create a 3-node Kubernetes cluster, using one control plane node and 2 worker nodes.
* Use the following scripts provided in the course Git repository at https://github.com/sandervanvugt/cka to install the previous version of Kubernetes.
* setup-container.sh
» setup-kubetools-previousversion.sh

Task 2: Performing a Control Node Upgrade
* Upgrade the control node to the latest version of Kubernetes.
* Ensure that the kubelet and kubectl are upgraded as well.

Task 3: Configuring Application Logging
* Create a Pod with a logging agent that runs as a sidecar container.
* The Pod should have the name **exam2-task3**.
* Configure the main application to use Busybox and run the Linux date command every minute.
* The result of this command should be written to the directory /output/date.log.
* Set up a sidecar container that runs Nginx and provide access to the date.log file on /usr/share/nginx/html/date.log.

Task 4: Managing PersistentVolumeClaims
* Create a PersistentVolume with the name **exam2-task4-pv** that uses 1GB of hostPath storage.
* Configure this PV such that when it is no longer used by a PVC, it will be deleted.
* Create a PersistentVolumeClaim with the name **exam2-task4-pvc** that uses the PersistentVolume; the PersistentVolumeClaim should request **100MiB** of storage.
* Run a Pod with the name **storage**, using the Nginx image and mounting this PVC on the directory /data.
* After creating the configuration, change the PersistentVolumeClaim to request a size of **200MiB**.

Task 5: Investigating Pod Logs
* Run a Pod with the name **failingdb**, which starts the **mariadb** image without any further options (it should fail).
* Investigate the Pod logs and write all lines that start with ERROR to /tmp/failingdb.log

Task 6: Analyzing Performance
* Find out which Pod currently has the highest CPU load.

Task 7: Managing Application Scheduling
* Run a Pod with the name **lab167pod** using the Nginx image.
* Ensure that it only runs on nodes that have the label storage=ssd set.

Task 8: Configuring Ingress
* Run a Pod with the name **lab168pod**, using the Nginx image.
» Expose this Pod using a NodePort type Service with the name **lab168svc**.
* Configure Ingress such that its web content is available on the path lab168.info/hi
* You will not have to configure an Ingress controller for this assignment, just the API resource is enough.

Task 9: Preparing for Node Maintenance
* Schedule node worker2 for maintenance in such a way that all running Pods are evicted.

Task 10: Scaling Applications
* Run a Deployment with the name **lab1610deploy**, using the Nginx image.
* Scale it such that it will always at least run 2 application instances, and never more than 6 application instances.

Task 11: Etcd Backup and Restore
* Before creating the backup, create a Deployment that runs Nginx.
* Create a backup of the etcd and write it to /tmp/etcdbackup.
* Delete the Deployment you just created.
* Restore the backup that you have created in the first step of this procedure and verify that the Deployment is available again.
