                                          #Setup Kubernetes Cluster on AWS
										  
### Launch Linux EC2 instance(ubuntu/Redhat/centos) in AWS (4cpus and 16Gb RAM) as a Master

```
sudo apt update
sudo apt -y upgrade(It will upgrade the python,git,java ..... )
```

### Create and attach IAM role to EC2 Instance or create user and pass the access key and secret key. 
Kops need permissions to access EC2,S3,vpc,Route53,IAM and Autoscaling ...

```	
Attach IAM role to our server
Actions-->Instance settings--> Attach/Replace IAM Role--> select our IAM role

            (or)

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

### Install kubectl(It will download Stable version)

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

### Install Kops(It will download latest version)
```
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x kops-linux-amd64
sudo mv kops-linux-amd64 /usr/local/bin/kops
```

#Create S3 bucket in AWS
S3 bucket is used by kubernetes to persist cluster state
**Note:**  Make sure you choose bucket name that is uniqe accross all aws accounts
```
aws s3 mb s3://dominar.in
export KOPS_STATE_STORE=s3://dominar.in
```
                        or

#Configure environment variables.
Open .bashrc file 
```
aws s3 mb s3://dominar.in --region us-east-1
vi ~/.bashrc
```
Add following content into .bashrc, you can choose any arbitary name for cluster and make sure buck name matches the one you created in previous step.

```sh
export KOPS_CLUSTER_NAME=dominar.in
export KOPS_STATE_STORE=s3://dominar.in
```
Then running command to reflect variables added to .bashrc
```
source ~/.bashrc
```
#Create ssh key pair
This key is used for connect kubernetes nodes

```
ssh-keygen
```

#Create a Route53 private hosted zone (you can create Public hosted zone if you have a domain)
  Head over to aws Route53 and create hostedzone
  Choose name for example (dominar.in)
  Choose type as privated hosted zone for VPC
  Select default vpc in the region you are setting up your cluster and create the hosted zone

                                           ###Create kubernetes cluser

#Create kubernetes cluser definition.
```
kops create cluster --cloud=aws --zones=us-east-1a --name=dominar.in --dns-zone=dominar.in --dns private
```
                                          or
```
kops create cluster \
     --state "s3://dominar.in" \
     --name=dominar.in \
     --zones=us-east-1a,us-east-1b \
     --master-size="t2.medium" \
     --master-zones=us-east-1a,us-east-1b,us-east-1f \
     --node-size="t2.medium" \
     --node-count="2" \
     --master-count 3 \
     --ssh-public-key ~/.ssh/id_rsa.pub \
     --dns-zone=dominar.in \
     --dns=private
```
                                           or
```
kops create cluster   --cloud=aws       --zones=us-east-1a,us-east-1b      --name=dominar.in       --master-size="t2.micro"       --node-size="t2.micro"       --node-count="2"       --dns-zone=dominar.in       --dns private
```

```
--state is the S3 bucket, where kops stores the state files
--zones we specify two availability zones in the same region, us-east-1a and us-east-1d
--master-count the number of masters must be odd (1,3,5...), so if we want to have a HA cluster we need at least 3 masters. 
  Since for simplicity we've chosen to use just two AZ, one of the two zones will have two masters.
--master-size this is the type of EC2 Instance for the master servers. For a medium size cluster I usually use C4/C5.
  large masters, but for this example t2.micro works well. You find t2 instances pricing here.
--node-count and --node-size in this example we just need two nodes, which in this case are two t2.micro instances.
--name the name of our cluster, which is also a real subdomain which will be created on route53.


The nodes are the kubernetes workers, the servers where we run our containers. 
Usually these servers are much bigger than the masters, since is where most of the load located.

If you run the command without --yes, kops prints the list of the whole actions is going to do on your AWS account.
Creation of IAM roles, security groups, Volumes, EC2 instances etc.
It's usually a good practice to take a look at what kops is going to do, before running the command with the --yes option.

```



#Create kubernetes cluster

```
kops update cluster dominar.in --yes
```

Above command may take some time to create the required infrastructure resources on AWS. 
Execute the validate command to check its status and wait until the cluster becomes ready
This will create the Kubernetes cluster and will also create two instance groups: 
one each for the master node and the worker nodes.
Verify that the instance groups have been created.

```
kops get ig --name dominar.in
```

#For validate cluster and get cluster

For the above above command, you might see validation failed error initially when you create cluster and it is expected behaviour,
you have to wait for some more time and check again.

```						     
kops validate cluster
kubectl get nodes 
```

## Update Nodes and Master in the cluster
We can change numner of nodes and number of masters using following commands
```
   kops edit cluster
   kops edit ig nodes change minSize and maxSize to 0
   kops get ig- to get master node name
   kops edit ig - change min and max size to 0
   kops update cluster --yes
 
```

#To connect Master or node

```
ssh -i .ssh/id_rsa admin@Master_ip
ssh -i .ssh/id_rsa admin@Node_ip
```

###Deploying Nginx container on Kubernetes
					  
```
kubectl run sample-nginx --image=nginx --replicas=2 --port=80

kubectl get pods

kubectl get deployments

kubectl expose deployment sample-nginx --port=80 --type=LoadBalancer

kubectl get services -o wide
```

##Deploying Wordpress Web Application with MySQL in Kubernetes
			        
# Infrastructure Diagram:
On the left is a traditional diagram for this 3-tier web application. On the right, you see how each part of that infrastructure maps to kubernetes concepts.

Don't worry if this doesn't make sense at the beginning.

    Some config data            (k8s ConfigMaps and Secrets)

    MySQL Container             (k8s replicaset)
    MySQL Service               (k8s service)
        |
        |
    WordPress Container         (k8s deployment)
    [ apache and php-fpm ]
        |
        |
    DO Loadbalancer             (k8s service)

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

```
Get a shell inside the mysql container, log into mysql, and set up the DB:
```
kubectl get pods
kubectl exec -it mysql-abcde -- bash    # replace mysql-abcde with the actual pod name
```
# use the root password you created earlier (mysql-pass)
# use the following to decode your password if needed
# echo -n YOURBASE64PASSWORD | base64 -d
```
    mysql -u root -ppassword
```
    # In your mysql shell:
```
    CREATE DATABASE wordpress;
```
Ctrl-d to get back out.

```
kubectl apply -f wordpress-datavolume-claim.yaml

kubectl apply -f wordpress-deployment.yaml

kubectl apply -f DO-loadbalancer.yaml
```

## View your work!
Grab the load balancer's external IP here:

````
kubectl get services
```
You can also see this in aws console EC2--> loadbalancer and take the Record name
```
                          #Remove all the deployment
```
```
kubectl delete daemonsets,replicasets,services,deployments,pods,rc --all
````

   #Delete S3 bucket
```
aws s3 rb s3://dominar.in

```

   #Deleting Kubernetes cluster
					       
```
kops delete cluster dominar.in --yes

```
