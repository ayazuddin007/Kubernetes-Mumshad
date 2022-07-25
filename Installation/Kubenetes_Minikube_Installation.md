Pre-Requisite
------------------
AMI: Ubuntu Server 18.04 LTS (Free Tier)
Instance Type	t3.micro (2 vCPU, 1GB Memory)
		t2.medium
Storage		8 GB (gp2)
Tags		K8S-Minikube
Security Group	K8S-Minikube-SG
	SSH  22  0.0.0.0/0
	Http   80  0.0.0.0/0
	Custom 30000 - 65535  0.0.0.0/0
Key Pair	ayzdn.pem

Note: 	- An update to Minikube required a minimum of 2 vCPUs
	-  changed the Instance Type from t2.micro (1 vCPU) to t3.micro (2 vCPU)/t2.medium


SSH to K8S-Minikube
---------------------------
	ssh -i "ayzdn.pem" ubuntu@<K8S-Minikube Public IP>

Become Root user
----------------------
	sudo su -
	apt-get upgrade -y && apt-get update -y
	
Install Docker
---------------
	apt-get install docker.io -y
	docker -v
It will show in output:
	Docker version 20.10.12, build 20.10.12-0ubuntu2~20.04.1

Install Kubectl
-----------------
	curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
	chmod +x ./kubectl
	mv ./kubectl /usr/local/bin/kubectl
	kubectl version --client
It will show in output:
	Client Version: version.Info{Major:"1", Minor:"24"}

	curl -LO "https://dl.k8s.io/release/v1.22.1/bin/linux/amd64/kubectl"
	sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
	kubectl version --client
