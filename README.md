# Virtualized Cloud Deployment with Docker Compose

## What is this repository?

This repository holds the documentation and source files for the assignment of **Cloud Infrastructure and Virtualisation** module. In this assignment the objective is to follow the official Docker documentation guide in order to implement a containerised application environment.

## What is implemented in this project?

In this project you will see how Docker can be used to build and run containers within a **Linux Vrtual Machine** in Azure cloud and ilustrate the use of cloud infrastrucure and virtualisation.

All the instruction to make a deployment of **multiple containers using Docker Compose** is provided by Docker Documentation, demonstrating how a containerised environment can be used to define, deploy, and manage multitple services together.

## What is included in this repository?

The repository includes:

- Docker configuration files
- Docker Compose configuration for multi-container deployment
- Documentation describing the setup and deployment process
- Instructions to reproduce the environment on a cloud virtual machine

---

## Technologies Used

- Docker
- Microsoft Azure Virtual Machine - Linux (ubuntu 24.04)
- Git and GitHub for version control

---

## Prerequisites

- A **Linux Virtual Machine**
- **Docker Engine installed**
  - Install Docker using apt:
  - https://docs.docker.com/engine/install/ubuntu
- Access to the **terminal / SSH connection**
- **Docker Account** for sharing application
- **Git** (optional, for cloning the repository)

The image below shows that we are now connected to a Virtual Machine in Azure Cloud via SSH.

![Connected to VM](/images/connected-vm-cloud-ssh.jpg)

Tips:

- When we use Docker commands to interact with the Docker Engine we are required to use "sudo" privileges, because the communication occurs through socket. This socket is owned by the "root" user and the "docker group". In order to run Docker commands without using "sudo", we can add the user to the "docker group" and this grants permission to access the Docker socket.

Run:
sudo usermod -aG docker `whoami`

After running the command, log out and log in again for the group membership to take effect, now you can try running Docker commands such as "docker ps" without using "sudo".

---

# 1. Containerize an application

Containerization is a software deployment process that bundles an application's code with all the files and libraries it needs to run on any infrastructure.

In this section we are going to run a todo list manager provided by Docker that runs on Node.js.

## Get The APP

Fork the repository rather than cloning, as this avoids the need to push back from the server and free to make any change. Then clone it into the VM.

https://github.com/docker/getting-started-app/tree/main

![get-the-app](/images/get-the-app.jpg)

## Build the app's image

1. We use Dockerfile to build the image. Dockerfile is blue-print that holds a script of insctructions such as, the base of image (e.g. node:24-alpine), sets the working directory, copy source code into the image, dependencies, commands to start the application and port por the application. Docker uses this script to build a container image.

- Go to the repository, change the browser to "dev mode" to make it possible to modifiy and commit the Dockerfile we will create.

- insede Getting-started-app folder, create a file called "Dockerfile" and paste the text:

![dockerfile](/images/dockerfile.jpg)

- save the change, commit and go to the terminal to pull the file running the command "git pull"

2. Build the image

- Building the images is the process of wrapping the instructions from Dockerfile such as, dependencies and the enviroment into the appliction.

- Navegate to the directory where dockerfile is located:
  - cd getting-started-app
- Build the image running:
  - docker build -t getting-started .
  - -t command tags the image with following name and the dot means it uses the dockerfile in the current direcotry (.).

## Start an app container

Use the command "docker run" to run a container:

- docker run -d -p 8080:3000 getting-started

- The -d flag runs the container in the backgroud, and the -p flag creates a port mapping where 8080 is the port exposed on the VM, while 3000 is the port the applications is listening (defined on Dockerfile)

- Now we can see the application on browser running on port 8080.

![docker-run](/images/docker-run.jpg)
