Recently, during an audit, it was identified that there are some deployments on Kubernetes cluster which are no longer needed. Therefore, the team wants to delete some obslete deployments. Find below more details about the same.


There is a deployment named web-app-t2q4, delete the same.

Note: The kubectl utility on jump_host has been configured to work with the kubernetes cluster.

Deployment 'web-app-t2q4' has been deleted


kubectl delete deployment web-app-t2q4
