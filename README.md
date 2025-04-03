# Deploying an E-commerce Application on AKS

In this tutorial, we will walk through deploying a fully functional e-commerce application on an Azure Kubernetes Service (AKS) cluster. This project is perfect for anyone looking to understand the fundamentals of microservices, containerization, and Kubernetes. Even if you’re a beginner, you’ll learn how to handle real-world deployments using various tools and technologies such as Docker, Kubernetes, Helm, and AKS.

## Project Overview

This e-commerce application is made up of eight microservices:

User Microservice — Handles user registration and login.
    
Catalog Microservice — Displays products and their details.
    
Cart Microservice — Manages items added to the shopping cart.
    
Shipping Microservice — Calculates shipping based on location.
    
Payment Microservice — Integrates with payment gateways.
    
Dispatch Microservice — Handles product dispatch and delivery.
    
Web Microservice — Provides the front-end interface.
    
Ratings Microservice — Allows users to rate products.

These microservices are written in various programming languages — Node.js, Python, PHP, and Java — giving you exposure to working with different technologies. The application also uses MySQL and MongoDB databases, and Redis for in-memory data storage.

We’ll cover the following steps:

### Architecture Overview

### Dockerizing Microservices
   
### Setting Up the AKS Cluster
    
### Deploying Microservices to AKS
    
### Implementing Ingress for External Access

# 1. Architecture Overview

The e-commerce application follows a microservices architecture where each component, like the User Service, Catalog Service, and Cart Service, operates independently. These microservices communicate through APIs, and each has its own database.

## Key Components:

 Microservices: Independently deployed services for different functions like user management, product catalog, etc.

 Databases:

 MySQL for user registration and structured data.
    
 MongoDB for unstructured data like product images.
    
 Redis: Used as an in-memory data store for handling dynamic data like product ratings.
    
 Ingress Controller: Exposes the application to the outside world via path-based routing.



 # 2. Dockerizing Microservices

 Here you can build the docker images and store them in your docker repository or you can use the existing ones which are already available on docker. I am using the existing ones for this project.

 Run below command to check for application is running locally.

 ```shell
docker-compose up
```

