>We need to cover the HPA (Horizontal Pod Autoscaler) topic.<br>
>HPA can be implemented in a kubeadm-based cluster, but it requires extra setup like metrics server, CPU/memory resources, and infrastructure, which may involve cost.<br>
>As students, we usually do not have budget for paid cloud resources.<br>
>So, for learning and practice purposes, we use Minikube.Minikube runs a Kubernetes cluster locally on our system.<br>
>It supports HPA and metrics server without any cost.That’s why Minikube is ideal for hands-on practice and concept clarity.



>So here, we are going to see what Minikube is, how to install and set it up, and how to work with `HPA` using Minikube.




#  Minikube


###  What is Minikube?

Minikube is a lightweight tool that allows you to run a Kubernetes cluster on your local machine.<br>
It creates a single-node Kubernetes environment, where both the control plane and worker node run together.<br>
Minikube is mainly used for learning, testing, and practice purposes, especially by students and beginners.<br>
It supports most Kubernetes features such as Pods, Deployments, Services, ConfigMaps, Secrets, and HPA.


---

###  Why Minikube is Used

- To learn Kubernetes without cloud cost
- To practice real Kubernetes commands locally
- To test applications before deploying to production
- To understand concepts like HPA, scaling, networking, and volumes

---

###  Advantages of Minikube


- Free and no cloud account required
- Easy to install and setup
- Runs on local system (Laptop/PC)
- Supports HPA, Metrics Server, Dashboard
- Best for students and practice labs

---

###  Disadvantages of Minikube

- Not suitable for production environments
- Limited resources (depends on local system)
- Single-node cluster (not multi-node like real production)
- Performance depends on system hardware



---

###  When to Use Minikube

- Kubernetes learning and certification preparation
- Practice for HPA, Deployments, Services
- Demo and testing environments
- When budget or cloud access is not available

---

###  When Not to Use Minikube

- Production workloads
- High-availability setups
- Large-scale enterprise applications

---
---
---



#  Minikube Setup on AWS EC2 (Ubuntu)


This helps explains how to launch an EC2 instance and set up Docker, kubectl, and Minikube for Kubernetes practice.


---

##  Step 1: Launch EC2 Instance


- Name: minikube
- OS: Ubuntu
- Instance Type: t3.small
- Security Group – Open Ports:
  - 22 → SSH (Required)
  - 80 → Optional (For apps)
  - 30000–32767 → NodePort (Optional for Kubernetes services)

Launch the instance and connect using Linux terminal (SSH).



<p align="center">
  <img src="" width="500" alt="Initialize Repository Screenshot">
</p>





---

##  Step 2: Update the System

```Bash
sudo apt update && sudo apt upgrade -y
```







<p align="center">
  <img src="" width="500" alt="Initialize Repository Screenshot">
</p>










---

##  Step 3: Install Docker

```Bash
sudo apt install docker.io -y
```



<p align="center">
  <img src="" width="500" alt="Initialize Repository Screenshot">
</p>




###  Why Docker is Required?

Minikube needs a container runtime to run Kubernetes components.<br>
Here, Docker is used as the driver to run Minikube inside a container.

---

##  Step 4: Start Docker & Check Version


```Bash
sudo systemctl start docker
sudo systemctl enable docker
docker --version
```

---

##  Step 5: Run `docker ps` and Understand the Error

```
docker ps
```

You will get a permission denied error because the ubuntu user is not part of the Docker group.<br>
By default, Docker commands require root (sudo) access.



---

##  Step 6: Add Ubuntu User to Docker Group

```
sudo gpasswd -a ubuntu docker
```

Check if user is added:
```
groups ubuntu
```
Apply changes (important):
```
newgrp docker
```
Now you can run Docker commands without sudo.

Test again:
```
docker ps
```





<p align="center">
  <img src="" width="500" alt="Initialize Repository Screenshot">
</p>





---

##  Step 7: Install kubectl


