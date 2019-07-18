<---------------------Setup Kubernetes Cluster on AWS--------------------------->
---------------------------------------------------------------------------------

curl https://s3.amazonaws.com/aws-cli/awscli-bundle.zip -o awscli-bundle.zip

apt install unzip python

unzip awscli-bundle.zip


./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws

aws configure
 
-------------------------------------------------------------------------------------

curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl
 
--------------------------------------------------------------------------------------
  
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64

chmod +x kops-linux-amd64

sudo mv kops-linux-amd64 /usr/local/bin/kops
 
---------------------------------------------------------------------------------------

aws s3 mb s3://dev.k8s.kopscluster.in
 
export KOPS_STATE_STORE=s3://dev.k8s.kopscluster.in

---------------------------------------------------------------------------------------- 
ssh-keygen

<----------Create kubernetes cluser---------->

kops create cluster --cloud=aws --zones=us-west-2c --name=dev.k8s.kopscluster.in --dns-zone=kopscluster.in --dns private

kops update cluster dev.k8s.kopscluster.in --yes

kops validate cluster

kubectl get nodes 

<------------------Deploying Nginx container on Kubernetes------------------->
------------------------------------------------------------------------------

kubectl run sample-nginx --image=nginx --replicas=2 --port=80

kubectl get pods

kubectl get deployments

kubectl expose deployment sample-nginx --port=80 --type=LoadBalancer

kubectl get services -o wide

<--------Deploying Wordpress Web Application with MySQL in Kubernetes------->
-----------------------------------------------------------------------------

git clone https://github.com/kiranpayyavuala/kubernetes.git

cd kubernetes

kubectl create secret generic mysql-pass --from-literal=password=password

kubectl get secrets

kubectl create -f pv.yaml

kubectl get pv

kubectl describe pv persistent-storage


kubectl create -f mysql-deployment.yaml

kubectl get service,pvc,deployment,pods


kubectl exec -it wordpress-mysql-abcd -- bash (Replace with mysql pod)

mysql -u root -ppassword

CREATE DATABASE wordpress;


kubectl apply -f wordpress-datavolume-claim.yaml

kubectl apply -f wordpress-deployment.yaml

kubectl apply -f DO-loadbalancer.yaml

kubectl get services

<-------------------------Deleting Kubernetes cluster------------------------->
------------------------------------------------------------------------------
kops delete cluster dev.k8s.kopscluster.in --yes
