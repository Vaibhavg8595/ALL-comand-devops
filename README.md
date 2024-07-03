git add 
git commit -m ""
git push 
git config --global user.name ""
git config --global user.email ""

git checkout -b dev

## DOCKER

sudo docker pull httpd
sudo docker run httpd
sudo docker run -d httpd
sudo docker run -d -P nginx
sudo docker run -d -p 80:80 nginx
sudo docker exec -it (container id) curl localhost
sudo docker exec -it (container id) /bin/bash
sudo docker kill (container id) 
docker stop (container id)
docker rmi -f (image name)
sudo docker run -d -P --name dollikichai nginx
sudo docker logs (container id)
sudo docker ps -q
sudo docker kill $(sudo docker ps -q)
sudo docker network ls
docker inspect  (container id)
docker network create (network name)
docker volume create (volume name)
docker volume ls 
docker volume create dev
docker run -d -P -v dev:/usr/share/nginx/html nginx
docker system prune -a -f


##Dockerfile
FROM
COPY
ADD
RUN
CMD
ENTRYPOINT
EXPOSE

##
FROM amazonlinux 
RUN yum update && yum install nginx -y
COPY index.html /usr/share/nginx/html/.
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

## tomcat
FROM amazonlinux 
RUN yum update && yum install -y  java-11-amazon-corretto.x86_64 && yum install -y unzip && yum install -y wget && yum install -y maven 
RUN wget https://dlcdn.apache.org/tomcat/tomcat-8/v8.5.100/bin/apache-tomcat-8.5.100.zip
RUN unzip apache-tomcat-8.5.100.zip 
RUN mv apache-tomcat-8.5.100 /mnt/tomcat
RUN chmod 770 /mnt/tomcat/bin/catalina.sh
COPY student-ui /
RUN mvn clean package
RUN cp /target/*.war /mnt/tomcat/webapps/student.war
EXPOSE 8080
CMD ["/mnt/tomcat/bin/catalina.sh" , "run"]

##https://github.com/Pritam-Khergade/student-ui/tree/master

## multistage 
FROM maven:3.9-sapmachine-11 as builder
COPY . /
RUN mvn clean package

FROM tomcat:jre8-alpine
COPY --from=builder /target/*.war webapps/student.war\


## Kubernetes cluster creation 
admin ec2 role
kubectl 
awscli
eksctl
#https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html
##install kubectl 
#refer https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.30.0/2024-05-12/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
kubectl version --client
#install eksctl https://eksctl.io/installation

# for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

# (Optional) Verify checksum
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz

sudo mv /tmp/eksctl /usr/local/bin

# eksctl 
eksctl create cluster --name bramhos --node-type=t2.medium --nodes=2 --region=us-east-1
#eksctl create cluster --name my-cluster --region region-code --version 1.29 --vpc-private-subnets subnet-ExampleID1,subnet-ExampleID2 --without-nodegroup

## kubectl 

kubectl get nodes
kubectl get nodes -o wide

##pod 
kubectl get pods 
kubectl describe po nginx 
## create pods
kubect apply -f pod.yaml


## container type
 sidecar container -> 
 init container -> 


## 
 pod -> 
 rs-> 
    prefix -> rs\
deploy -> 
    rs-> prefix-> 
        pod -> 
## services
ClusterIP  -> default -> internal 
NodePort -> expose -> service -> internet
LoadBalancer -> only available on cloud 

## 
kubectl create configmap nginxconf --from-file=index.html --dry-run=client -o yaml > cm.yaml
  kubectl create secret generic nginx-secret --from-file=index.html --dry-run=client -o yaml > secret.yaml

  kubectl port-forward pod/dapi-test-pod :80

  echo "aGVsbG8gCg==" | base64 --decode

#ns
  kubectl create ns dev
  kubectl run nginx --image=nginx -n dev
    kubectl get po -n dev
kubectl get po -A 


resources: {}
    limits: 
        cpu:
        memory:
    requests:
        cpu:
        memory:

## volumes
 host PATH
 CSI drivers  -> dynamic volumes

#pv

pv 
  pvc
  volume pod 
 
dynamic volumes
    pvc 
    volume pod 
  

##
 
1. eksctl create iamserviceaccount \
    --name ebs-csi-controller-sa \
    --namespace kube-system \
    --cluster thor \
    --role-name AmazonEKS_EBS_CSI_DriverRole \
    --role-only \
    --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
    --approve --region=us-east-2

# https://docs.aws.amazon.com/eks/latest/userguide/csi-iam-role.html

2. cluster_name=thor
3. oidc_id=$(aws eks describe-cluster --name $cluster_name  --region=us-east-2 --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
4. echo $oidc_id
5. aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4
6. eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve --region=us-east-2
7. again 1st step


8. eksctl create addon --name aws-ebs-csi-driver --cluster thor --service-account-role-arn arn:aws:iam::533266958719:role/AmazonEKS_EBS_CSI_DriverRole --force

9. eksctl get addon --name aws-ebs-csi-driver --cluster thor --region=us-east-2
10. kubectl apply -f volume