It will show in output:
	Client Version: version.Info{Major:"1", Minor:"22"

	
Install Minikube
--------------------
	curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
	chmod +x minikube
	mv minikube /usr/local/bin/
	minikube version
It will show in output:
	minikube version: v1.26.0

Running Minikube
-------------------
	minikube start --vm-driver=none

Error:  	* Using the none driver based on user configuration
	X Sorry, Kubernetes v1.18.2 requires conntrack to be installed in root's path
Solution:  sudo apt-get install -y conntrack
		
Again Try
	minikube start --vm-driver=none

It will show in output:
	To use kubectl or minikube commands as your own user, you may need to relocate them. For example, to overwrite your own settings, run:
	*
  	- sudo mv /root/.kube /root/.minikube $HOME
  	- sudo chown -R $USER $HOME/.kube $HOME/.minikube

	As long as you see the message ‘Kubectl is now configured to use the cluster.’ you have successfully ran Minikube.



Error: Failed to enable unit: Unit file cri-docker.socket does not exist
Solution:  https://www.mirantis.com/blog/how-to-install-cri-dockerd-and-migrate-nodes-from-dockershim

Error:  X Exiting due to RUNTIME_ENABLE: Temporary Error: sudo crictl version: exit status 1
Solution: https://citizix.com/how-to-install-cri-o-container-runtime-on-ubuntu-20-04/

Check the status of Minikube
--------------------------------
	minikube status

It will show in output:
	minikube
	type: Control Plane
	host: Running
	kubelet: Running
	apiserver: Running	
	kubeconfig: Configured
	timeToStop: Nonexistent

Running our first container (Creating a Kubernetes Deployment)
------------------------------------------------------------------
Now, you can interact with your cluster using kubectl.
Let’s create a Kubernetes Deployment using an existing image named echoserver, which is a simple HTTP server and expose it on port 8080 using --port.

	kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.10 

It will show in output:
	deployment.apps/hello-minikube created

To check Deployment
	kubectl get deployment

	NAME             READY   UP-TO-DATE   AVAILABLE   AGE
	hello-minikube   1/1     1            1           37s

To access the hello-minikube Deployment, expose it as a Service: (Creating a Kubernetes Service)
----------------------------------------------------------------------
	kubectl expose deployment hello-minikube --type=NodePort --port 8080
The option --type=NodePort specifies the type of the Service.

It will show in output:
	service/hello-minikube exposed

To Check Service
	kubectl get service

	NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
	hello-minikube   NodePort    10.110.39.44   <none>        8080:31052/TCP   3m8s
	kubernetes       ClusterIP   10.96.0.1      <none>        443/TCP          15m

Get Pod list and Status
-------------------------
The hello-minikube Pod is now launched but you have to wait until the Pod is up before accessing it via the exposed Service.
Check if the Pod is up and running:
To Check Pod
	kubectl get pod
	
	NAME                              READY   STATUS    RESTARTS   AGE
	hello-minikube-6cb9d888c7-l4x96   1/1     Running   0          8m27s

If the output shows the STATUS as ContainerCreating, the Pod is still being created:

	NAME                              READY     STATUS              RESTARTS   AGE
	hello-minikube-3383150820-vctvh   0/1       ContainerCreating   0          3s

If the output shows the STATUS as Running, the Pod is now up and running:

	NAME                              READY     STATUS    RESTARTS   AGE
	hello-minikube-3383150820-vctvh   1/1       Running   0          13s

Opening the Service Exposed Port in Security Group(K8S-Minikube-SG)
------------------------------------------------------------------------------
K8S-Minikube-SG
	Custom TCP  31052  0.0.0.0/0

Because By exectuing
	kubectl get service
It will show in output:
	
	NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
	hello-minikube   NodePort    10.110.39.44   <none>        8080:31052/TCP   3m8s
	kubernetes       ClusterIP   10.96.0.1      <none>        443/TCP          15m

So Here 8080:31052/TCP means that
Port 31052  - Port of EC2 Instance
Port 8080   - Port of Pod

Get the URL of the exposed Service to view the Service details:
-----------------------------------------------------------------
	minikube service hello-minikube --url
It will show in output:
	http://172.31.10.73:31052
To Access Within the Minikube
	http://<Minikube Private IP>:<Port>
	http://172.31.10.73:31052
To Access Outside of Minikube(On Browser)
	http://<Minikube Public IP>:<Port>
	http://65.1.248.140:31052

Delete the hello-minikube Service
-------------------------------------
	kubectl delete services hello-minikube
It will show in output:
	service "hello-minikube" deleted

Delete the hello-minikube Deployment
---------------------------------------
	kubectl delete deployment hello-minikube
It will show in output:
	deployment.apps "hello-minikube" deleted

Stop the local Minikube cluster
----------------------------------
	minikube stop
It will show in output:
	* Stopping node "minikube"  ...
	* 1 nodes stopped.

Delete the local Minikube cluster
-----------------------------------
	minikube delete
It will show in output::
	Deleting "minikube" ...
	The "minikube" cluster has been deleted.

Restart the local Minikube cluster
-------------------------------------
After Stop Minikube
	minikube start
After Deleting Minikube
	minikube start --vm-driver=none


Another Example
===========================================
docker login
username: ayazway
pass:	
Login Successful

Pull docker image
--------------------
docker pull docker.io/ayazway/<Image Name>:latest 
docker images

Create Deployment
-------------------------
kubectl create deployment <Deployment Name> --image=docker.io/ayazway/<Image Name>:latest 
i.e.
 


Get Deploymet List
---------------------
kubectl get deploy

Create Service
-------------------
kubectl expose deployment <Deployment Name> --type=<Service Type> --port <Port Number>
Service Type=NodePort
Port Number=8080


Get Service List
---------------------
kubectl get service
  open port 32074 in Minikube-SG


To get exposed Service URL
--------------------------------
minikube service <Service Name> --url

Delete Service and Deploy
-----------------------------
kubectl delete service <Service Name>
kubectl delete deploy <Deployment Name>

Example
========
1)
----
kubectl create deployment sa --image=docker.io/ayazway/sampleapp
kubectl get deploy
kubectl get po
kubectl expose deployment sa --type=NodePort --port 8080
kubectl get svc
minikube service sa --url
curl http://172.31.32.87:30604/sampleapp

Browse: http://172.31.32.87:30604/sampleapp

kubectl delete svc sa
kubectl delete deploy sa

2) 
---
kubectl create deployment sa --image=docker.io/ayazway/simple-devops-image
kubectl get deploy
kubectl get po
kubectl expose deployment sa --type=NodePort --port 8080
kubectl get svc
minikube service sa --url
curl http://172.31.32.87:30604/webapp

Browse: http://172.31.32.87:30604/webapp

kubectl delete svc sa
kubectl delete deploy sa

3) 
---
kubectl create deployment sa --image=docker.io/ayazway/myimg
kubectl get deploy
kubectl get po
kubectl expose deployment sa --type=NodePort --port 8080
kubectl get svc
minikube service sa --url
curl http://172.31.32.87:30604/webapp

Browse: http://172.31.32.87:30604/webapp

kubectl delete svc sa
kubectl delete deploy sa




