# Docker Cheat Sheet

## Docker Run

Docker `run` will `create` and `start` a container using the hello-world `image`. Pay attention to the output of this container which describes exactly what happened during the `Run` process:

```bash 
# Create a container using the hello-world image and start it
docker run hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete 
Digest: sha256:2557e3c07ed1e38f26e389462d03ed943586f744621577a99efb77324b0fe535
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```

We'll talk about Docker `create` and Docker `start` later.

<hr>

## Overriding Default Commands

You can override the images's `Startup Command`, provided that the command exists on the `FS Snapshot` (Remember an image is a `FS Snapshot` + `Startup Command`). The busybox image containes a `FS snapshot` that has the echo and ls command so we can successfully run these commands when overriding the default `Startup Command`:

``` bash
# Override the busybox image's default command (sh)
docker run busybox echo "Mary had a little lamb"
Mary had a little lamb

# Any command that is available in the busybox image's FS image will work
docker run busybox ls
bin
dev
etc
home
proc
root
sys
tmp
usr
var
```

Note that you cannot override the default commands once the container is created:

``` bash 
# Creating with override
docker create busybox echo "Hello"
4be888c490c0ee410a19929b08c2965c4abe674979d875c529370547af6fb694

# Starting the container
docker start -a 4be888c490c0ee410a19929b08c2965c4abe674979d875c529370547af6fb694
Hello

# Attempting to overrride the default command when starting a pre-created container: ERROR
docker start -a 4be888c490c0ee410a19929b08c2965c4abe674979d875c529370547af6fb694 echo "Bye"
you cannot start and attach multiple containers at once


```


<hr>

## Docker Create
Docker `create` loads the `FS Snapshot` but does not `start` the container. Notice that the ID of the newly created container is returned:

``` bash
# Create a container using the hello-world image 
docker create hello-world
50c4e972f8423d498d74ea3a225911b33338f37d6fc2953bd71f4a0756ee67a8

```

## Docker Start

There are two different ways to run docker `start`:


``` bash
# The -a or`--attach option allows us to attach to the container
# This will cause the output of the container to be printed to the console
docker start -a 50c4e972f8423d498d74ea3a225911b33338f37d6fc2953bd71f4a0756ee67a8
```

``` bash
# If the -a option is not used, the output of the container will not be printed to the terminal
docker start 50c4e972f8423d498d74ea3a225911b33338f37d6fc2953bd71f4a0756ee67a8
```

Notice how docker `run` is actually a combination of docker `create` + docker `start -a` since it automaticaly attaches to the container during startup by default.

<hr>

## Docker Logs

If we forgot to attach to the container using docker `start -a`, we can use the `logs` to view what was logged/being logged by the container. It does not restart the container!

``` bash
# Create a container overriding the default command
docker create busybox echo "Hello"
844b154986f7f8ea16867323b07a53eb218acddffd31b744e5bb56a923e7df30

# Start the container without attaching to it
docker start 844b154986f7f8ea16867323b07a53eb218acddffd31b744e5bb56a923e7df30
844b154986f7f8ea16867323b07a53eb218acddffd31b744e5bb56a923e7df30

# Use docker logs to retrieve the logs of the container
zng-mac:docker chester$ docker logs 844b154986f7f8ea16867323b07a53eb218acddffd31b744e5bb56a923e7df30
Hello
```

<hr>

## Stopping Containers. Stop or Kill?

Both docker `stop` and docker `kill` will stop the container. Here are the differences:

``` bash
# Docker stop sends a hardware signal SIGTERM to the primary process inside the container (graceful). Waits 10s before falling back to docker kill
docker stop 

# Docker kill sends a hardware signal SIGKILL to the primary process inside the container (non-graceful)
docker kill

# Create a container that will ping google.com continuously
docker create busybox ping google.com
56b8414d6b4f2c6cbd33adbb33b581f1f158882796975dcfdb89fe6b93949de0

# Start the container
docker start 56b8414d6b4f2c6cbd33adbb33b581f1f158882796975dcfdb89fe6b93949de0

# Attempt to stop it (hint, ping command does not responds well to SIGTERM signal) so it will wait for 10s beore a SIGKILL signal is sent to termniate it, taking down the container
docker stop 56b8414d6b4f2c6cbd33adbb33b581f1f158882796975dcfdb89fe6b93949de0

```

<hr>

## Executing commands in running containers

Docker `exec` allows us to execute additional commands in the container. The `-it` options allows us to type input. More about the `it` option below.

```bash
# Create a container using the redis image which starts a redis server
docker create redis

# Start the redis server container
docker start 59f5e0d11745

# Verify that redis server is running
docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
59f5e0d11745        redis               "docker-entrypoint.s…"   3 minutes ago       Up About a minute   6379/tcp            vibrant_pike

# Execute the redis-cli in the redis server container
docker exec -it 59f5e0d11745 redis-cli
127.0.0.1:6379> set myvalue 5
OK
127.0.0.1:6379> get myvalue
"5"

# If we try to run the same command without the -it option, the redis-cli command will run but we will immediately be kicked out since there is no way for us to input anything to the prompt
docker exec 59f5e0d11745 redis-cli
```

###  The Purpose of the IT flag

The `-it` option is actually made up of two distinct options `-i` and `-t`

`-i` Then we execute the command, attach terminal to the STDIN stream of that process

`-t` Formatting 

### Getting a command prompt in a container
``` bash
# sh into the container!
docker exec -it 59f5e0d11745 sh

```

Note that we could also have done this without the `exec` command and overriding the default command of a container

```bash
# Create and start a busybox container, immediately sh into it
docker run -it busybox sh
```

<hr>

## List Running Containers

``` bash
# List all running containers. without the -a or --all option only live containers are listed

docker ps --all

CONTAINER ID        IMAGE                            COMMAND                  CREATED             STATUS                      PORTS                  NAMES
01a964a9e8e7        busybox                          "sh"                     2 minutes ago       Exited (0) 2 minutes ago                           fervent_kapitsa
eb12ad3ae977        busybox                          "ls"                     7 minutes ago       Exited (0) 7 minutes ago                           lucid_spence
855431f63598        busybox                          "echo 'little lamb'"     8 minutes ago       Exited (0) 8 minutes ago                           cranky_booth
b2b266dce88e        busybox                          "echo 'little lamb'"     8 minutes ago       Exited (0) 8 minutes ago                           gallant_hermann
282c7d0d5daa        busybox                          "echo 'Mary had a li…"   8 minutes ago       Exited (0) 8 minutes ago                           angry_hodgkin

```


## Removimg Stopped Containers

``` bash
# Remove all stopped containers
docker system prune
```






