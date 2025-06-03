# Container Services


So here is a solution to a hypothetical scenario where perhaps the city's premier medical research center wants to containerize and deploy their application in the cloud.

<p align="center">
  <img src="./img/1.png" alt="" style="display: block; margin: auto;" />
</p>

## Table of Contents

- [Requirements](#requirements)
- [Steps](#Steps)
- [Conclusion](#conclusion)
- [Contributors](#contributors)

## Requirements

To complete this solution, I utilized the following services

- Amazon ECR
- Amazon ECS
- Amazon Fargate
- Amazon Cloud9

## Steps

### Step 1. This solution uses Amazon Elastic Container Registry (Amazon ECR), Amazon Elastic COntainer Service (Amazon ECS), and AWS Fargate to host containerized applications without the need to provision and manage servers :

- Create an Amazon SQS queue
- Create an Amazon SNS topic
- Subscribe the Amazon SQS queue to the Amazon SNS topic
- Create an additional SQS queue and subscribe it to your existing SNS topic

### Step 2. I used the AWS EC2 terminal to create a Docker image of the clients application

### Step 3. To create a Docker image, I had to create a file called `Dockerfile`. A `Dockerfile` is a manifest that describes the base image to use for Docker image and what we want to install and run on it.

### Step 4. I used the "docker build" command to build Docker images from a `Dockerfile` and a context. A build's context is teh set of files for the containerized application located in a specified path or URL.

```bash
cd ~/environment/first_app
docker build  -t ${repo_name} .
```

### Step 5. Amazon ECR is an AWS managed container image registry service that is secure, scalable, and reliable. With it I can store, share, and deploy container software anywhere.

### Step 6. After the Docker image was created, I pushed the container to an Amazon ECR repository.

### Step 7. A task definition is required to run Docker containers in Amazon ECS. A task definition specifies parameters (such as CPU and memory) to use with each task, launch type, networking mode, logging configuration, run command, data volume, and IAM role that the task uses.

- To get the value of the Region, at the command prompt, run:

```bash
region=${region:-us-east-1}
```

- To create a repository name, run the following commands one at a time:

```bash
repo_name="my_app"

account=$(aws sts get-caller-identity --query Account --output text)

fullname="${account}.dkr.ecr.${region}.amazonaws.com/${repo_name}:latest"
```

- To create an Amazon ECR repository, run:

```bash
aws ecr create-repository --repository-name "${repo_name}"
```

- To retrieve an authentication token, run:

```bash
aws ecr get-login-password --region ${region}|docker login --username AWS --password-stdin ${fullname}
```

- To create, build, and tag Docker images locally, run the following commands one at a time:



<p align="center">
  <img src="./img/2.png" alt="" style="display: block; margin: auto;" />
</p>

### Step 2: Build second app

The next step is to psuh second app on ECR. Follow these steps:

- To compile and push the image of the second_app to Amazon ECR, run the following commands one at a time.

```bash
cd ~/environment/install_scripts/
./push_second_app.sh
```

- The push_second_app.sh shell script creates the my_second_app image in Amazon ECR.

<p align="center">
  <img src="./img/3.png" alt="" style="display: block; margin: auto;" />
</p>

### Step 3: Deploy first app

The next step is to deploy the application with ECS and Fargate using the image from Amazon ECR. To deploy the application, follow these steps:

1. Go to the Amazon ECS console and click on "Create cluster". Choose the "Fargate" launch type, and follow the prompts to create a new cluster.
2. Click on "Create task definition". Choose "Fargate" launch type, and select "ecsTaskExecutionRole" for the task role. Under "Container definitions", click on "Add container". Give the container a name, and under "Image", enter the ECR repository URI for the Docker image. Click on "Add".
3. Follow the prompts to create the task definition.
4. Click on "Create service". Choose the task definition created in the previous step, and follow the prompts to create a new service.

<p align="center">
  <img src="./img/4.png" alt="" style="display: block; margin: auto;" />
</p>

### Step 4: Deploy second app

The final step is to deploy the second application called my_second_app using Fargate by using the image from Amazon ECR and validate access to the second application. To deploy the second application, follow these steps:

1. Create a new task definition for the second application using the same steps as in Step 3, but with the appropriate ECR repository URI for the Docker image.
2. Click on "Create service". Choose the task definition created in the previous step, and follow the prompts to create a new service

<p align="center">
  <img src="./img/5.png" alt="" style="display: block; margin: auto;" />
</p>

<p align="center">
  <img src="./img/6.png" alt="" style="display: block; margin: auto;" />
</p>







## Conclusion

The Decoupling Application quest of AWS is a valuable resource for developers looking to build highly scalable and decoupled applications. By learning how to create Amazon SQS queues, SNS topics, and subscriptions, developers can build resilient and fault-tolerant systems that can scale to meet increasing demand. The skills learned in this quest can be applied to a wide variety of use cases, from simple message queuing systems to complex distributed applications. With the knowledge gained from this quest, developers can create efficient and reliable applications that can adapt to changing business needs and user demands.

<p align="center">
  <img src="./img/7.png" alt="" style="display: block; margin: auto;" />
</p>

[![GitHub Followers](https://img.shields.io/github/followers/DanieleBocchino?style=social)](https://github.com/ldco2016)  