```Bash
curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

Verify installation:
```Bash
kubectl version --client
```





<p align="center">
  <img src="" width="500" alt="Initialize Repository Screenshot">
</p>





---

##  Step 8: Install Minikube

```Bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube-linux-amd64
sudo mv minikube-linux-amd64 /usr/local/bin/minikube
```

Check version:

```
minikube version
```






<p align="center">
  <img src="" width="500" alt="Initialize Repository Screenshot">
</p>






---

##  Step 9: Start Minikube with Resource Allocation

We allocate CPU and memory to avoid excessive resource usage.
```Bash
minikube start --driver=docker --memory=1800 --cpus=2
```





<p align="center">
  <img src="" width="500" alt="Initialize Repository Screenshot">
</p>





---

##  Step 10: Verify Minikube is Running

```
docker ps
```
You will see a running container named minikube, which confirms that the setup is complete.


---

##  Step 11: Enable Metrics Server

###  What is Metrics Server?

Metrics Server is a Kubernetes component that collects resource usage data like CPU and Memory from Pods and Nodes.<br>
It works similar to CloudWatch, but only inside the Kubernetes cluster.<br>
HPA (Horizontal Pod Autoscaler) depends on Metrics Server to decide when to scale Pods up or down.<br>
Without Metrics Server, HPA will not work.


---

Enable Metrics Server in Minikube
```
minikube addons enable metrics-server
```
⚠️ It may take a few minutes to become fully active.

---

Verify Metrics Server
```Bash
kubectl get pods -n kube-system
```
You should see a pod named metrics-server in Running state.

---

Test Metrics Collection
```
kubectl top nodes
kubectl top pods
```
If CPU and memory values are shown, the Metrics Server is working correctly ✅





<p align="center">
  <img src="" width="500" alt="Initialize Repository Screenshot">
</p>







---
---
---


#  HPA (Horizontal Pod Autoscalling) Concept  



##  HPA Example Using Deployment, Service, and HPA


Now we will create three files:<br>
`deployment.yml`, `service.yml`, and `hpa.yml`.<br>
These three files work together to demonstrate Horizontal Pod Autoscaling.<br>
The Deployment manages pod replicas, the Service exposes the application, and the HPA automatically increases or decreases pods based on CPU usage.


---

###  Check Node Metrics Before Starting

```
kubectl top nodes
```

***Use***:<br>
This command shows CPU and memory usage of nodes.<br>
If this command gives an error, it means Metrics Server is not installed or not working properly.


---

Apply the Kubernetes Files

```Bash
kubectl apply -f deployment.yml
kubectl apply -f service.yml
kubectl apply -f hpa.yml
```






<p align="center">
  <img src="" width="500" alt="Initialize Repository Screenshot">
</p>

<p align="center">
  <img src="" width="500" alt="Initialize Repository Screenshot">
</p>














---

Verify Resources Are Running

```
kubectl get deploy
kubectl get svc
kubectl get hpa
kubectl get pods
```
Make sure all resources are in Running state.


---


###  What is Artificial Load?

Artificial load means intentionally generating traffic to increase CPU usage.<br>
This helps us test whether HPA is scaling pods automatically or not.

---

###  Create Load Generator Pod

Before running the load command, open one more terminal.<br>
This helps to observe real-time scaling behavior.

Run this command:


```Bash
kubectl run load-generator \
--image=busybox \
-- /bin/sh -c "while true; do wget -q -O- http://hpa-service; done"
```

This pod continuously sends requests to the service, increasing CPU load.

---

###  Watch Pod Scaling in Real Time

In the second terminal, run:


```Bash
kubectl get pods -w
```
As load increases, new pods will be created automatically by HPA.

---

###  Stop Load Generator

When you delete the load generator pod:

```
kubectl delete pod load-generator
```

The load stops, and HPA will scale down the pods automatically.


---
---

#  Script's


`deployment.yml`

```yaml
apiVersion: apps/v1                     # API version for Deployment resource
kind: Deployment                      # Resource type is Deployment (manages ReplicaSets & Pods)

metadata:                             # Metadata of the Deployment
  name: hpa-demo                      # Name of the Deployment

spec:                                 # Specification of the Deployment
  replicas: 1                         # Number of Pod replicas to run initially

  selector:                           # Selector to identify Pods managed by this Deployment
    matchLabels:                      # Matching labels
      app: hpa-demo                   # Pods with label app=hpa-demo are managed

  template:                           # Pod template used by the Deployment
    metadata:                         # Metadata for Pods
      labels:                         # Labels applied to Pods
        app: hpa-demo                 # Label used by selector

    spec:                             # Pod specification
      containers:                    # List of containers in the Pod
      - name: hpa-container           # Name of the container
        image: nginx                  # Nginx container image

        ports:                        # Ports exposed by the container
        - containerPort: 80           # Container listens on port 80

        resources:                    # Resource management for the container
          requests:                   # Minimum resources required
            cpu: "100m"               # Requests 100 millicores of CPU

          limits:                     # Maximum resources allowed
            cpu: "200m"               # CPU usage cannot exceed 200 millicores
```


-  `service.yml`

```yaml
apiVersion: v1                          # API version for core Kubernetes resources
kind: Service                           # Resource type is Service (used to expose Pods)

metadata:                               # Metadata information of the Service
  name: hpa-service                     # Name of the Service

spec:                                   # Specification of the Service
  type: NodePort                        # Service type is NodePort (exposes service outside cluster)

  selector:                             # Label selector to choose target Pods
    app: hpa-demo                       # Service routes traffic to Pods with label app=hpa-demo

  ports:                                # Port configuration for the Service
  - port: 80                            # Service port inside the cluster
    targetPort: 80                     # Target port on the container
```

- `hpa.yml`

```yaml
apiVersion: autoscaling/v2              # API version for HorizontalPodAutoscaler (newer stable version)
kind: HorizontalPodAutoscaler           # Resource type is HPA (auto scales Pods)

metadata:                               # Metadata of the HPA
  name: hpa-demo-hpa                    # Name of the HorizontalPodAutoscaler

spec:                                   # Specification of the HPA
  scaleTargetRef:                       # Reference to the resource to be scaled
    apiVersion: apps/v1                 # API version of the target resource
    kind: Deployment                    # Target resource type is Deployment
    name: hpa-demo                      # Name of the Deployment to scale

  minReplicas: 1                        # Minimum number of Pod replicas
  maxReplicas: 5                        # Maximum number of Pod replicas

  metrics:                              # Metrics used for autoscaling decision
  - type: Resource                      # Metric type is Resource (CPU / Memory)
    resource:                           # Resource metric configuration
      name: cpu                         # CPU metric is used
      target:                           # Target threshold for scaling
        type: Utilization               # Scaling based on CPU utilization percentage
        averageUtilization: 50          # Scale if average CPU usage exceeds 50%
```

-  `




---
---
---









