# Example Voting App

## How to set up a Kubernetes cluster

### Master Node

 1. In a VM, use a linux
 2. run `sudo apt update && sudo apt install ufw`
 3. run `sudo ufw disable`
 4. then, run `sudo reboot`
 5. `sudo -s`
 6. `curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="644" sh -s` to install k3s
 7. `kubectl get nodes` to see all nodes running and if a master is set
 8. extract token from `sudo cat /var/lib/rancher/k3s/server/node-token` we will use it when connecting the pis
 9. you can use config from `sudo kubectl config view --raw > ~/.kube/config` to access a cluster from other machines

### Worker Node

 1. Download Raspberry Pi Imager from https://www.raspberrypi.com/software/
 2. Choose Pi version that suits your board
 3. Choose Operating System: “Raspberry Pi OS (64-bit)” 
 4. Select your SD card Storage
 7. Click the settings button and configure the options as follows
	 - Hostname: raspberrypi.local
	 - Set username: sds and set password: sds
	 - Enable SSH, use password authentication.
	 - Configure Wireless LAN with the router's SSID and password
 5. After finish writing, put the SD card back in Raspi
 6. run `ssh sds@raspberrypi.local` 

After SSH into Raspi 
 1. run `sudo apt update && sudo apt install ufw`
 2. run `sudo nano /boot/firmware/cmdline.txt` and add the following options: `cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory`
 3. run `sudo ufw disable`
 4. then, run `sudo reboot` and let pi reboot

Connect to a worker node to master node
 1. grab token and IP address from master node and run `curl -sfL https://get.k3s.io | K3S_TOKEN="<Token From Main Cluster>" K3S_URL="https://<Main Cluster IP>:6443" K3S_NODE_NAME="<Node Name>" sh -`
 2. try to run `kubectl get nodes` on master to check the connection

<hr>

## About a project

A simple distributed application running across multiple Docker containers.

## Getting started

Download [Docker Desktop](https://www.docker.com/products/docker-desktop) for Mac or Windows. [Docker Compose](https://docs.docker.com/compose) will be automatically installed. On Linux, make sure you have the latest version of [Compose](https://docs.docker.com/compose/install/).

This solution uses Python, Node.js, .NET, with Redis for messaging and Postgres for storage.

Run in this directory to build and run the app:

```shell
docker compose up
```

The `vote` app will be running at [http://localhost:8080](http://localhost:8080), and the `results` will be at [http://localhost:8081](http://localhost:8081).

Alternately, if you want to run it on a [Docker Swarm](https://docs.docker.com/engine/swarm/), first make sure you have a swarm. If you don't, run:

```shell
docker swarm init
```

Once you have your swarm, in this directory run:

```shell
docker stack deploy --compose-file docker-stack.yml vote
```

## Run the app in Kubernetes

The folder k8s-specifications contains the YAML specifications of the Voting App's services.

Run the following command to create the deployments and services. Note it will create these resources in your current namespace (`default` if you haven't changed it.)

```shell
kubectl create -f k8s-specifications/
```

The `vote` web app is then available on port 31000 on each host of the cluster, the `result` web app is available on port 31001.

To remove them, run:

```shell
kubectl delete -f k8s-specifications/
```

## Architecture

![Architecture diagram](architecture.excalidraw.png)

* A front-end web app in [Python](/vote) which lets you vote between two options
* A [Redis](https://hub.docker.com/_/redis/) which collects new votes
* A [.NET](/worker/) worker which consumes votes and stores them in…
* A [Postgres](https://hub.docker.com/_/postgres/) database backed by a Docker volume
* A [Node.js](/result) web app which shows the results of the voting in real time

## Notes

The voting application only accepts one vote per client browser. It does not register additional votes if a vote has already been submitted from a client.

This isn't an example of a properly architected perfectly designed distributed app... it's just a simple
example of the various types of pieces and languages you might see (queues, persistent data, etc), and how to
deal with them in Docker at a basic level.
