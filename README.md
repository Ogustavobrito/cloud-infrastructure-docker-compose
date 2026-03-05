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
