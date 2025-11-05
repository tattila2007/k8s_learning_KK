# 1 / 14
**Weight: 4**
* From student-node ssh cluster1-controlplane to solve this question.
* Create a StorageClass named local-sc with the following specifications and set it as the default storage class:
* The provisioner should be kubernetes.io/no-provisioner
* The volume binding mode should be WaitForFirstConsumer
* Volume expansion should be enabled

* Is the StorageClass local-sc created?
* Is Provisioner kubernetes.io/no-provisioner used?
* Is the volume binding set to WaitForFirstConsumer?
* Is local-sc set to the default storage class?

# 2 / 14
**Weight: 8**
* From student-node ssh cluster1-controlplane to solve this question.
* Create a deployment named logging-deployment in the namespace logging-ns with 1 replica, with the following specifications:
* The main container should be named app-container, use the image busybox, and should start by creating a log directory /var/log/app and run the below command to simulate generating logs :

while true; do 
  echo "Log entry" >> /var/log/app/app.log
  sleep 5
done

* Add a co-located container named log-agent that also uses the busybox image and runs the commands:

touch /var/log/app/app.log
tail -f /var/log/app/app.log

* log-agent logs should display the entries logged by the main app-container
* Co-located container displays logs from main container
* Co-located container properly configured

# 3 / 14
**Weight: 8**
* From student-node ssh cluster1-controlplane to solve this question.
* A Deployment named webapp-deploy is running in the ingress-ns namespace and is exposed via a Service named webapp-svc.
* Create an Ingress resource called webapp-ingress in the same namespace that will route traffic to the service. The Ingress must:

* Use pathType: Prefix
* Route requests sent to path / to the backend service
* Forward traffic to port 80 of the service
* Be configured for the host kodekloud-ingress.app
* Test app availablility using the following command:

curl -s http://kodekloud-ingress.app/


* Ingress exposed and serving traffic via kodekloud-ingress.app host

# 4 / 14
**Weight: 8**
* From student-node ssh cluster1-controlplane to solve this question.
* Create a new deployment called nginx-deploy, with image nginx:1.16 and 1 replica. Next, upgrade the deployment to version 1.17 using rolling update.

* Deployment: nginx-deploy, Image: nginx:1.16
* Image: nginx:1.16
* Version upgraded to 1.17

# 5 / 14
**Weight: 8**
* From student-node ssh cluster1-controlplane to solve this question.
* Create a new user called john. Grant him access to the cluster using a csr named john-developer. Create a role developer which should grant John the permission to create, list, get, update and delete pods in the development namespace . The private key exists in the location: /root/CKA/john.key and csr at /root/CKA/john.csr.

* Important Note: As of kubernetes 1.19, the CertificateSigningRequest object expects a signerName.
* Please refer to the documentation to see an example. The documentation tab is available at the top right of the terminal.

* CSR: john-developer Status:Approved
* Role Name: developer, namespace: development, Resource: Pods
* Access: User 'john' has appropriate permissions

# 6 / 14
**Weight: 6**
* From student-node ssh cluster1-controlplane to solve this question.
* Create an nginx pod called nginx-resolver using the image nginx and expose it internally with a service called nginx-resolver-service. Test that you are able to look up the service and pod names from within the cluster. Use the image: busybox:1.28 for dns lookup. Record results in /root/CKA/nginx.svc and /root/CKA/nginx.pod

* Pod: nginx-resolver created
* Service DNS Resolution recorded correctly
* Pod DNS resolution recorded correctly

# 7 / 14
**Weight: 8**
* From student-node ssh cluster1-controlplane to solve this question.
* Create a static pod on cluster1-node01 called nginx-critical with the image nginx. Make sure that it is recreated/restarted automatically in case of a failure.
* For example, use /etc/kubernetes/manifests as the static Pod path.

* Is the static pod configured under /etc/kubernetes/manifests?
* Is pod nginx-critical-cluster1-node01 up and running?

# 8 / 14
**Weight: 6**
* From student-node ssh cluster1-controlplane to solve this question.
* Create a Horizontal Pod Autoscaler with name backend-hpa for the deployment named backend-deployment in the backend namespace with the webapp-hpa.yaml file located under the root folder.
* Ensure that the HPA scales the deployment based on memory utilization, maintaining an average memory usage of 65% across all pods.
* Configure the HPA with a minimum of 3 replicas and a maximum of 15.

* Is backend-hpa HPA deployed in backend namespace?
* Is deployment configured for metrics memory utilization?

# 9 / 14
**Weight: 8**
* From student-node ssh cluster2-controlplane to solve this question.
* As a Kubernetes administrator, you are unable to run any of the kubectl commands on the cluster. Troubleshoot the problem and get the cluster to a functioning state.

* Kubelet service is running
* kubectl is functional

# 10 / 14
**Weight: 10**
* From student-node ssh cluster1-controlplane to solve this question.
* Modify the existing web-gateway on cka5673 namespace to handle HTTPS traffic on port 443 for kodekloud.com, using a TLS certificate stored in a secret named kodekloud-tls.

* Is the web gateway configured to listen on the hostname kodekloud.com?
* Is the HTTPS listener configured with the correct TLS certificate?

# 11 / 14
**Weight: 8**
* From student-node ssh cluster1-controlplane to solve this question.
* On the cluster, the team has installed multiple helm charts on a different namespace. By mistake, those deployed resources include one of the vulnerable images called kodekloud/webapp-color:v1. Find out the release name and uninstall it.

* Is helm release uninstalled?

# 12 / 14
**Weight: 6**
* From student-node ssh cluster1-controlplane to solve this question.
* You are requested to create a NetworkPolicy to allow traffic from frontend apps located in the frontend namespace, to backend apps located in the backend namespace, but not from the databases in the databases namespace. There are three policies available in the /root folder. Apply the most restrictive policy from the provided YAML files to achieve the desired result. Do not delete any existing policies.

* Correct NetworkPolicy applied
* Incorrect NetworkPolicy is not applied
* Second incorrect NetworkPolicy is not applied

# 13 / 14
**Weight: 6**
* From student-node ssh cluster3-controlplane to solve this question.
* Your cluster has a failed deployment named backend-api with multiple pods. Troubleshoot the deployment so that all pods are in a running state. Do not make adjustments to the resource limits defined on the deployment pods.
* NOTE: A ResourceQuota named cpu-mem-quota is applied to the default namespace and should not be edited or modified.

* Are all pods currently running?
* Are the limits unchanged?
* Is the ResourceQuota unchanged?

# 14 / 14
**Weight: 6**
* From student-node ssh cluster4-controlplane to solve this question.
* Utilize the official Calico definition file, available at:

https://raw.githubusercontent.com/projectcalico/calico/v3.29.3/manifests/tigera-operator.yaml

to deploy the Calico CNI on the cluster.

* Make sure to configure the CIDR to 172.17.0.0/16
* After the CNI installation, verify that pods can successfully communicate.
* Custom Definitions for calico can be retrieved via:

curl https://raw.githubusercontent.com/projectcalico/calico/v3.29.2/manifests/custom-resources.yaml -O

* Is Calico cni installed?
* Can pods communicate with one another?
* Ingress exposed and serving traffic via kodekloud-ingress.app host
