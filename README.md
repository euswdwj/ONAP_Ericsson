# ONAP_Ericsson
This project provides steps and artifacts for building ONAP by OOM and Integration Projects around it such as ESON.

It is based on [ OOM User Guide ](http://onap.readthedocs.io/en/latest/submodules/oom.git/docs/oom_user_guide.html)
 
 
![ESON](https://github.com/moffzilla/ONAP_LAB/blob/master/media/OOM_ONAP_Single.png)

 
## Requirements

- Running on Ubuntu Xenial (Cores=8 Mem=16G Root-Disk=160G - minimal)
- This project has tested it in OpenStack RHOSP 10 
- Make sure resources as flavor, image, SSH Keys, Security Groups referenced in the artifacts exist in the selected region.
- You’ll need OpenStack CLI installed ( tested with CLI version "openstack 3.11.0")
- More minimum requirements can be found examining the Playbooys.
- All use cases required ONAP Base Deployment ( unless indicated at sub-project section )

====================================
sudo yum update && sudo yum install git -y

Install the OpenStack CLI
https://docs.openstack.org/newton/user-guide/common/cli-install-openstack-command-line-clients.html

pip install python-openstackclient
pip install python-novaclient

   24  sudo pip install python-openstackclient
   25  sudo pip install python-novaclient
   26  sudo pip install python-glenceclient
   27  sudo pip install python-glanceclient
   28  sudo pip install python-neutronclient
   29  hostory
   30  history
   31  sudo pip install python-heatclient
   
Clone the git:
(no sudo) git clone https://github.com/euswdwj/ONAP_Ericsson.git 

From Horizon GUI source the scripts. 
nova list
openstack stack list
=========================================
## ONAP Base Deployment

Source your authentication credentials:

Execute:

	$ source keystonerc
	$ git clone https://github.com/euswdwj/ONAP_Ericsson.git
  	$ cd ONAP_Ericsson/
	

Deploy:

Create VM's on Openstack using below commands 

1) Rancher ( Master node )

	    $ openstack stack create -t deploy/oom_onap.yaml  --parameter "flavor=linux-medium" Rancher

2) ONAP Cluster node 

	    $ openstack stack create -t deploy/oom_onap.yaml  --parameter "flavor=ONAP_Client" ONAP-Stack

  
You can also create the stack at the Horizon Dashboard

Use the command 

	openstack stack list 

to verify that the stack was created for the instance.

Display:

Execute nova list to view list virtual machine instances and obtain the virtual machine instance UUID for the virtul machine created by the heat stack template.

        
    openstack stack show ONAP-stack | grep output_value

	  output_value: OOM
	  output_value: <instance IP> 

The command openstack stack show <instance UUID> can be also used

Full details:

	openstack stack show ONAP-stack

You can access by 

	ssh ubuntu@<instance IP>

Execute below command to setup Rancher node:

	sudo ansible-pull -U https://github.com/euswdwj/ONAP_Ericsson.git deploy/rancher.yml -vvv

Execute below steps to create hosts using Rancher GUI:

Access Rancher server via web browser
	
	Select “Manage Environments”
	Select “Add Environment”
	Add unique name for your new Rancher environment
	Select the Kubernetes template
	Click “create”
	Select the new named environment (ie. SB4) from the dropdown list (top left).
       
Add Kubernetes Host

	If this is the first (or only) host being added - click on the “Add a host” link
	Click on “Save” (accept defaults) otherwise select INFRASTRUCTURE→ Hosts and click on “Add Host”
	Enter the management IP for the k8s VM (e.g. 10.0.0.4) that was just created.
	Click on “Copy to Clipboard” button
	Click on “Close” button
	
	On Rancher web interface create a host in Kubernetes environment --> copy script to clipboard
	
	1. Run Kubernetes host sudo script on K8 instance
	2. K8 is installed by Rancher. See host progress in Rancher - INFRASTRUCTURE - Hosts.
	3. Monitor install progress by clicking on Kubernetes menu option. When you see >_CLI the K8 master is up

Execute below command to setup ONAP Worker node ( repeat same command on other ONAP cluster node ):

	sudo ansible-pull -U https://github.com/euswdwj/ONAP_Ericsson.git  deploy/onap_worker.yml -vvv

