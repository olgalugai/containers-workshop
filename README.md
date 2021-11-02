# Hands-On: Introduction to Containers (Docker and Minikube)

> **Tasks**:

> - [Task 0: Prerequisites](#Task_0)
> - [Task 1: Run simple containers](#Task_1)
> - [Task 2: Package your custom application using Docker](#Task_2)
> - [Task 3: Running your application on Kubernetes locally](#Task_3)

## <a name="task0"></a>Task 0: Prerequisites

First, you need to install Docker and Minikube.

### Install Docker Desktop on Mac

[Docker Official Documentation](https://docs.docker.com/desktop/mac/install/)

### Install Docker Engine with Homebrew

Download and install both the docker-machine and virtualbox packages. Docker requires both of these to run correctly on macOS.

```
brew install docker
brew install docker-machine
brew install --cask virtualbox
```

### Install Docker Desktop with Homebrew

```
#Install Docker Desktop
brew install --cask docker
#Run Docker Desktop
open /Applications/Docker.app
```

### Install Minikube

To install the latest minikube stable release on x86-64 macOS using binary download:

```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikube
```

### Install Minikube with Homebrew

`brew install minikube`
If which minikube fails after installation via brew, you may have to remove the old minikube links and link the newly installed binary:

```
brew unlink minikube
brew link minikube
```

## <a name="Task_1"></a>Task 1: Run simple containers

### Run a simple nginx container

1. Check if Docker was installed successfully:

`docker version`

2. Run the following command:

   ```
   $ docker run -d -p 80:80 --name webserver nginx

   ```

When `nginx:latest` image could not be found locally, Docker automatically _pulls_ it form Docker Hub.

You’ll notice a few flags being used. Here’s some more info on them:

- -d - run the container in detached mode (in the background)
- -p 80:80 - map port 80 of the host to port 80 in the container
- nginx - the image to use

3. Try to access a container on localhost:80 on your machine.

4. Check how long it will take to start a new container with a new image:

`docker run -d -p 8080:80 --name webserver2 nginx`

The image was downloaded already, so this container started faster.

5. You can run bash commands inside a Linux container:

`docker exec -it webserver /bin/bash`

docker exec runs a command in a running container and the switch -it opens an interactive console

6. Stop containers and remove the image:

```
docker stop webserver && docker stop webserver2
docker rm webserver && docker rm webserver2
Docker rmi <image name>
```

### Task 2: Package your custom application using Docker

You can create an image for your application by using Dockerfile. It contains a list of instructions to build images such as installing a package, downloading source code, using a base image.

1. Create a Dockerfile and add the following:

```
FROM ubuntu
RUN apt-get update
RUN apt-get install -y php5
COPY preconditions.txt /usr/temp
```

[FROM](https://docs.docker.com/engine/reference/builder/#from) specifies the base image to use as the starting point for this new image you're creating.
[RUN](https://docs.docker.com/engine/reference/builder/#run) Executes a command on top of an existing layer and creates a new resulting layer
[COPY](https://docs.docker.com/engine/reference/builder/#copy) Copies files and directories from a source (host executing build command) to a destination (Container image)

2. Run`docker image build` command to create a new Docker image using the instructions in your Dockerfile.

- `--tag` allows us to give the image a custom name. It usually consists of your Docker Hub username, the application name, and a version.
- `.` tells Docker to use the current directory as the build context

Be sure to include period (`.`) at the end of the command.

```
$ docker image build --tag <yourusername>/<image-name>:1.0 .
```

3. Push your image to Docker Hub

List local images:
`docker images`

Log in to Docker Hub:
`docker login --username <yourusername>`

Push the image to your repository in Docker Hub
`docker push yourusername/nginx:1.00`

### Task 3: Running your application on Kubernetes locally

1. kubectl - the Kubernetes command-line tool allows you to run commands against Kubernetes clusters. Install kubectl:

`curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"`

Make the kubectl binary executable.

`chmod +x ./kubectl`

Move the kubectl binary to a file location on your system PATH.

```
sudo mv ./kubectl /usr/local/bin/kubectl
sudo chown root: /usr/local/bin/kubectl
```

Install with Homebrew on macOS

`brew install kubectl`

Check if kubectl was installed:

`kubectl version --client`

2. Start your cluster:

`minikube start`

After the cluster is started, you can verify its status.

`minikube status`

See pods in your cluster:

`kubectl get po -A`

3. Create a sample deployment and expose it on port 8080:

```
kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.4
kubectl expose deployment hello-minikube --type=NodePort --port=8080
```

It may take a moment, but your deployment will soon show up when you run:

```
kubectl get services hello-minikube
```

The easiest way to access this service is to let minikube launch a web browser for you:

`minikube service hello-minikube`

Alternatively, use kubectl to forward the port:

`kubectl port-forward service/hello-minikube 7080:8080`

Now you can access your application at http://localhost:7080/.