# Virtualized Cloud Deployment with Docker Compose

## What is this repository?

This repository holds the documentation and source files for the assignment of **Cloud Infrastructure and Virtualisation** module. In this assignment the objective is to follow the official Docker documentation guide in order to implement a containerised application environment.

## What is implemented in this project?

In this project you will see how Docker can be used to build and run containers within a **Linux Virtual Machine** in Azure cloud and illustrate the use of cloud infrastrucure and virtualisation.

All the instruction to make a deployment of **multiple containers using Docker Compose** is provided by Docker Documentation, demonstrating how a containerised environment can be used to define, deploy, and manage multiple services together.

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

The image below shows that we are now connected to a Virtual Machine in Azure Cloud via SSH. SSH can be generated on terminal using the command "ssh-keygen", which generate private and public keys, the public key is the one we register on the VM and the private key is the one we use to access the VM.

![Connected-to-VM](/images/connected-vm-cloud-ssh.jpg)

Tips:

- When we use Docker commands to interact with the Docker Engine we are required to use "sudo" privileges, because the communication occurs through socket. This socket is owned by the "root" user and the "docker group". In order to run Docker commands without using "sudo", we can add the user to the "docker group" and this grants permission to access the Docker socket.

Run: sudo usermod -aG docker \`whoami\`

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

1. We use Dockerfile to build the image. Dockerfile is blue-print that holds a script of instructions such as, the base of image (e.g. node:24-alpine), sets the working directory, copy source code into the image, dependencies, commands to start the application and port por the application. Docker uses this script to build a container image.

- Go to the repository, change the browser to "dev mode" to make it possible to modifiy and commit the Dockerfile we will create.

- insede Getting-started-app folder, create a file called "Dockerfile" and paste the text:

![dockerfile](/images/dockerfile.jpg)

- save the change, commit and go to the terminal to pull the file running the command "git pull"

2. Build the image

- Building the images is the process of wrapping the instructions from Dockerfile such as, dependencies and the enviroment into the appliction.

- Navigate to the directory where dockerfile is located:
  - cd getting-started-app
- Build the image running:
  - docker build -t getting-started .
  - -t command tags the image with following name and the dot means it uses the dockerfile in the current directory (.).

## Start an app container

Use the command "docker run" to run a container:

- docker run -d -p 8080:3000 getting-started

- The -d flag runs the container in the backgroud, and the -p flag creates a port mapping where 8080 is the port exposed on the VM, while 3000 is the port the applications is listening (defined on Dockerfile)

- Now we can see the application on browser running on port 8080.

![docker-run](/images/docker-run.jpg)

---

# 2. Update the application

We will change the text displayed for when there is no item in the list, the currente text is "No items yet! Add one above!", the new text will be "You have no todo items yet! Add one above!".

- For this we are going to modify src/static/js/app.js.

  Current:
  `<p className="text-center"> No items yet! Add one above!"</p>`

  Update to:
  `<p className="text-center"> You have no todo items yet! Add one above!</p>`

- Commit and push the update.
- Pull the update on the VM.
- Build the new image.
  - docker build -t getting-started .

![text-update](/images/update-content-push-pull-and-build-new-image.jpg)

## Remove a container and Start the updated app container

As the port 8080 is already allocated by the old container we have to stop and remove the old container first and then we can run the new image with the updated text.

- Get the Id of the container:
  - docker ps

- Stop the container:
  - docker stop <the-container-id>

- Remove the container:
  - docker rm <the-container-id>

- Start the updated app container
  - docker run -dp 8080:3000 getting-started

![stop-remove-container](/images/stop-remove-container.jpg)
![run-new-image-text-update](/images/run-new-image-text-update.jpg)

# 3. Share the application

We can share a Docker image by using docker registry. Docker Hub is the defualt registry, where we can create repositories and share the application. Lets share the image created on step 2.

## Create a repository

After singing up in Docker Hub, create a repository named "getting-started" and make sure visibility is "Public".

