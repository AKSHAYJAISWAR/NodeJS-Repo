Created EC2 instance (T2 Medium) on AWS and Installed Minikube on it for Kubernetes cluster setup.
Installed AWScli and configured it using below code.
$aws configure set aws_access_key_id YOUR_ACCESS_KEY
$aws configure set aws_secret_access_key YOUR_SECRET_KEY
$aws configure set default.region YOUR_DEFAULT_REGION
$aws configure set default.output json

Created a custom Image of NodeJS app using Dockerfile (listed in repository)
Before Bulding Image from Docker file extract nodejsfile (2).zip in same dir as Docker file.
Build using below command 
$docker build -t nodejs-test . -f Dockerfile

Created ECR on AWS account and pushed Image nodejs-test to this private Repo using Below command.
$aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 52897365177.dkr.ecr.us-east-1.amazonaws.com/nodejs-test
$docker tag nodejs-test:latest 528973675177.dkr.ecr.us-east-1.amazonaws.com/nodejs-test:latest
$docker push 528973675177.dkr.ecr.us-east-1.amazonaws.com/nodejs-test:latest

We can pull the image locally using 
$docker pull 528973675177.dkr.ecr.us-east-1.amazonaws.com/nodejs-test

Now we will create Kubernetes Deployment object but before pulling image in Kubernetes Deployment object we need to follow certain steps as the image is in private repository (ECR).
Follow the below steps
$ pip install awscli
$ export AWS_ACCESS_KEY_ID='AKIAI44QH8DHBEXAMPLE'
$ export AWS_SECRET_ACCESS_KEY='hjhjjhjyjhvftrr4546555465/gtySDESSHRd543545'
$ aws ecr get-login --region eu-west-1
$docker login -u AWS -p SomeVeryLongToken -e none https://4338991606.dkr.ecr.eu-west-1.amazonaw
The above command will give DOCKER_REGISTRY_SERVER, DOCKER_USER, DOCKER_PASSWORD (very long string) put all in below mentioned command.

$ kubectl create secret docker-registry my-docker-credentials \
    --docker-server=DOCKER_REGISTRY_SERVER \
    --docker-username=DOCKER_USER \
    --docker-password=DOCKER_PASSWORD \
    --docker-email=DOCKER_EMAIL
    
The above command will generate secret named as my-docker-credentials which will be used in nodejs.yml file for authentication while pulling image directly from ECR .
Now deploy pods using nodejs.yml file also create Service object to access the NodeJS app outside the network and create priority.yml to set high priority for pods with below command
$ kubectl apply -f nodejs.yml 
$ kubectl apply -f nodejsservice.yml
$ kubectl apply -f priority.yml

Now create a Horizontal POD autoscaling to ensure that 7 min pods are running in any situation.
$ kubectl apply -f nodejshpa.yml

We can generate a load with below mentioned command and check whether min 7 pod remains running or not.
$kubectl run --generator=run-pod/v1 -it --rm load-generator --image=busybox /bin/sh
Hit enter for command prompt
while true; do wget -q -O- http://EC2publicIP.local; done

Now we are able to access the webpage of NodeJS app using public IP of instance and port number provided by Kubernetes service object in my case port is 31175.
Now configure EC2 Classic load balancer via AWS and forward port 31175 to 3000.
We can now access Dns:3000 provided by EC2 CLB.
