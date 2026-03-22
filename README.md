# Virtualized Cloud Deployment with Docker Compose

## Overview

This repository holds the documentation for the assignment of ** B8IS003 Cloud Infrastructure and Virtualisation (B8IS003_2526_TMD2)** module - Dublin Business School.

In this assignment the objective is to follow the official Docker documentation guide to demonstrate the implementation of a containerised application using Docker in a cloud-based virtualised environment.

- Containerisation using Docker
- Deploy application in a virtual machina
- Share the application via Docker Hub
- Demonstrate data persistence using Docker Volumes
- Container Network
- Multi-container applications using Docker Compose

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
- **Docker Hub Account** to share images
- **Git**

The image below shows that we are now connected to a Virtual Machine in Azure Cloud via SSH. SSH can be generated on terminal using the command "ssh-keygen", which generate private and public keys, the public key is the one we register on the VM and the private key is the one we use to access the VM.

![Connected-to-VM](/images/connected-vm-cloud-ssh.jpg)

Tips:

- When we use Docker commands to interact with the Docker Engine we are required to use "sudo" privileges, because the communication occurs through socket. This socket is owned by the "root" user and the "docker group". In order to run Docker commands without using "sudo", we can add the user to the "docker group" and this grants permission to access the Docker socket.

Run: sudo usermod -aG docker \`whoami\`

After running the command, log out and log in again for the group membership to take effect, now you can try running Docker commands such as "docker ps" without using "sudo".

---

# 1. Containerize an application

Containerization is a software deployment process that bundles an application's code with all the files and libraries it needs to run on any infrastructure.

In this section we are going to run a Todo List manager provided by Docker that runs on Node.js.

## Get The APP

Fork the repository rather than cloning, as this avoids the need to push back from the server and free to make any change. Then clone it into the VM.

https://github.com/docker/getting-started-app/tree/main

![get-the-app](/images/get-the-app.jpg)

## Build the app's image

1. We use Dockerfile to build the image. Dockerfile is a blue-print that holds a script of instructions such as, the base of image (e.g. node:24-alpine), sets the working directory, copy source code into the image, dependencies, commands to start the application and port for the application. Docker uses this script to build a container image.

- Go to the repository, change the browser to "dev mode" to make it possible to modify and commit the Dockerfile we will create.

- inside Getting-started-app folder, create a file called "Dockerfile" and paste the text:

![dockerfile](/images/dockerfile.jpg)

- save the change, commit and go to the terminal to pull the file running the command "git pull"

2. Build the image

- Building the images is the process of wrapping the instructions from Dockerfile such as, dependencies and the environment into the application.

- Navigate to the directory where Dockerfile is located:
  - cd getting-started-app
- Build the image running:
  - docker build -t getting-started .
  - -t command tags the image with following name and the dot means it uses the Dockerfile in the current directory (.).

## Start an app container

Use the command "docker run" to run a container:

- docker run -d -p 8080:3000 getting-started

- The -d flag runs the container in the backgroud, and the -p flag creates a port mapping where 8080 is the port exposed on the VM, while 3000 is the port the applications is listening (defined on Dockerfile)

- Now we can see the application on browser running on port 8080.

![docker-run](/images/docker-run.jpg)

---

# 2. Update the application

We will change the text displayed for when there is no item in the list, the current text is "No items yet! Add one above!", the new text will be "You have no todo items yet! Add one above!".

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

We can share a Docker image by using docker registry. Docker Hub is the default registry, where we can create repositories and share the application. Let's share the image created on step 2.

## Create a repository

After singing up in Docker Hub, create a repository named "getting-started" and make sure visibility is "Public".

To push an image in to Docker Hub we need to tag the image with Docker ID + repository name using the command "docker tag". Log in Docker Hub to get the docker ID.
run:
`docker tag getting-started:latest <docker-id>/getting-started:<newtag>`

Now that we have the new tag, we can push the image in to Ducker hub repository, but first make sure to log in using the command "docker login". (access token ca be generate on Docker Hub portal account settings).

![dockerhub-new-tag-for-sharing-image](/images/new-tag-for-sharing-image.jpg)

The image is now on Docker Hub, to test it we will first remove all old images, then use the shared image by pulling it from Docker Hub to run a new container.

![removed-img-pull-img-from-docker-hub](/images/remove-img-pull-img-from-docker-hub.jpg)

# 4. Persist the DB

Each Docker container has its own filesystem composed of multiple layers, it also contains the writable "scratch space" to create, update, and remove files, which allow them to have isolated filesystem for each container even though they are created from the same image. However, those files are lost once the container is removed and this is why the "todo list" in "getting-started-app" is empty every time a new container is created, unless "Docker volume" is used to persist the data.

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

- After running the container with the volume, open the application in your browser to add items to the todo list, after that remove and delete the container and start a new container with the volume we created to see the persistent data.

- Check where Docker stores the volume running:
  `docker volume inspect todo-db`

The image below shows the creation of a volume, running a container with volume and checking where volume is stored.

![persisting-data-docker-volume](/images/persisting-data-docker-volume.jpg)

# 5. Use bind mounts

The **bind mount** is another way of sharing data from the filesystem on the host machine into the container. The difference is that we can specify the path of the file or directory to be mounted in to the container when using bind mounts, while volumes are managed by Docker in the filesystem's Docker area.

A bind mount is great solution for the case where sharing data of a specific file/directory already defined in the host machine.

We will see how we can use bind mounts and **nodemon** tool to watch for file changes.

![mounts-bind-diagram](/images/mounts-bind-diagram.png)

(Check Docker documentation for **tmpfs mounts** - temporary filesystem in memory: https://docs.docker.com/engine/storage/tmpfs/)

As volumes (named volumes) are manage by Docker we only need to provide the name of the volume when mounting, while running with bind volumes we provide the path for this data, see the following commands comparison:

- Named Volume: `type=volume,src=my-volume,target=/usr/local/data`
- Bind mount: `type=bind,src=/path/to/data,target=/usr/local/data`

## Trying out bind mounts

We will understand now how bind mounts work with an experiment and then use bind mounts for the getting-started-app.

Note: Since this application is running on a Linux VM in Azure Cloud, Docker can access the filesystem in the host machine without the need of any configuration. Docker Desktop with WSL2 makes it possible for Docker to access the filesystem on Windows host machine, while on MacOS Docker Desktop uses a lightweight Linux VM where filesystem sharing is handled automatically, therefore if Docker is in Hyper-v mode, which is an isolated virtual machine, configuration is needed in the "File Sharing".

The experiment is to run an Ubuntu container just to use bind mount and see the file created inside getting-started-app (host machine) and inside the container.

- Go into getting-started-app `cd getting-started-app`
- Run the command to start bash in an Ubuntu container with a bind mount:
  - `docker run -it --mount type=bind,src=.,target=/src ubuntu bash`

The "--mount type=bind" means craetes a bind mount, "src" means the source (origem - host machine), while "target" is the destination inside the container.

The "-it" combines two flags:
"-i" = interactive (container accepts input), while "-t" = tty (allocates a pseudo terminal session, allowing the interaction). This is why you will see: root@ac1237fad8db:/#.

"Ubunto bash" means that Docker starts an Ubunto container and runs "bash".

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

After saving the change, refresh the browser and confirm the change of button label.

Every time a change is made, the **nodemon** watches the change and reflects it into the container. After stopping the container, we can build a new image with the changes.

![new-image-docker-dev-updated](/images/new-image-docker-dev-updated.jpg)

# 6. Multi Container Apps

So far the application we run was in a single container. We will add MySQL to the stack application and the question is where the database should run?

The recommended approach is that every container should have a single responsibility. When each process runs in a separate container, the application and the data base can be scaled, updated, and managed independently, otherwise, running multiple processes together in the same container would require a process manager, which increases complexity.

## Container Networking

By default, containers run in isolation and they do not know anything about others processes. To allow container to communicate with each other, we need to place them in the same Docker network.

## Start MySQL

Two ways to place a container on a network:

- Assign the network when starting the container.
- Connect an already running container to a network.

For this application we will create the network first and then attach MySQL container at startup:

To create the network run:
`docker network create todo-app`

Start a MySQL container and attach it to the network:

docker run -d \
 --network todo-app --network-alias mysql \
 -v todo-mysql-data:/var/lib/mysql \
 -e MYSQL_ROOT_PASSWORD=secret \
 -e MYSQL_DATABASE=todos \
 mysql:8.0

(There are a few "environment variables" required to initialize the database. You can learn more about variable at: https://hub.docker.com/_/mysql/)

Confirm that the database is running correctly:
`docker exec -it <mysql-container-id> mysql -u root -p`

- password: secret

![docker-network](/images/docker-network.jpg)

Now the database is ready to use.

## Connect to MySQL

Each container has its own IP address, and containers communicate to each other through these addresses when they are connected to the same network.

Previously, we defined the container for database using option "--network-alias mysql", which allows Docker to resolve the hostname "mysql" in the todo-app network DNS.

To look up this hostname we will use **nicolaka/netshoot** container that has a lot of useful tools for troubleshooting and debugging network container.

Learn more about **nicolaka/netshoot** at: https://github.com/nicolaka/netshoot

Start the netshoot container using nicolaka/netshoot image connected to the same network (todo-ap).

- `docker run -it --network todo-app nicolaka/netshoot`

![container-nicolaka-netshoot](/images/container-nicolaka-netshoot.jpg)

Inside the container run the command `dig mysql` which is a DNS tool - it returns the IP address for the hostname "mysql"

![dig-mysql](/images/dig-mysql.jpg)

The application only needs to connect to the hostname "mysql" to talk to the database.

## Run getting-started-app with MySQL

Todo app supports a few "environment variables" that allow it to connect to MySQL:

- MYSQL_HOST - the hostname for the running MySQL server
- MYSQL_USER - the username for the connection
- MYSQL_PASSWORD - the password for the connection
- MYSQL_DB - the database to use once connected

Note: Env vars are accepted for development, but they are not recommended for storing sensitive data in production. Instead, container orchestration framework should be used. Learn more about it at: https://blog.diogomonica.com/2017/03/27/why-you-shouldnt-use-env-variables-for-secret-data/

## Run the Application

To run the application, first go in to the "getting-started-app" directory and then add the previous environment variables, as the image shows. See that after starting the container we can inspect the logs with the command `docker logs -f <container-id>` and confirm its connected with MySQL.

![run-app-with-mysql](/images/run-app-with-mysql.jpg)

## Verify that data is stored in MySQL

Now the the container running and connected to the database, we can add items in the Todo List in browser. Then confirm that the data is stored in the "todos database" using the command:
`docker exec -it <mysql-container-id> mysql -p todos`

Inside MySQL:

`mysql> select * from todo_items;`

![todo-items-on-mysql-data-base](/images/todo-items-on-mysql-data-base.jpg)

# 7. Use Docker Compose

**Docker Compose** is used to define and share multi-container applications by writing a YAML file with all services. It allows us to run or tear down the application with a single command line.

The YAML file is generally created in the root of the application which is version controlled and easy to run.

## Create the Compose file

Inside "getting-started-app" directory, we will create a file named "compose.yaml".

## Define the app service

These are the commands we used to run multi-container application in "part 6".

docker run -dp 8080:3000 \
 -w /app -v ".:/app" \
 --network todo-app \
 -e MYSQL_HOST=mysql \
 -e MYSQL_USER=root \
 -e MYSQL_PASSWORD=secret \
 -e MYSQL_DB=todos \
 node:24-alpine \
 sh -c "npm install && npm run dev"

They will be defined in the "compose.yaml" file.

1. Start by defining the "name"(app:) and the "image:" of the first service or container. The name will become automatically a network alias (useful to define MySQL service).

2. Add the "command:" to install dependencies and run the application.

3. Define the "ports:" (8080:3000)

4. Define the working directory "-w /app" and volume mapping "-v: "./:/app"" by using "working_dir:" and "volumes:" definitions.

5. Define the environment variables by using "environment:" key.

![define-services-name-image-command-ports-wdir-volumes-env](/images/define-services-name-image-command-ports-wdir-volumes-env.jpg)

## Define MySQL service

Previous command used to run MySQL container:

docker run -d \
 --network todo-app --network-alias mysql \
 -v todo-mysql-data:/var/lib/mysql \
 -e MYSQL_ROOT_PASSWORD=secret \
 -e MYSQL_DATABASE=todos \
 mysql:8.0

1. Define the new service and name it "mysql" (it will be the network alias). Also define the image "mysql:8.0".

2. Define volume mapping. In Docker Compose, we need to declare the named volume at the global level and reference it inside the service so when the application starts Docker will create the volume automatically.

3. Define environment variables.

![define-mysql-service-compose-file](/images/define-mysql-service-docker-compose-file.jpg)

When the compose file is ready, commit and push it, and then pull to the VM.

![git-pull-compose-file](/images/git-pull-compose-file.jpg)

## Run the application stack

Before running, make sure there is no other copies of containers running. List containers using `docker ps` and remove them using `docker rm -f <ids>`.

Start the application stack running `docker compose up` and add -d to run in the background.

Check the logs using `docker compose logs -f` command.|

- It shows logs in real time for all services(app or MySQL), making it easier to identify connection issues.

Note that Docker creates the "network" automatically, allowing services to communicate with each other using the services names as hostnames, this is why we did not define it in the compose file.

![run-with-docker-compose](/images/run-with-docker-compose.jpg)

## Conclusion

In this assignment we demonstrated how Docker can be used to containerise applications and deploy them in a cloud-based virtual environment. Multi-containers were managed by using Docker Compose, improving scalability and maintainability.

The use of volumes for data persistence. Container networking enabled communication between container where each container followed a single responsability, reflecting best practices for deployment.
