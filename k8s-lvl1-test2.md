2 / 10
Weight: 10
The Nautilus application development team aims to test a straightforward deployment by creating an Nginx-based Pod on the Kubernetes cluster. The specifications for this deployment are as follows:


Create a Pod named dummy-nginx-httpd-t1q6, it must use nginx:stable-alpine3.17-slim image. Finally, ensure the Pod remains in the Running state.

Note: The kubectl utility on jump_host has been configured to work with the kubernetes cluster.

Pod 'dummy-nginx-httpd-t1q6' exists

Pod is in 'Running' state

Pod is using image 'nginx:stable-alpine3.17-slim'

apiVersion: v1
kind: Pod
metadata:
  name: red-devops-t1q5
  labels:

spec:
  containers:
  - name: red-main-devops-t1q5
    image: fedora:latest

apiVersion: v1
kind: Pod
metadata:
  name: dummy-nginx-httpd-t1q6
spec:
  containers:
  - name: dummy-nginx-httpd-t1q6
    image: nginx:stable-alpine3.17-slim
    command: ['sh', '-c', 'sleep 7200']
