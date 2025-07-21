🚀 Dockerized Java Microservices Deployment with Kubernetes (Minikube & EKS)
This project demonstrates how to containerize and deploy a Java-based microservices architecture using Docker, Kubernetes (Minikube & Amazon EKS), and Docker Hub for image hosting.

📦 Microservices Overview
The application consists of three interdependent microservices:

shopfront – The frontend application

productcatalogue – Provides product data

stockmanager – Manages stock information

📌 Prerequisites
Java & Maven installed

Docker installed and running

Docker Hub account

AWS CLI configured (for EKS setup)

IAM role with admin access (for EKS)

SSH key pair for EC2 access

🔧 Local Development Setup with Minikube
Step 1: Install kubectl
bash
Copy
Edit
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.20.4/2021-04-12/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin
cp ./kubectl $HOME/bin/kubectl
export PATH=$HOME/bin:$PATH
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
kubectl version --short --client
Step 2: Install Docker
bash
Copy
Edit
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
Step 3: Install Minikube
Follow the official guide: https://minikube.sigs.k8s.io/docs/start/

☁️ EKS Cluster Setup on AWS
Step 1: Launch EC2 Instance
Use t2.medium instance

Attach an IAM role with AdminAccess

Step 2: Install kubectl on EC2
bash
Copy
Edit
curl -o kubectl https://amazon-eks.s3-us-west-2.amazonaws.com/1.14.6/2019-08-22/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin
cp ./kubectl $HOME/bin/kubectl
export PATH=$HOME/bin:$PATH
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
kubectl version --short --client
Step 3: Install eksctl
bash
Copy
Edit
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/bin
eksctl version
Step 4: Create EKS Cluster (Master Only)
bash
Copy
Edit
eksctl create cluster --name=eksdemo \
                      --region=us-west-1 \
                      --zones=us-west-1b,us-west-1c \
                      --without-nodegroup
Step 5: Associate IAM OIDC Provider
bash
Copy
Edit
eksctl utils associate-iam-oidc-provider \
       --region us-west-1 \
       --cluster eksdemo \
       --approve
Step 6: Add Worker Node Group
bash
Copy
Edit
eksctl create nodegroup --cluster=eksdemo \
                        --region=us-west-1 \
                        --name=eksdemo-ng-public \
                        --node-type=t2.medium \
                        --nodes=2 \
                        --nodes-min=2 \
                        --nodes-max=4 \
                        --node-volume-size=10 \
                        --ssh-access \
                        --ssh-public-key=Praveen-test \
                        --managed \
                        --asg-access \
                        --external-dns-access \
                        --full-ecr-access \
                        --appmesh-access \
                        --alb-ingress-access
Note:
To delete the node group:

bash
Copy
Edit
eksctl delete nodegroup --cluster=eksdemo --region=us-west-1 --name=eksdemo-ng-public
To delete the entire cluster:

bash
Copy
Edit
eksctl delete cluster --name=eksdemo --region=us-west-1
🚀 Application Deployment (Minikube or EKS)
Step 1: Build Maven Projects
bash
Copy
Edit
mvn clean install -DskipTests
Step 2: Build Docker Images
bash
Copy
Edit
docker build -t dnraju7747/shopfront:latest ./shopfront
docker build -t dnraju7747/productcatalogue:latest ./productcatalogue
docker build -t dnraju7747/stockmanager:latest ./stockmanager
Step 3: Push Images to Docker Hub
bash
Copy
Edit
docker push dnraju7747/shopfront:latest
docker push dnraju7747/productcatalogue:latest
docker push dnraju7747/stockmanager:latest
Step 4: Deploy Services to Kubernetes
bash
Copy
Edit
kubectl apply -f kubernetes/shopfront-service.yaml
kubectl apply -f kubernetes/productcatalogue-service.yaml
kubectl apply -f kubernetes/stockmanager-service.yaml
Step 5: Access Services via Minikube
bash
Copy
Edit
minikube service shopfront
minikube service productcatalogue
minikube service stockmanager
✅ Service Build Order
Ensure services are built and deployed in the following order:

shopfront

productcatalogue

stockmanager

🌐 API Endpoints
Service	Endpoint
ProductCatalogue	/products
StockManager	/stocks
