# Docker Container
## What happens in 'docker container run'
- Looks for image locally in image cache, doesn't find anything.
- Then looks in remote image repository (defaults to Docker Hub).
- Downlaods the latest version (nginx:latest by default).
- Creates a new container based on that image and prepares to start.
- Gives it a virtual IP on a private network inside docker engine.
- Open up port 80 on host and forwards to port 80 in container i.e -p 80:80.
- Starts container by using CMD in the image Dockerfile.

`docker container run --publish 8080:80 --name webhost -d nginx:1.11 nginx -T`

# Container vs VM
## Containers aren't Mini-VM's
- They are just processes.
- Limited to what resources they can access.
- Exit when the process stops.

## Basic Step: Manage Multiple Containers
## Task
- docs.docker.com and `--help` are your friends.
- Run a nginx, a mysql, and a httpd (apache) server.
- Run all of them --detach (or -d => to make it run in background), name with --name (--name=webhost).
- nginx should listen on `80:80`, httpd on `8080:80`, mysql on `3306:3306`
- When running mysql, use the --env option (or -e ) to pass in `MYSQL_RANDOM_ROOT_PASSWORD=yes`
- Use `docker container logs ` on mysql to find the random password it created on startup.
- Clean it all up with `docker container stop` and `docker container rm {container_id}` (both can accept multiple names or ID's) 

## Solution
- Running mysql
    `docker container run -d -p 3306:3306 --name=db -e MYSQL_RANDOM_ROOT_PASSWORD=yes mysql`
    - Mysql password in logs `docker container logs db`

- Running apache
    `docker container run -d --name= webserver -p 8080:80 httpd`

- Running ngninx
    `docker container run -d --name proxy -p 80:80 nginx`

- Check all
    `docer container ls` should return procces running all these containers