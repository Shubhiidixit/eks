# eks


AWS EKS
Step-1: Go to IAM console and create a user. 
  

Step-2: Name the user.
 


Step-3: Now click on the user that you just created. Here it is “eks”.
 


Step-4: Click on “Create access key”.
  

Step-5: Choose the option “Other” and click on next. Leave the other details as default.
 

Step-6: Now copy the “Access key” and “Secret Access Key” and save it somewhere as it can’t be retrieved later.

 


Step-7: Now go to “Add Permissions”. 
Note: You can attach the policies while creating the user or after creating the user as well.

 

Step-8: Click on “Attach policies directly” and choose the following permissions and click on next and save them.
AdministratorAccess
AmazonAPIGatewayAdministrator
AmazonEC2ContainerRegistryFullAccess 
 
 

Step-9: Now go to AWS Console and search “AWS Cloudshell”.

Step-10: First step is to configure AWS CLI. Write the command “aws configure”. After that provide the access key and secret access key that you have copied earlier. Write your region and the format. Keep the format in “json” for example.
 

Installing the required tools
Step-11: Now install kubectl using the following command:
curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
	kubectl is the command-line tool used to interact with Kubernetes clusters.
 
Step-12: After the installation is complete, run the following command:
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
	This command is installing the kubectl binary into the /usr/local/bin directory with root ownership and executable permissions, allowing users to run kubectl commands globally on the system.
 

Step-13: In order to install the “eksctl” command line tool, we need to create a script. Create a file called “eksctl.sh” using vim or nano or any other test editor.
vim eksctl.sh
	eksctl is a command-line utility for creating, managing, and interacting with Amazon Elastic Kubernetes Service (EKS) clusters.
 
Step-14: Now in order to insert text, press “I” and then copy the below given commands inside it.

ARCH=amd64 
PLATFORM=$(uname -s)_$ARCH  
curl -sLO "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"  

# (Optional) Verify checksum 
curl -sL "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check  
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz  

sudo mv /tmp/eksctl /usr/local/bin
 


Step-15: Run the below commands:

chmod +x eksctl.sh
sudo sh eksctl.sh

The command chmod +x eksctl.sh is used to grant execute permissions (+x) to the file named eksctl.sh.
The command sudo sh eksctl.sh is attempting to run the script named eksctl.sh that we created.
 

Creating an EKS Cluster
Step-16: Now create a new Amazon EKS cluster using the `eksctl` command:

eksctl create cluster --name my-eks-cluster --region ap-south-1 --nodegroup-name my-nodegroup --node-type t2.small --nodes 3 --nodes-min 1 --nodes-max 5 –managed

	The eksctl create cluster command is creating an Amazon EKS cluster named "my-eks-cluster" in the Asia Pacific (Mumbai) region with a managed node group named "my-nodegroup," using t2.small instances with a desired count of 3 nodes, a minimum of 1 node, and a maximum of 5 nodes.
Note: Make sure to change the region name to your specific region in the command portion that is highlighted.
 

Step-17: Now after the creation is complete in the above step, go to AWS console and search “EKS”. You can see that the cluster is created.
 

Step-18: Also go to EC2 Dashboard and click on instances. We can see that the specified number of node instances are created i.e. 3.
 

Deploying a sample application
Step-19: Now create a “deployment.yaml” file using any editor and copy the following yaml code.
NOTE: MAKE SURE THE INDENTATION OF THE YAML FILE IS SAME AS SHOWN OTHERWISE IT WILL THROW MAPPING ERROR.

apiVersion: apps/v1 
kind: Deployment 
metadata:   
   name: nginx-deployment 
spec:   
   selector:     
      matchLabels:       
        app: nginx   
    replicas: 3   
    template:     
      metadata:       
        labels:         
          app: nginx     
        spec:       
         containers:      
         - name: nginx         
         image: nginx:1.14.2         
         ports:         
           - containerPort: 80
 
 
Step-20: Deploy the sample application to your EKS cluster:
kubectl apply -f deployment.yaml
 
Exposing the Application

Step-21: Now expose the application using a Kubernetes service and get the external IP address of the LoadBalancer:

kubectl expose deployment nginx-deployment --type=LoadBalancer --name=my-service

	The kubectl expose command is creating a new Kubernetes Service named "my-service" and exposing the "nginx-deployment" Deployment to the external network using a LoadBalancer type service.

kubectl get services my-service

	The kubectl get services my-service command is used to retrieve information about the Kubernetes Service named "my-service," displaying details such as the service's IP address, ports, and other relevant information.
 

Step-22: Now go to EC2 dashboards and click on load balancers.
 

Step-23: We can see on the dashboard a “classic” load balancer that is made by default after running the command.
 
Step-24: Copy the DNS name and paste it in browser to check whether the service is accessible or not.
 
We can see that it is accessible on browser.
 

Setting up Horizontal Pod Autoscaler
Step-25: Now create a new file named `hpa.yaml`(just like we made the deployment.yaml) with the following content:

apiVersion: autoscaling/v1 kind: HorizontalPodAutoscaler metadata:   name: nginx-deployment-hpa spec:   scaleTargetRef:     apiVersion: apps/v1     kind: Deployment     name: nginx-deployment   minReplicas: 3   maxReplicas: 10   targetCPUUtilizationPercentage: 50
 
Step-26: 
kubectl apply -f hpa.yaml
 

Step-27: After performing the above commands, you can delete the service and cluster using the following commands:
kubectl delete svc my-service  
eksctl delete cluster --name my-eks-cluster



minikube

Minikube dashboard- 4)	This command opens a web-based interface for managing a local Kubernetes cluster created with Minikube. 

This command retrieves a list of all pods running in all namespaces within the Minikube Kubernetes cluster.  
minikube kubectl -- get po -A 



Deploy Applications 
 
i)Service 
1) Create a sample deployment and expose it on port 8080: 
kubectl create deployment hello-minikube --image=kicbase/echo-server:1.0
kubectl expose deployment hello-minikube --type=NodePort --port=8080 



2) The above command will take a moment but the deployment will soon show up when you run: 
     kubectl get services hello-minikube 
3) Now, the easiest way to access this service is to let the minikube launch a web browser for you: 
     minikube service hello-minikube



4) Alternatively, we can use kubectl to forward the port: 
kubectl port-forward service/hello-minikube 7080:8080 



Now we can access the application at:  
http://localhost:7080/



ii)Load Balancer 
1) To use a LoadBalancer deployment, use the “minikube tunnel” command: 
kubectl create deployment balanced --image=kicbase/echo-server:1.0 
kubectl expose deployment balanced --type=LoadBalancer --port=8080 



2) In another window, start the tunnel to create a routable IP for the ‘balanced’ deployment: 
minikube tunnel



3) To find the routable IP, run this command and examine the EXTERNALIP column:
 kubectl get services balanced



Your deployment is now available at <EXTERNAL-IP>:8080 
Here in my case the External IP will be replaced by : 127.0.0.1 



Manage Your Cluster 
1)	To pause Kubernetes without impacting deployed applications, run: 
minikube pause 
2)	To unpause the paused instance, run: 
minikube unpause 
3)	To change the default memory limit run: 
minikube config set memory 9001



4)	To browse the catalog of easily installed Kubernetes services,run: 
minikube addons list


5)	To create a second cluser running an older Kubernetes release, run: 
minikube start -p aged --kubernetes-version=v1.16.1
6)	To halt the cluser, run: 
minikube stop
7) To delete all the minikube clusters, run: 
minikube delete –all



