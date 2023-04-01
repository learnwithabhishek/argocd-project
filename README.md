# argocd-project

![Jenkins-Argocd-Pileline2](https://user-images.githubusercontent.com/110404399/229304623-41f24827-1894-4b00-a847-95c0cee69a67.jpg)

Docker image of vpro-app is built from cicd-Jenkins-Docker project and the latest image is used here to deploy on kubernetes cluster with Argocd. Argocd will maintain the state of deployment on kubernetes cluster and will use this repo as a single source of truth.

#Install awscli and configure it with IAM user of admin access.

sudo apt update 

sudo apt install awscli -y

aws configure            # (After that enter your user's access key id and secret access key)

#Install kubectl

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

#Install kops

curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64

chmod +x kops-linux-amd64

sudo mv kops-linux-amd64 /usr/local/bin/kops

#kops uses DNS for discovery, both inside the cluster and outside, so that you can reach the kubernetes API server from clients.

#Purchge a domain (ex. godaddy.com) then created a public hosted zone in aws Route53.

#In Godaddy created NS records for subdomain pointing to Route53 hosted zone NS servers.

#Created a S3 bucket to store the cluster's state.

#After completing all prerequisites create cluster using kops command
 
kops create cluster --name=k8.learnwithabhi.xyz --state=s3://mykopsbucketfork8 --zones=us-east-1a --node-count=1 --node-size=t3.small --master-size=t3.medium --dns-zone=k8.learnwithabhi.xyz --node-volume-size=8 --master-volume-size=8

kops update cluster --name k8.learnwithabhi.xyz --state=s3://mykopsbucketfork8 --yes --admin

kops validate cluster --state=s3://mykopsbucketfork8

# Install Argocd

kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Download & install Argocd Cli

curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64

sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd

rm argocd-linux-amd64

#To access argocd api-server change the argocd-server service type to LoadBalancer

kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

![argocd-loadbalancer](https://user-images.githubusercontent.com/110404399/229288652-4a797c07-1af7-4355-9be3-b7115c840db4.jpg)

#To login copy loadbalancer address and paste in browser. username is admin, to get initial password run below command:

argocd admin initial-password -n argocd

![argocd-login-page](https://user-images.githubusercontent.com/110404399/229288267-f9053e9e-53d6-43fd-b525-495c4d87ce38.jpg)

#To change password go to user info and click on update password

![argocd-password-change](https://user-images.githubusercontent.com/110404399/229288339-bd90278f-a16e-43d7-bdef-06aaea54ab0d.jpg)

#Clone this git repo & create application using application.yaml file

git clone https://github.com/learnwithabhishek/argocd-project.git

kubectl apply -f application.yaml

![argocd-browser-application-successful](https://user-images.githubusercontent.com/110404399/229289006-8197b4f0-0314-4e4a-9e1a-f49a5ca58e21.jpg)

#We can check details of deployments, replicaset, pods etc by clicking on the application.

![argocd-browser-details1](https://user-images.githubusercontent.com/110404399/229289116-82807ed6-c427-4e85-a702-f0299b4367ad.jpg)

![argocd-browser-details2](https://user-images.githubusercontent.com/110404399/229289130-ecced62a-5033-448e-acd9-15c641e2a30f.jpg)

#To get details about prod cluster enter below command:

kubectl get all -n prod

![k8cluster-after-deploy](https://user-images.githubusercontent.com/110404399/229289257-63bd763a-a613-4e8e-8d47-657d51314e17.jpg)

#Access your vpro-app with loadbalancer external-ip in browser

![vpro-app-browser](https://user-images.githubusercontent.com/110404399/229289334-32a8ed69-9be7-4ff7-91eb-254f3431824d.jpg)

#Can login with username: admin_vp , password: admin_vp

![vpro-app-browser2](https://user-images.githubusercontent.com/110404399/229289390-547e1f3d-005e-48c8-9229-3ee54f8b230b.jpg)

#Click on "All Users" , it will show list of users

![vpro-app-browser-allusers](https://user-images.githubusercontent.com/110404399/229289612-04aea855-eb17-4914-ae73-ea4df3d81039.png)

#click on any user id, data will be fetched from database and inserted in cache

![vpro-app-browser-allusers2](https://user-images.githubusercontent.com/110404399/229289686-e1ecff77-12e2-4d70-8db1-f8fede0366d9.jpg)

#Go back and click same user id again, now data will be displayed from cache 

![vpro-app-browser-allusers3](https://user-images.githubusercontent.com/110404399/229289758-f2e2f1da-f98e-4396-bf9d-8eabb6ff67cc.jpg)

#Return back to main page of app and click on RabbitMq to initiate it

![rmq-tested](https://user-images.githubusercontent.com/110404399/229290031-07e579a5-a5b9-4194-bae4-4452fc1bb471.jpg)

#After completing the project delete the application and then delete kubernetes cluster on aws

kops delete cluster --name k8.learnwithabhi.xyz --state=s3://mykopsbucketfork8 --yes