To push an image in to Docker Hub we need to tag the image with Docker ID + repository name using the command "docker tag". Log in Docker Hub to get the docker ID.
run:
`docker tag getting-started:latest <docker-id>/getting-started:<newtag>`

Now that we have the new tag, we can push the image in to Ducker hub. Make sure to log in (access token ca be generate on Docker Hub account settings).

![dockerhub-new-tag-for-sharing-image](/images/new-tag-for-sharing-image.jpg)

The image is now on Docker Hub, to test it we will first remove all old images, then use the shared image by pulling it from Docker Hub to run a new container.

![removed-img-pull-img-from-docker-hub](/images/remove-img-pull-img-from-docker-hub.jpg)

# 4. Persist the DB

Each Docker container has its own filesystem composed of multiple layers, it also contains the writable "scratch space" to create, update, and remove files, which allow them to have isolated filesystem for each container even though they are created from the same image. However, those files are lost once the container is removed and this is why the "todo list" in "getting-started-app" is empty everytime a new container is created, unless "Docker volume" is used to persist the data.

The next image shows exactly how each container has its own filesystem. The command `docker run --rm alpine touch greeting.txt` starts a container with the alpine image and removes it (--rm) once it stops running, it also adds the text file called "greeting.txt". After that another container starts from the same image "alpine" with the "stat" command that looks for the "greeting.txt" but not file is found as they have its own filesystem.

![individual-filesystem-for-each-container](/images/individual-filesystem-for-each-container.jpg)

## Container volumes

Docker provides two main mechanisms to persist the storage, we will see first how **volume mounts** work and how to use them.

**Docker volumes** allow the data to be stored in directory outside the container on the host machine that is linked to the container's filesystem, so when a container is removed the data persists. This makes it possible to mount the same volume across container restarts or new containers.

## Persist the todo data

We will use **volume mount** to persist the todo list data for getting-started-app. The application stores the data at /etc/todos/todo.db inside container by default, while the volume mount in the host machine is managed by Docker. We are going to see how **bind mounts** work later on.

## Create a volume and start the container

- As **volume mount** is managed by Docker, all we have to do to create the volume is run "docker volume create" command:
  `docker volume create todo-db`

- After creating the volume, stop and remove the container as this one is not  
  running with the persistent volume. `docker rm -f <container-id>`

