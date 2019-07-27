# Docker Container Tutorials
## What happens in 'docker container run'
- Looks for image locally in image cache, doesn't find anything.
- Then looks in remote image repository (defaults to Docker Hub).
- Downlaods the latest version (nginx:latest by default).
- Creates a new container based on that image and prepares to start.
- Gives it a virtual IP on a private network inside docker engine.
- Open up port 80 on host and forwards to port 80 in container i.e -p 80:80.
- Starts container by using CMD in the image Dockerfile.

`docker container run --publish 8080:80 --name webhost -d nginx:1.11 nginx -T`

### Container vs VM
#### Containers aren't Mini-VM's
- They are just processes.
- Limited to what resources they can access.
- Exit when the process stops.

### Basic Step: Manage Multiple Containers
#### Task
- docs.docker.com and `--help` are your friends.
- Run a nginx, a mysql, and a httpd (apache) server.
- Run all of them --detach (or -d => to make it run in background), name with --name (--name=webhost).
- nginx should listen on `80:80`, httpd on `8080:80`, mysql on `3306:3306`
- When running mysql, use the --env option (or -e ) to pass in `MYSQL_RANDOM_ROOT_PASSWORD=yes`
- Use `docker container logs ` on mysql to find the random password it created on startup.
- Clean it all up with `docker container stop` and `docker container rm {container_id}` (both can accept multiple names or ID's) 

#### Solution
- Running mysql
    `docker container run -d -p 3306:3306 --name=db -e MYSQL_RANDOM_ROOT_PASSWORD=yes mysql`
    - Mysql password in logs `docker container logs db`

- Running apache
    `docker container run -d --name= webserver -p 8080:80 httpd`

- Running ngninx
    `docker container run -d --name proxy -p 80:80 nginx`

- Check all
    `docer container ls` should return procces running all these containers

- Stop all containers
    `docker container stop {container ID's sep wwith , }`

- Remove them all
    List them `docker container ls -a`
    Remove them `docker container rm {container ID's sep with , }`

### What's going on in containers
- `docker container top` proccess list in one container.
- `docker container inspect` details of on container config.
- `docker container stats` performance stats for all containers.

#### Let's try these commands
- Start nginx 
    `docker container run -d --name nginx nginx` will fetch from images locally if the images is not removed.

- Start mysql
    `docker container run -d --name mysql -e MYSQL_RANDOM_ROOT_PASSWORD=true mysql` will fetch from images locally if the images is not removed.

 - List containers
    `docker container ls`

##### Docker top
- Docker Top on mysql
    `docker container top mysql` -> will return running proccesses inside the container

- Docker Top on nginx
    `docker container top mysql` -> will return running proccesses inside the container

##### Docker inspect
- Docker inspect on mysql
    `docker container inspect mysql` -> this will return json format metadata of the mysql container (startup config, volumes, networking etc.)

- Docker inspect on nginx
    `docker container inspect nginx` -> this will return json format metadata of the nginx container (startup config, volumes, networking etc.)

#### Docker stats
- Docker stats
    `docker container stats` will give us live streaming view of live performance data for all containers.

### Getting a shell inside containers
- Start a new container interacively
    `docker container run -it`
    `-t` pseudo-tty simulates a real terminal, like what SSH does.
    `-i` interactive session open to recieve teminal input
- Getting a shell inside a new container
    `docker container run -it --name proxy nginx bash`
- Run additional commands/get shell in existing container
    `docker container exec -it proxy bash`
    proxy the container name and bash the shell

### Docker network concepts
- Review of `docker container -p`
- For local dev/testing, networks usually "just work"
- Quick port check with `docker container port <container name>`
- Learn concepts of Docker Networking
- Understand how networks packets move around Docker

#### Docker networks defaults
- Each container connect to a private virtual network "bridge"
- Each virtual network routes through NAT firewall on host IP
- All containers on virtual netwrok can talk to each other without `-p`
- If we want specific containers to talk to each other inside our host we don't really need the `-p` option.
- Best practce is to create a new virtual network for each app
    - network "my_web" for mysql and php/apache containers
    - network "my_api" for mongo and nodejs containers
- Attach containers to more then one virtual networks (or none)
- Skip virtual network and use host IP (`--net=host`)
- Use different Docker network drivers to gain new abilities and much more ...

#### Docker network practice
- `-p` --publish
- Remember publishing ports are always in **HOST:CONTAINER** format
- Let's run nginx 
    - `docker container run -p 80:80 --name webhost -d nginx`
- Let's run the port command on nginx
    - `docker container port webhost`
    returns -> `80/tcp -> 0.0.0.0:80`
- Containers IP address
    we can find the ip address of any container with this command `docker container inspect --format '{{.NetworkSettings.IPAddress}}' webhost`
    where webhost is the name of the container

### Docerk Network CLI Managment
- show networks `docker network ls`
- inspect a network `docker network inspect`
- Attach a network to container `docker network create --driver`
- Detach a netwrok from container `deocker network disconnect`
#### Docker Network Default Security
- Create your apps so frontend/backend sit on same Docker network
- Their inter-communication never leaves host
- All externally exposed ports closed by default
- You must manually expose via `-p`, which is better default security
- This gets even better later with swarm and Overlay netwroks

#### Docker Networks DNS
- Understand how DNS is the key to easy inter-container communication
- See how it works by default with custom netwrorks
- Learn how to use `--link` to enable DNS on default bridge network
- Docker deamon has a buil-in DNS server that containers use by default
- Containers shouldn't rely on IP's for inter-communication
- DNS for friendly names is built-in if you use custom networks
- This gets way easier with Docker Compose in future section

## Practice 
###  Task 01
- Use different linux distro containers to check `curl` cli tool version
- Use two different terminal window to start bash in both `centos:7` and `ubuntu:14.04`, using `-it`
- Learn `docker container --rm` option so you can save cleanup
- Ensure `curl` is installed on latest version for that distro
    - Ubuntu: `apt-get update && apt-get install curl`
    - centos: `yum update curl`
- Check `curl --version`

#### Solution
- CentOS
    - `docker container run --rm -it centos:7 bash` once inside a shell, intall curl `yum update curl` & run `curl --version`
- Ubunutu
    - `docker container run --rm  -it ubuntu:14.04 bash` once insde a shell, install curl `apt-get update && apt-get install curl` & run `curl --version`
- `--rm` is used when we need to do a temporary task like testing or getting a shell for a while for some task to be done, once we exit the shell the container is gone we can verify this by running `docker container ls`

# Docker Images Tutorials
## What's in an image (what isn't)
- App binaries and dependencies
- Metadata about the image data and how to run the image
- Official definition: "An image is an ordered coolection of root filesystem changes and the corresponding execution parameters for use within a container runtime"
- Not a complete OS. No kernal, kernal modules (e.g drivers)
- Small as one file (your app binary) like a golang static binary
- Big as a ubuntu distro with apt, and Apache, PHP and more installed
### The mighty hub
