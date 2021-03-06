gcloud compute instances create myinstance - create instance in the cluster
gcloud compute firewall-rules create allow-80 --allow tcp:80 - create firewall rule on port 80
gcloud compute ssh myinstance - create ssh key and enter to instance

sudo su - - change user to root

install nginx

# apt-get update
# apt-get install -y nginx
# service nginx start
# exit

wget -q -O - localhost:80 - test instance
gcloud compute instances list - return list of already created instances

create startup file startup.sh with text
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"$HOSTNAME"'/' /var/www/html/index.nginx-debian.html

gcloud compute instances create nginx --metadata-from-file startup-script=startup.sh - set startup file

Instance template creation
gcloud compute instance-templates create nginx-template --metadata-from-file startup-script=startup.sh

create a target pool. A target pool allows us to have a single access point to all the instances in a group and is necessary for load balancing in the future steps.
gcloud compute target-pools create nginx-pool

create an instance group using the template:
gcloud compute instance-groups managed create nginx-group \
         --base-instance-name nginx \
         --size 2 \
         --template nginx-template \
         --target-pool nginx-pool

create a network load balancer targeting our instance group:
gcloud compute forwarding-rules create nginx-lb --ports 80 --target-pool nginx-pool

gcloud compute backend-services get-health - get status of instances
gcloud compute firewall-rules list - get list of firewall rules
gcloud compute forwarding-rules list - list of load balancers

gcloud compute forwarding-rules delete nginx-lb - delete LB
gcloud compute instance-groups managed delete nginx-group - delete group
gcloud compute target-pools delete nginx-pool - delete pool
gcloud compute instance-templates delete nginx-template - delete template
gcloud compute firewall-rules delete allow-80 - delete firewall rule
gcloud compute instances delete myinstance - delete instance


Create your Kubernetes Cluster

gcloud config set compute/zone europe-west11-b
gcloud config set compute/region europe-west11
gcloud container clusters create guestbook --num-nodes 3 - create a cluster

To deploy NGINX, run:
kubectl run nginx --image=nginx --replicas=3

kubectl get pods -o wide - status of deployment

kubectl expose deployment nginx --port=80 --target-port=80 --type=LoadBalancer - expose NGINX cluster as external service
kubectl get service nginx - get services list

kubectl delete service nginx - delete service
kubectl delete deployment nginx - delete deployment

create a pod using kubectl, the Kubernetes CLI tool:
kubectl create -f redis-pod.yaml

kubectl get pods - get list of pods
kubectl describe pod <pod-name> | grep Node - find the node the pod is running

gcloud compute ssh <node-name> - ssh to node name
sudo docker ps - see actual pod (list of containers)

kubectl create -f redis-service.yaml - create a service
kubectl get service - get list of services

gcloud compute disks create mysql-disk --size 20GB - create disk

deploy both the MySQL Pod and the Service with a single command:
kubectl create -f mysql.yaml -f mysql-service.yaml

deploy the <Name> Replication Controller:
kubectl create -f helloworldservice-controller-v1.yaml

kubectl logs -f helloworldservice-controller-v1-XXXXX - get pod logs

create the Guestbook Service replication controller and service too!:
kubectl create -f guestbookservice-controller.yaml -f guestbookservice-service.yaml

login to pod:
kubectl exec -ti mysql /bin/bash

ps auwx - list of users and entered commands
hostname -i - IP address

Scaling the number of replicas of our <Name> controller is as simple as running :
kubectl scale rc helloworldui-controller-v1 --replicas=12

find the Compute Engine Instance Group that's managing the Kubernetes nodes (the name is prefixed with "gke-")
gcloud compute instance-groups list

resize the number of nodes by updating the Instance Group size:
gcloud compute instance-groups managed resize gke-guestbook-a3e896df-group --size 5

gcloud container clusters describe guestbook | egrep "password"