![image](https://github.com/user-attachments/assets/8c2370ce-a54e-4cc2-b6e4-70d821d1889c)


# 3. Setting Up the AKS Cluster

Azure Kubernetes Service (AKS) is a fully managed Kubernetes service. To deploy the application on AKS, we first need to create an AKS cluster. Select the resource group and name for Kubernetes cluster then click next.

![image](https://github.com/user-attachments/assets/c0c48eb1-b369-445b-af22-e4b180d0bca7)


Then select agentpool under node pools section.

![image](https://github.com/user-attachments/assets/8c22ef25-c540-4ba6-987d-eb4b04eeb18a)


Next we need to select Azure CNI Node subnet, because this will enable the loadbalancer for the cluster, so that the website can be accessed in browser.

![image](https://github.com/user-attachments/assets/e0dc33e1-dbaa-46f2-a85c-d37f6db9784c)

Finally click on review and create option for the cluster to be created.

The Stan Robot Shop is a microservices-based application designed to give hands-on experience with different databases, programming languages, and deployment strategies.

## Key Technologies Used
### Databases:

 MySQL: Stores structured data such as product images and metadata.

 MongoDB: Handles user authentication and unstructured data.

Redis (In-Memory Datastore): Stores dynamic product ratings for fast retrieval.

### Microservices:

Written in different languages for educational purposes.

Examples:

Cart Service → Node.js

Payments Service → Python

Dispatch Service → PHP

Shipping Service → Java

### Why Use Redis Instead of a Database for Ratings?

Ratings are frequently updated and need fast retrieval.

Storing ratings in Redis (in-memory cache) ensures lower latency.

Redis is deployed as a StatefulSet in Kubernetes with persistent storage to prevent data loss during restarts.

### Why Use Microservices Instead of Monolithic Architecture?

Each component (Cart, Payments, Shipping, etc.) is independent.

If one service fails (e.g., Payments), the rest of the system remains functional.

Teams can develop and scale each service separately.


## Connect to the Cluster:

```shell
az login
```

Once the Kubernetes cluster is created, run below command to check it

```shell
az aks get-credentials --resource-group <resource group name> --name <cluster name>
```

![image](https://github.com/user-attachments/assets/a328a073-2b0a-4e47-8771-a338d3ae239a)

Once successfully connected, it shows ouput as above.

We can also verfify using below command.

![image](https://github.com/user-attachments/assets/8a567ede-af07-4bee-a92b-2dedb28551e2)

Next go to folder AKS/helm and create namespace robot-shop.

![image](https://github.com/user-attachments/assets/5425057a-6dc9-4f4e-aa71-5a2b20d3699a)


# 4. Helm Install

Next install helm using below

![image](https://github.com/user-attachments/assets/c869033f-0dc2-4846-824d-a855039fe7b4)


## What is Helm and Why Use It?

Helm is a package manager for Kubernetes that simplifies the deployment and management of applications. Instead of manually writing multiple Kubernetes YAML files, Helm helps manage these files dynamically using templates.

Three Key Components of a Helm Chart

### Chart.yaml

- Metadata about the Helm chart, including its name, version, and dependencies.

- Helps in maintaining multiple versions of the chart.

### values.yaml

- Stores configuration parameters that can be customized for different environments (Dev, Staging, Production).

- Any dynamic values such as image repositories, environment variables, CPU limits, etc., are stored here.

### templates/ Folder

- Contains all Kubernetes YAML manifests (Deployments, Services, ConfigMaps, etc.).

- Uses Jinja templating to replace dynamic values with those defined in values.yaml.

- Helps avoid maintaining multiple YAML files for different environments.


# Why Use Helm Instead of Direct Kubernetes YAML Files?

### If we use direct Kubernetes YAML manifests, we would have:

8 microservices → 16 YAML files (Deployment + Service for each microservice).

2 databases → 4 YAML files (Deployment + Service for MySQL & Redis).

Total: 24 YAML files per environment.

### If an organization has multiple environments (Dev, Staging, Production), this number multiplies:

5 environments → 24 × 5 = 120 YAML files  (hard to maintain).

## Solution: 
 
 Using Helm, we can store all YAML manifests in the templates/ folder and use values.yaml to dynamically update values for different environments.


Next we need to define the storage class for redis statefulset.yaml

![image](https://github.com/user-attachments/assets/8c8435f2-9c82-4a11-b0df-c89995f84890)

As shown above, we need to specify the storage value and type of storage. In Azure we have different types as shown below.

![image](https://github.com/user-attachments/assets/ec503e13-6e0a-48ab-bc2d-24b236bab0ea)


Next, we can view the pods in robot shop namespace.
![image](https://github.com/user-attachments/assets/d2658350-69b1-48c7-b309-5f05757f7857)



Here we can view the PVS (persistent volume claim) used by the redis can be seen using below command.
```shell
Kubectl describe pod <redis-pod> -n <namespace>
```

![image](https://github.com/user-attachments/assets/c5f7742f-3f8d-4ba3-8231-bac515e815ab)

We can also view the PVS using below command. Here we can see the storage class type and capacity of storage created. i.e default and 1GB.

![image](https://github.com/user-attachments/assets/9ad171c6-0c12-4761-b686-58f386917334)


### When to choose Azure disks or Files?

If storage is accessed by single pod, we can choose azure disk. If we have multiple pods to access the storage, then we can choose azure files.



# 5. Implementing Ingress for External Access

Finally, we need to enable the load balancer so that we can access the website externally.
![image](https://github.com/user-attachments/assets/031d1a46-99db-43ce-af58-66445085922b)


Next, apply the ingress.yaml file using below command.
```shell
Kubectl apply -f ingress.yaml
```

We can verify the ingress by using below command.
```shell
Kubectl get pods -n kube-system
```

![image](https://github.com/user-attachments/assets/26e0a12d-5844-4a21-8803-e196f230a146)


Next we need to get address for the ingress controller.
![image](https://github.com/user-attachments/assets/13f8e655-c902-4c2e-b4fb-2fe7877400e7)



We can access the website finally using load balancer as shown below.
![image](https://github.com/user-attachments/assets/95fe34ed-ab14-46a8-8e96-85abcce642cd)



This guide provides a structured approach to deploying a real-world e-commerce application using microservices on Azure Kubernetes Service. The project offers hands-on experience with Docker, Kubernetes, and Helm while exploring the benefits of microservices architecture. Whether you’re a beginner or an aspiring DevOps engineer, this project will give you the foundational skills you need to work with containerized applications in the cloud.


## Conclusion:

Deploying an e-commerce application with multiple microservices on AKS provides beginners with exposure to critical concepts like containerization, microservice architecture, automation, security, and scalability. This project is a foundational step for anyone looking to build expertise in cloud-native applications, Kubernetes, and Azure DevOps, setting them up for success in modern cloud engineering roles.













 

 

    

    
