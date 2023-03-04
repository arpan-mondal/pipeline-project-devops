# End-to-End-CI-CD-Project
This Project basically provide an End-to-End-CI/CD where we Checkout SCM, Sonar Quality Check of the code, Check Quality Gate Status, Docker build and Docker push to Nexus, Identifying misconfiguration using datree in Helm Charts, Pushing helm charts to nexus and finally a Declarative Post Action.

# INITIAL THOUGHT-PROCESS
![initial_thought](https://user-images.githubusercontent.com/66848339/222870337-3a4b4048-001e-418a-af51-8570afca8854.JPG)

# HIGH-LEVEL DESIGN OF THE PROJECT
![High-Level-Design@2x](https://user-images.githubusercontent.com/66848339/222868569-810c4d1e-5bc6-42ca-a9d9-e48fb3535d3d.png)

# LOW-LEVEL DESIGN OF THE PROJECT 
![2023-03-04 07 08 31](https://user-images.githubusercontent.com/66848339/222868771-f1dac6d2-b2ba-43bc-a6cf-1cce6875daa1.jpg)

# TOOLS USED IN THE PROJECT
## Git/GitHub
- Git is Distributed Version Control System
- It is one of the most popular SCM(Source Code mmmnt tool)
- GitHub used here to store the application code, Dockerfile, K8s menifest, Jenkinsfiles and many other code related files.

## Maven
- Maven is a build automation tool used primarily for JAVA projects. Maven can also be used to build and manage projects written in C#, Ruby, Scala, and other languages.
- In CI/CD I am using Maven sonarQube plugin which helps in static code analysis and also, used to build application code. 

## SonarQube
- SonarQube is a open-source platform developed by SonarSouce for continous inspection of code quality to perform autometic reviews with static code analysis to detect bugs, Code Coverage, code smells on 29 programming languages and you can add custom rule using sonarqube dashboard.  
- In CI/CD I am using sonarqube plugin to verify or validate against sonar plugin.

## Jenkins
- Jenkins is a open-source automation server. It helps in automate the parts of software development related to building, testing, and deploying, facilitating continuous integration and continous delivery. 
- In CI/CD I am using Jenkins to integrate all the tools eg : GitHub,Gradle,SonarQube, Docker, Kubernetes and Datree. 

## Docker 
- Docker is a set of platform as a service products that use OS-level virtualization to deliver software in package called containers. The service has both free and premium tiers. Ths software that hosts the containers is called Docker Engine.
- In CI/CD I am writing Dockerfile to containerize my application and create image and push to nexus repository to maintain different version of the images.

## Nexus 
- Nexus by Sonartype is a repository manager that organizes, stores and distribute artifacts needed for development. With Nexus, developers can completely control access to, and deployment of, every artifact in a organization from a single location, making it easier to distribute software.
- In CI/CD I am using nexus to store Docker Images and Helm Charts.

## Kubernetes
- Kubernetes is a open-source container orchestration system for automation, software development, scaling, and management.
- In CI/CD I will be deploying application on k8s cluster using Helm Charts.

## Helm
- Helm is a package manager for Kubernetes, which allows you to define, install, and manage applications as packages called charts. Helm charts are collections of YAML templates that define Kubernetes resources such as deployments, services, and ingress rules, along with any necessary configurations and dependencies.
- I am using Helm Chats to automate the deployment of applications on Kubernetes clusters.

## datree.io
- Datree is a policy enforcement tool for Kubernetes. It provides a way to manage and enforce policies for Kubernetes deployments, and helps teams ensure compliance and best practices.
- Check the validation of the code against some rules and regulation.

## 
##

## Server Management & Installation
- In total we are creating 5 server for Jenkins, SonarQube, Nexus, 

## 1st Server - Jenkins
- ubuntu20
- t2.medium
- Select the key pair if you have.
- Minimum 20 GB configuration storage (need to install some packages)
- Jenkins Installation Guide.
```
#!/bin/bash

sudo apt update -y

sudo apt install default-jre -y

java -version

sudo apt update -y

wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -

sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'

sudo apt update -y

sudo add-apt-repository universe -y

sudo apt-get install jenkins -y

sudo service jenkins start

sudo cat /var/lib/jenkins/secrets/initialAdminPassword

#sudo service jenkins status


#34.232.26.121
```
- Now from IP you will not able to access due to the security group, therefore allow ingress rules
- Head to edit inbound rule.
- Enable all the traffic & source anywhere.
- Now copy the IP again or reload now you will be able to install.

## 2nd Server - SonarQube
- ubuntu20
- t2.medium
- If you have jenkins key pair and use the same key-pair
- Minimum 20 GB configuration storage
- when the jenkins is installed configure the sonarqube.
- In sonarqube install the docker.
```
#!/bin/bash
sudo apt update -y

sudo apt install apt-transport-https ca-certificates curl software-properties-common -y

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable" -y

sudo apt update -y

apt-cache policy docker-ce -y

sudo apt install docker-ce -y

#sudo systemctl status docker

sudo chmod 777 /var/run/docker.sock

docker info
```
- Now create a container 
```
docker run -d --name sonarqube -p 9000:9000 -p 9092:9092 sonarqube
```
- Run the container in detached mode so it will keep on running. 
- Now Check
```
docker container ls
```
- To access take the IP of sonarqube and change the inbound rules as we did earlier.
- Now reload your sonarqube will be ready up and running.
- Username - admin Password - admin, and change the password.

## 3rd Server - Nexus
- ubuntu20
- t2.medium
- If you have jenkins key pair and use the same key-pair
- Minimum 20 GB configuration storage
- Installation Script
```
apt-get update -y

echo "Install Java"
apt-get install openjdk-8-jdk -y
java -version

echo "Install Nexus"
useradd -M -d /opt/nexus -s /bin/bash -r nexus
echo "nexus ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/nexus

mkdir /opt/nexus
wget https://sonatype-download.global.ssl.fastly.net/repository/downloads-prod-group/3/nexus-3.29.2-02-unix.tar.gz

tar xzf nexus-3.29.2-02-unix.tar.gz -C /opt/nexus --strip-components=1

chown -R nexus:nexus /opt/nexus

nano /opt/nexus/bin/nexus.vmoptions

```
- After second last command do 'vi' and whearever your see ../sonartype make it to ./sonartype
- Start the nexus server set the name 'nexus' in vi /opt/nexus/bin/nexus.rc

```
sudo -u nexus /opt/nexus/bin/nexus start
```

- Now access the server and edit the inbound rules.

```
https://www.howtoforge.com/how-to-install-and-configure-nexus-repository-manager-on-ubuntu-20-04/
```

## 4nd and 5th Server - Master & Node
- Ubuntu20
- t3.medium
- apply the same key pair which you have used earlier. 
- configure store to 20GB or more 
- No. of instances = 2
- Now SSH to master and node and enable the security and inbound rule.
- On master and worker node

```
sudo su
apt-get update  
apt-get install docker.io -y
service docker restart  
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -  
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" >/etc/apt/sources.list.d/kubernetes.list
apt-get update
apt install kubeadm=1.20.0-00 kubectl=1.20.0-00 kubelet=1.20.0-00 -y
```

- On master 

```
kubeadm init --pod-network-cidr=192.168.0.0/16
```
- Copy the command of kubeadm join and paste it on node.
- On master 

```
  exit
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
    
- On master

```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.49.0/deploy/static/provider/baremetal/deploy.yaml
```

- To check the node is connected to the master or not.

```
kubectl get nodes
```

- Our Kubernetes installation and configuration are complete.

# Creating Docker hosted repository in Nexus and pushing the docker image through Jenkins

## Pre-requisite

 - We need to have Jenkins and nexus server up and running (by default Jenkins runs on 8080 and nexus at 8081).
 - On Jenkins host we need to install docker 

## Initial setup 

In nexus click on gear button --> click on repositories --> click on create repository ( below image can help in creating )

![nexus1](https://user-images.githubusercontent.com/29688323/136389371-2bc67fb7-8a21-47d8-82bf-59753d6a1ee4.JPG)

once we click on create repository ( types of repository will be listed ) --> click on docker(hosted)

![nexus2](https://user-images.githubusercontent.com/29688323/136389787-76a847cb-df1b-4a42-b031-2aa364fa76ad.JPG)

fill out the details in Name ( unique name ), enable checkbox beside to HTTP and enter a valid port ( preferred 8083 ) once that click on create repository   

![nexus3](https://user-images.githubusercontent.com/29688323/136390256-f6f67caf-a86a-4049-9712-3f43043fbcbe.JPG)
![nexus4](https://user-images.githubusercontent.com/29688323/136390268-16f2f9dd-2738-4270-9200-055808a604e9.JPG)

Once this set up is done in jenkins host we need to setup Insecure Registries. to do that we need to edit or if not present create a file ```/etc/docker/daemon.json``` in that file add details of nexus 

```
{ "insecure-registries":["nexus_machine_ip:8083"] }
```

once that's done we need to execute ```systemctl restart docker``` this is to apply new changes, also we can verify whether registry is added or not by executing ```docker info```
 
once this is done from jenkins host you can try ```docker login -u nexus_username -p nexus_pass nexus_ip:8083```








