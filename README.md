                                          #Setup Kubernetes Cluster on AWS
```
sudo apt update
sudo apt -y upgrade

curl https://s3.amazonaws.com/aws-cli/awscli-bundle.zip -o awscli-bundle.zip
apt install unzip python
unzip awscli-bundle.zip
./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
           (or)

sudo apt-get install awscli -y
aws --version

aws configure
```
---------------------------------------------------------------------------------------------------------------------------
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

```
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x kops-linux-amd64
sudo mv kops-linux-amd64 /usr/local/bin/kops
```

```
aws s3 mb s3://dominar.in
export KOPS_STATE_STORE=s3://dominar.in
ssh-keygen
```

                                               #Create kubernetes cluser
```
kops create cluster --cloud=aws --zones=us-west-2c --name=dominar.in --dns-zone=dominar.in --dns private
```
or
```
kops create cluster \
     --name=dominar.in \
     --zones=us-east-1a \
     --master-size="t2.medium" \
     --node-size="t2.medium" \
     --node-count="2" \
     --dns-zone=dominar.in \
     --dns=private
```
or
```
kops create cluster   --cloud=aws       --zones=us-east-2a,us-east-2b      --name=kopscluster.in       --master-size="t2.micro"       --node-size="t2.micro"       --node-count="2"       --dns-zone=kopscluster.in       --dns private
```
                                          #For edit cluster(Config) and Update cluster
```						    
kops edit cluster 
kops update cluster dominar.in --yes
kops validate cluster
kubectl get nodes 
```

                                                  #To connect Master or node
```
ssh -i .ssh/id_rsa admin@Master_ip
ssh -i .ssh/id_rsa admin@Node_ip
```


                                             # Deploying Nginx container on Kubernetes
					  
```
kubectl run sample-nginx --image=nginx --replicas=2 --port=80

kubectl get pods

kubectl get deployments

kubectl expose deployment sample-nginx --port=80 --type=LoadBalancer

kubectl get services -o wide
```
                                #Deploying Wordpress Web Application with MySQL in Kubernetes
			        
```
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
```
                                               #Deleting Kubernetes cluster
					       
```
kops delete cluster dominar.in --yes

```