Wait for the script to complete.


Login to the new Kubernetes Host:

	Paste Clipboard content and hit enter to install Rancher Agent:
	Return to Rancher environment (e.g. SB4) and wait for services to complete (~ 10-15 mins)

	Click on CLI and then click on “Generate Config”
	Click on “Copy to Clipboard” - wait until you see a “token” - do not copy user+password - the server is not ready at that point
	ubuntu@sb4-kSs-1:~$ mkdir .kube
	ubuntu@sb4-kSs-1:~$ vi .kube/config
	Paste contents of Clipboard into a file called “config” and save the file:

Validate that kubectl is able to connect to the kubernetes cluster  and show running pods:

	ubuntu@sb4-k8s-1:~$ kubectl config get-contexts
	CURRENT   NAME   CLUSTER   AUTHINFO   NAMESPACE
	*         SB4    SB4       SB4
	ubuntu@sb4-kSs-1:~$


	ubuntu@sb4-k8s-1:~$ kubectl get pods --all-namespaces -o=wide
	NAMESPACE    NAME                                  READY   STATUS    RESTARTS   AGE   IP             NODE
	kube-system  heapster—7Gb8cd7b5 -q7p42             1/1     Running   0          13m   10.42.213.49   sb4-k8s-1
	kube-system  kube-dns-5d7bM87c9-c6f67              3/3     Running   0          13m   10.42.181.110  sb4-k8s-1
	kube-system  kubernetes-dashboard-f9577fffd-kswjg  1/1     Running   0          13m   10.42.105.113  sb4-k8s-1
	kube-system  monitoring-grafana-997796fcf-vg9h9    1/1     Running   0          13m   10.42,141.58   sb4-k8s-1
	kube-system  monitoring-influxdb-56chd96b-hk66b    1/1     Running   0          13m   10.4Z.246.90   sb4-k8s-1
	kube-system  tiller-deploy-cc96d4f6b-v29k9         1/1     Running   0          13m   10.42.147.248  sb4-k8s-1
	ubuntu@sb4-k8s-1:~$

Validate helm is running at the right version. If not, an error like this will be displayed:

	ubuntu@sb4-k8s-1:~$ helm list
	Error: incompatible versions c1ient[v2.9.1] server[v2.6.1]
	ubuntu@sb4-k8s-1:~$

Upgrade the server-side component of helm (tiller) via helm init –upgrade:

	ubuntu@sb4-k8s-1:~$ helm init --upgrade
	Creating /home/ubuntu/.helm
	Creating /home/ubuntu/.helm/repository
	Creating /home/ubuntu/.helm/repository/cache
	Creating /home/ubuntu/.helm/repository/local
	Creating /home/ubuntu/.helm/plugins
	Creating /home/ubuntu/.helm/starters
	Creating /home/ubuntu/.helm/cache/archive
	Creating /home/ubuntu/.helm/repository/repositories.yaml
	Adding stable repo with URL: https://kubernetes-charts.storage.googleapis.com
	Adding local repo with URL: http://127.0.0.1:8879/charts
	$HELM_HOME has been configured at /home/ubuntu/.helm.

	Tiller (the Helm server-side component) has been upgraded to the current version.
	Happy Helming!
	ubuntu@sb4-k8s-1:~$


It shows the status of the charts and associated Pods and Containers

	kubectl get pods --all-namespaces -o=wide

It shows the status of Pods and Containers at Kubernetes level.

Execute below command to setup onap on ANY ONE OF THE K8S node:

	sudo ansible-pull -U https://github.com/euswdwj/ONAP_Ericsson.git deploy/onap.yml -vvv

## To Remove

	openstack stack delete ONAP-stack --y
	
You can also delete the stack at the Horizon Dashboard.

## Sub-Projects

[ ESON ](https://github.com/moffzilla/ONAP_LAB/tree/master/eson)
MEDIATION

## OpenStack Appendix


You can upload the base Ubuntu 16.04 image as follows:

	openstack image create --private --disk-format qcow2 --container-format bare --file xenial-server-cloudimg-amd64-disk1.img ubuntu1604
	
Image: [ xenial-server-cloudimg-amd64-disk1.img ](http://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-amd64-disk1.img) 