- To start a new container we will use the command "docker run" as we did before and add the --mount option which requires three pieces of information:
  - type=volume (volume type)
  - src=todo-db (volume name)
  - target=/etc/todos (container's volume path)

`docker run -dp 8080:3000 --mount type=volume,src=todo-db,target=/etc/todos getting-started`
(remember to change the value for the "port mapping" and "name of image" accordingly)

If you are using Git Bash you may use a different syntax for this command:

`docker run -dp 8080:3000 --mount type=volume,src=todo-db,target=//etc/todos getting-started`

You can check more Git Bash commands at:
https://docs.docker.com/desktop/troubleshoot-and-support/troubleshoot/topics/#docker-commands-failing-in-git-bash

- After running the container with the volume, open the application in your browser to add items to the todo list, after that remove and delete the container and start a new container with the volume we created to see the persistant data.

- Check where Docker stores the volume running:
  `docker volume inspect todo-db`

The image below shows the creation of a volume, running a container with volume and checking where volume is stored.

![persisting-data-docker-volume](/images/persisting-data-docker-volume.jpg)

# 5. Use bind mounts

The **bind mount** is another way of sharing data from the filesystem on the host machine into the container. The difference is that we can specify the path of the file or directory to be mounted in to the container when using bind mounts, while volumes are managed by Docker in the filesystem's Docker area.

A bind mount is great solution for the case where sharing data of a specific file/directory already defined in the host machine.

We will see how we can use bind mounts and **nodemon** tool to watch for file changes.

![mounts-bind-diagram](/images/mounts-bind-diagram.png)

(Check Docker documentaion for **tmpfs mounts** - temporary filesystem in memory: https://docs.docker.com/engine/storage/tmpfs/)

As volumes (named volumes) are manage by Docker we only need to provide the name of the volume when mouting, while running with bind volumes we provide the path for this data, see the following commands comparison:

- Named Volume: `type=volume,src=my-volume,target=/usr/local/data`
- Bind mount: `type=bind,src=/path/to/data,target=/usr/local/data`

## Trying out bind mounts

We will understand now how bind mounts work with an experiment and then use bind mounts for the getting-started-app.

Note: Since this application is running on a Linux VM in Azure Cloud, Docker can access the filesystem in the host machine without the need of any configuration. Docker Desktop with WSL2 makes it possible for Docker to access the filesystem on Windows host machine, while on MacOS Docker Desktop uses a lightweight Linux VM where filesystem sharing is handled automatically, therefore if Docker is in Hyper-v mode, which is an isolated virtual machine, configuration is needed in the "File Sharing".

The experiment is to run an Ubuntu container just to use bind mount and see the file created inside getting-started-app (host machine) and inside the container.

- Go into getting-started-app `cd getting-started-app`
- Run the command to start bash in an Ubuntu container with a bind mount:
  - `docker run -it --mount type=bind,src=.,target=/src ubuntu bash`

The "--mount type=bind" means the craeting of a bind mount, "src" means the source (origem) (getting-started-app), while "target" is the destination inside the container.

The "-it" combines two flags:
"-i" = interactive (container accepts input), while "-t" = tty (allocates a pseudo, allowing the interaction). This is why you will see: root@ac1237fad8db:/#.

- run: `pwd` to confirm you are in the root directory of the container "/", and then `ls`, after this we move to src directory `cd src` which is the mounted directory when starting the container - linked to the host machine.

- We will create a text file `touch myfile.txt` and see it will be crated in hot the host machine too.

(See on the image below that the myfile.txt was deleted from the host machine and it was listed anymore in the container)
![trying-bind-mounts-ubunto](/images/trying-bind-mounts-ubuntu.jpg)

## Development Containers

Bind mounts are useful for development as the development machine does not need to have all the build tools and environments installed. The container can pull dependencies and tools with one command line.

- We will run the "getting-started-app" in development container with bind mounts, where we will:
  - Mount source code into the container
  - Install dependencies
  - Run the app with nodemon

- List containers `docker ps` and make sure there is no "getting-started" containers running, if so remove them.

- Inside directory "getting-started-app" run:

docker run -dp 8080:3000 \
 -w /app --mount type=bind,src=.,target=/app \
 node:24-alpine \
 sh -c "npm install && npm run dev"

Breakdown of the command:

- `-dp 8080:3000 \` - run the container in the background and map port 8080 on the host to port 3000 in the container.
- `-w /app` - sets the "working directory" inside the container.
- `--mount type=bind,src=.,target=/app` - bind mount the current host directory into "/app" in the container.
- `node:24-alpine` - the base image to use .
- `sh -c "npm install && npm run dev"` - run a shell to install dependencies and start the dev server (nodemon).

- `docker logs <container-id>` - to see the logs of

![development-containers](/images/development-containers.jpg)

## Developing the Application with the Development Container

We will update the button text and see the change which is watched by nodemon and refreshes the UI.

- In the src/static/js/app.js file (line 109), we'll change the button label "Add Item" to "Add".

Two ways to test it:

- Edit the file on Github (dev mode), commit and push an then "git pull" on the VM via terminal.
- Edit the file on the VM:
- `cd getting-started-app`
- `nano src/static/js/app.js`

After saving the change, refresh the broswer and confirm the change of button label.

Every time a change is made, the **nodemon** watches the change and reflects it into the container. After stopping the container, we can build a new image with the changes.

![new-image-docker-dev-updated](/images/new-image-docker-dev-updated.jpg)
