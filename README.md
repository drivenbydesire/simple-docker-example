## About Docker

[Docker](https://www.docker.com/) is an application that simplifies the process of managing application processes in containers. Containers let you run your applications in resource-isolated processes. They’re similar to virtual machines, but containers are more portable, more resource-friendly, and more dependent on the host operating system.

For a detailed introduction to the different components of a Docker container, check out [The Docker Ecosystem: An Introduction to Common Components](https://www.digitalocean.com/community/tutorials/the-docker-ecosystem-an-introduction-to-common-components).

## Prerequisites
To follow this simple docker example, you will need the following:

- One Ubuntu 20.04 server set up by following the [Ubuntu 20.04 initial server setup guide](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04), including a sudo non-root user and a firewall.
  - I am using a microsoft-standard-WSL2 with Ubuntu 22.04.3 LTS Jammy Jellyfish.
- An account on [Docker Hub](https://hub.docker.com/) if you wish to create your own images and push them to Docker Hub, as shown in Steps 7 and 8.
<details>
<summary>Step 1 — Installing Docker</summary>

## Step 1 — Installing Docker


The Docker installation package available in the official Ubuntu repository may not be the latest version. To ensure we get the latest version, we’ll install Docker from the official Docker repository. To do that, we’ll add a new package source, add the GPG key from Docker to ensure the downloads are valid, and then install the package.

First, update your existing list of packages:
```
$ sudo apt update
```

Next, install a few prerequisite packages which let ```apt``` use packages over HTTPS:

```
$ sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

Then add the GPG key for the official Docker repository to your system:
```
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Add the Docker repository to APT sources:
```
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
```

This will also update our package database with the Docker packages from the newly added repo.

Make sure you are about to install from the Docker repo instead of the default Ubuntu repo:
```
$ apt-cache policy docker-ce
```

You’ll see output like this, although the version number for Docker may be different:
```
Output of apt-cache policy docker-ce
docker-ce:
  Installed: (none)
  Candidate: 5:19.03.9~3-0~ubuntu-focal
  Version table:
     5:19.03.9~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
```
Notice that ```docker-ce``` is not installed, but the candidate for installation is from the Docker repository for Ubuntu 20.04 (```focal```).

Finally, install Docker:
```
$ sudo apt install docker-ce
```
Docker should now be installed, the daemon started, and the process enabled to start on boot. Check that it’s running:
```
sudo systemctl status docker
```
The output should be similar to the following, showing that the service is active and running:

```
Output
o docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2020-05-19 17:00:41 UTC; 17s ago
TriggeredBy: o docker.socket
       Docs: https://docs.docker.com
   Main PID: 24321 (dockerd)
      Tasks: 8
     Memory: 46.4M
     CGroup: /system.slice/docker.service
             └─24321 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```
Installing Docker now gives you not just the Docker service (daemon) but also the ```docker``` command line utility, or the Docker client. We’ll explore how to use the ```docker``` command later.

</details>

<details>
<summary>Step 2 — Executing the Docker Command Without Sudo (Optional)</summary>  
  
  ## Step 2 — Executing the Docker Command Without Sudo (Optional) 
  
  By default, the ```docker``` command can only be run the **root** user or by a user in the **docker** group, which is automatically created during Docker’s installation process. If you attempt to run the ```docker``` command without prefixing it with ```sudo``` or without being in the **docker** group, you’ll get an output like this:
  ```
Output
docker: Cannot connect to the Docker daemon. Is the docker daemon running on this host?.
See 'docker run --help'.
```
If you want to avoid typing ```sudo``` whenever you run the ```docker``` command, add your username to the **docker** group:
```
$ sudo usermod -aG docker ${USER}
```
To apply the new group membership, log out of the server and back in, or type the following:

```
$ su - ${USER}
```
You will be prompted to enter your user’s password to continue.

Confirm that your user is now added to the **docker** group by typing:
```
$ groups
```
```
Output
sammy sudo docker
```

If you need to add a user to the **docker** group that you’re not logged in as, declare that username explicitly using:
```
$ sudo usermod -aG docker username
```

The rest of this article assumes you are running the ```docker``` command as a user in the **docker** group. If you choose not to, please prepend the commands with ```sudo```.

Let’s explore the ```docker``` command next.
</details>

<details>
<summary>Step 3 — Using the Docker Command</summary>

  ## Step 3 — Using the Docker Command

  Using ```docker``` consists of passing it a chain of options and commands followed by arguments. The syntax takes this form:

```
$ docker [option] [command] [arguments]
```
To view all available subcommands, type:
```
$ docker
```
As of Docker 19, the complete list of available subcommands includes:

```
Output
  attach      Attach local standard input, output, and error streams to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes
```

To view the options available to a specific command, type:

```
$ docker docker-subcommand --help
```

To view system-wide information about Docker, use:
```
$ docker info
```

Let’s explore some of these commands. We’ll start by working with images.
Remember this is just ```docker``` we will cover ```docker-compose``` in different repo


</details>

<details>
  <summary>Step 4 — Working with Docker Images</summary>

  ## Step 4 — Working with Docker Images

Docker containers are built from Docker images. By default, Docker pulls these images from [Docker Hub](https://hub.docker.com/), a Docker registry managed by Docker, the company behind the Docker project. Anyone can host their Docker images on Docker Hub, so most applications and Linux distributions you’ll need will have images hosted there.

To check whether you can access and download images from Docker Hub, type:
```
$ docker run hello-world
```
The output will indicate that Docker in working correctly:
```
Output
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete
Digest: sha256:6a65f928fb91fcfbc963f7aa6d57c8eeb426ad9a20c7ee045538ef34847f44f1
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

...
```

Docker was initially unable to find the ```hello-world``` image locally, so it downloaded the image from Docker Hub, which is the default repository. Once the image downloaded, Docker created a container from the image and the application within the container executed, displaying the message.

You can search for images available on Docker Hub by using the ```docker``` command with the ```search``` subcommand. For example, to search for the Ubuntu image, type:
```
$ docker search ubuntu
```

The script will crawl Docker Hub and return a listing of all images whose name match the search string. In this case, the output will be similar to this:
```
Output
NAME                                                      DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
ubuntu                                                    Ubuntu is a Debian-based Linux operating sys…   10908               [OK]
dorowu/ubuntu-desktop-lxde-vnc                            Docker image to provide HTML5 VNC interface …   428                                     [OK]
rastasheep/ubuntu-sshd                                    Dockerized SSH service, built on top of offi…   244                                     [OK]
consol/ubuntu-xfce-vnc                                    Ubuntu container with "headless" VNC session…   218                                     [OK]
ubuntu-upstart                                            Upstart is an event-based replacement for th…   108                 [OK]
ansible/ubuntu14.04-ansible                               Ubuntu 14.04 LTS with
...
```

In the **OFFICIAL** column, **OK** indicates an image built and supported by the company behind the project. Once you’ve identified the image that you would like to use, you can download it to your computer using the ```pull``` subcommand.

Execute the following command to download the official ```ubuntu``` image to your computer:
```
$ docker pull ubuntu
```

You’ll see the following output:
```
Output
Using default tag: latest
latest: Pulling from library/ubuntu
d51af753c3d3: Pull complete
fc878cd0a91c: Pull complete
6154df8ff988: Pull complete
fee5db0ff82f: Pull complete
Digest: sha256:747d2dbbaaee995098c9792d99bd333c6783ce56150d1b11e333bbceed5c54d7
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```
I did this

```
$ docker pull qwerty199369/nginx-php-mysql-redis:1.1.4
```
And you will see the output as:
```
1.1.4: Pulling from qwerty199369/nginx-php-mysql-redis
594cf71eb8ba: Pull complete
3b5244e3179b: Pull complete
37a19d5c10ab: Pull complete
d9e6649d6df2: Pull complete
2b401702069a: Pull complete
4f4fb700ef54: Pull complete
eae027dcdc0e: Pull complete
9f92b353dafc: Pull complete
31e4a09cb0b7: Pull complete
b2ac253feffe: Pull complete
d24eba9c955c: Pull complete
3c6456288dd3: Pull complete
3d6019e95f0b: Pull complete
5d9231d87964: Pull complete
969034f82c57: Pull complete
0f92df7564fd: Pull complete
cda147eae49f: Pull complete
7b8428dc5b68: Pull complete
Digest: sha256:b08c5cf201bd708a599b1c686a9114c0231b759cb77bae873448a3acd34aeb13
Status: Downloaded newer image for qwerty199369/nginx-php-mysql-redis:1.1.4
docker.io/qwerty199369/nginx-php-mysql-redis:1.1.4
```
After an image has been downloaded, you can then run a container using the downloaded image with the ```run``` subcommand. As you saw with the ```hello-world``` example, if an image has not been downloaded when ```docker``` is executed with the ```run``` subcommand, the Docker client will first download the image, then run a container using it.

To see the images that have been downloaded to your computer, type:
```
$ docker images
```

The output will look similar to the following:
```
Output
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              latest              1d622ef86b13        3 weeks ago         73.9MB
hello-world         latest              bf756fb1ae65        4 months ago        13.3kB
```
This will show if you followed my lines
```
qwerty199369/nginx-php-mysql-redis   1.1.4     099d247497ad   6 years ago     3.01GB
```

As you’ll see later in this tutorial, images that you use to run containers can be modified and used to generate new images, which may then be uploaded (_pushed_ is the technical term) to Docker Hub or other Docker registries.

Let’s look at how to run containers in more detail.

</details>

<details>
  <summary>Step 5 — Running a Docker Container</summary>

  ## Step 5 — Running a Docker Container

The ```hello-world``` container you ran in the previous step is an example of a container that runs and exits after emitting a test message. Containers can be much more useful than that, and they can be interactive. After all, they are similar to virtual machines, only more resource-friendly.

As an example, let’s run a container using the latest image of Ubuntu. The combination of the **-i** and **-t** switches gives you interactive shell access into the container:
```
$ docker run -it ubuntu
```

Your command prompt should change to reflect the fact that you’re now working inside the container and should take this form:
```
Output
root@d9b100f2f636:/#
```

For me, it was
```
$ docker run -it 099d247497ad
```

It took sometime to get some output, but here what I got:
```
docker run -it 099d247497ad

Starting MySQL (Percona Server)
.. 

```
So, I cancelled, and followed the ubuntu one as provided. Will try with own image later.

**Lets delete the 3GB image**

```
$ docker rmi 099d247497ad
Error response from daemon: conflict: unable to delete 099d247497ad (must be forced) - image is being used by stopped container a3a7bbd10bbe

$ docker rm -f a3a7bbd10bbe
a3a7bbd10bbe

$ docker rmi 099d247497ad
Untagged: qwerty199369/nginx-php-mysql-redis:1.1.4
Untagged: qwerty199369/nginx-php-mysql-redis@sha256:b08c5cf201bd708a599b1c686a9114c0231b759cb77bae873448a3acd34aeb13
Deleted: sha256:099d247497ad070904c7c21c3007a1652913be2d6885355f7289f9672f073306
Deleted: sha256:e084d6ecc305623de5c84aae46c4d649bfcc29ea4b1811ea88e0f073b83cf78c
Deleted: sha256:9b83a539f429f713aeb0a435baf009d1cc7693d0cfc2a95413acbda8e310f8dc
Deleted: sha256:ad0c09116cafbd42eb65339f07c9814d4acd66e73f4bf35a0a84f305ad31ba01
Deleted: sha256:5334d385f7b947ca7a7974788a49dc70e6346b60d8a8e9572e63bf508b94b8de
Deleted: sha256:86f01108df168ab5cc09e6851debb3c8b71acfe7e9dbe238d445d5848ed5fd6b
Deleted: sha256:f006367e453579872a1589068819efa3f0a673726d5ffc9fb440f34dde73da59
Deleted: sha256:9a96a498fc3d8e19c02ec0916d32653aef958b01e9afed42886448fd37ab1aa7
Deleted: sha256:3df362b96781fe6cb6bf36844ce58e22d22c7b21638f99fa9bdb5307006c890d
Deleted: sha256:33838cc1ea6cbebafd26b863028fd86c301ba1b206a6befc22aa0944e013368f
Deleted: sha256:00194d20792bccae2d524457ff7063455d8f5f2c406342a2f5f4a473c1c67def
Deleted: sha256:e1d74c74d4cf7ca6afc43bf90adfca1211119c3ce28b6d3a2356f60386c4b87d
Deleted: sha256:6f93dc6efe7c0399da9527dc3fafb95a55c36ebc6fc0d02b41e97cd0f0be7b5b
Deleted: sha256:08f29b3d2fa8865a3a64b7d0a29eca8fbc07a2ffdf451ef81e02a919848c8604
Deleted: sha256:8bbc3b474e1322b8c38c0888c1f5a7272c1f93712495e929a5bc1e4302a6ddce
Deleted: sha256:6c051a29e1326241964ebd30dada3f4dadd26ae1c4fb122a708655d116b9ceac
Deleted: sha256:fe33d1f36efd1d7965e94f28f69f89834293a985069027e72d7aaf47ef757662
Deleted: sha256:e49c10d52a57d0cdd0b056ab1466948f6098fd73cba650a222c8be6456974208
Deleted: sha256:733fd966395da1d275e0c6d38736decc590c90e65134f20e0117e72171484947
Deleted: sha256:ef5b99eed7c2ed19ef39f72ac19bb66e16ed6c0868053daae60306a73858fbd4
Deleted: sha256:257e51479af1e9d2e0c9b958e68f6b992329904df24d81efa191cef515a9bf8b
Deleted: sha256:6e1d2d371500e2fe6df75f5755d0b9f2a3b69a42fe88100d514212bbba7ad23f
Deleted: sha256:afa9e7a5e3f3b006942d128c562a3273947c7ab50cdac33fea7213890072a5b6
Deleted: sha256:2df9b8def18a090592bf1cbd1079e1ac2274435c53f027ee5ce0a8faaa5d6d4b
```


Note the container id in the command prompt. In this example, it is ```d9b100f2f636```. You’ll need that container ID later to identify the container when you want to remove it.

Now you can run any command inside the container. For example, let’s update the package database inside the container. You don’t need to prefix any command with ```sudo```, because you’re operating inside the container as the **root** user:

```
root@d9b100f2f636:/# apt update
```

Then install any application in it. Let’s install Node.js:
```
root@d9b100f2f636:/# apt install nodejs
```
This installs Node.js in the container from the official Ubuntu repository. When the installation finishes, verify that Node.js is installed:
```
root@d9b100f2f636:/# node -v
```
You’ll see the version number displayed in your terminal:
```
Output
v10.19.0
```
Any changes you make inside the container only apply to that container.

To exit the container, type ```exit``` at the prompt.

Let’s look at managing the containers on our system next.
</details>


<details>
  <summary>Step 6 — Managing Docker Containers</summary>

  ## Step 6 — Managing Docker Containers

After using Docker for a while, you’ll have many active (running) and inactive containers on your computer. To view the **active ones**, use:
```
$ docker ps
```
You will see output similar to the following:
```
Output
CONTAINER ID        IMAGE               COMMAND             CREATED
```
In this tutorial, you started two containers; one from the ```hello-world``` image and another from the ubuntu image. Both containers are no longer running, but they still exist on your system.

To view all containers — active and inactive, run ```docker ps``` with the ```-a``` switch:
```
$ docker ps -a
```
You’ll see output similar to this:
```
1c08a7a0d0e4        ubuntu              "/bin/bash"         2 minutes ago       Exited (0) 8 seconds ago                       quizzical_mcnulty
a707221a5f6c        hello-world         "/hello"            6 minutes ago       Exited (0) 6 minutes ago                       youthful_curie
```
To view the latest container you created, pass it the ```-l``` switch:
```
$ docker ps -l

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
1c08a7a0d0e4        ubuntu              "/bin/bash"         2 minutes ago       Exited (0) 40 seconds ago                       quizzical_mcnulty
```
To start a stopped container, use ```docker start```, followed by the container ID or the container’s name. Let’s start the Ubuntu-based container with the ID of ```1c08a7a0d0e4```:
```
$ docker start 1c08a7a0d0e4
```
The container will start, and you can use ```docker ps``` to see its status:
```
Output
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
1c08a7a0d0e4        ubuntu              "/bin/bash"         3 minutes ago       Up 5 seconds                            quizzical_mcnulty
```
To stop a running container, use ```docker stop```, followed by the container ID or name. This time, we’ll use the name that Docker assigned the container, which is ```quizzical_mcnulty```:
```
$ docker stop quizzical_mcnulty
```
Once you’ve decided you no longer need a container anymore, remove it with the ```docker rm``` command, again using either the container ID or the name. Use the ```docker ps -a``` command to find the container ID or name for the container associated with the ```hello-world``` image and remove it.
```
$ docker rm youthful_curie
```
You can start a new container and give it a name using the ```--name``` switch. You can also use the --rm switch to create a container that removes itself when it’s stopped. See the ```docker run help``` command for more information on these options and others.

Containers can be turned into images which you can use to build new containers. Let’s look at how that works.

</details>
<details>
  <summary>Step 7 — Committing Changes in a Container to a Docker Image</summary>

  ## Step 7 — Committing Changes in a Container to a Docker Image

  When you start up a Docker image, you can create, modify, and delete files just like you can with a virtual machine. The changes that you make will only apply to that container. You can start and stop it, but once you destroy it with the ```docker rm``` command, the changes will be lost for good.

This section shows you how to save the state of a container as a new Docker image.

After installing Node.js inside the Ubuntu container, you now have a container running off an image, but the container is different from the image you used to create it. But you might want to reuse this Node.js container as the basis for new images later.

Then commit the changes to a new Docker image instance using the following command.
```
$ docker commit -m "What you did to the image" -a "Author Name" container_id repository/new_image_name
```
The **-m** switch is for the commit message that helps you and others know what changes you made, while **-a** is used to specify the author. The ```container_id``` is the one you noted earlier in the tutorial when you started the interactive Docker session. Unless you created additional repositories on Docker Hub, the ```repository``` is usually your Docker Hub username.

For example, for the user **sammy**, with the container ID of ```d9b100f2f636```, the command would be:
```
$ docker commit -m "added Node.js" -a "sammy" d9b100f2f636 sammy/ubuntu-nodejs
```
When you _commit_ an image, the new image is saved locally on your computer. Later in this tutorial, you’ll learn how to push an image to a Docker registry like Docker Hub so others can access it.

Listing the Docker images again will show the new image, as well as the old one that it was derived from:
```
$ docker images
```
You’ll see output like this:
```
Output
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
sammy/ubuntu-nodejs   latest              7c1f35226ca6        7 seconds ago       179MB
...
```

In this example, ```ubuntu-nodejs``` is the new image, which was derived from the existing ```ubuntu``` image from Docker Hub. The size difference reflects the changes that were made. And in this example, the change was that NodeJS was installed. So next time you need to run a container using Ubuntu with NodeJS pre-installed, you can just use the new image.

You can also build Images from a ```Dockerfile```, which lets you automate the installation of software in a new image. However, that’s outside the scope of this tutorial.

Now let’s share the new image with others so they can create containers from it.

</details>

<details>
  <summary>Step 8 — Pushing Docker Images to a Docker Repository</summary>

  ## Step 8 — Pushing Docker Images to a Docker Repository

  The next logical step after creating a new image from an existing image is to share it with a select few of your friends, the whole world on Docker Hub, or other Docker registry that you have access to. To push an image to Docker Hub or any other Docker registry, you must have an account there.

This section shows you how to push a Docker image to Docker Hub. To learn how to create your own private Docker registry, check out [How To Set Up a Private Docker Registry on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-private-docker-registry-on-ubuntu-18-04).

To push your image, first log into Docker Hub.
```
$ docker login -u docker-registry-username
```
You’ll be prompted to authenticate using your Docker Hub password. If you specified the correct password, authentication should succeed.

```
Note: If your Docker registry username is different from the local username you used to create the image, you will have to tag your image with your registry username. For the example given in the last step, you would type:

$ docker tag sammy/ubuntu-nodejs docker-registry-username/ubuntu-nodejs

```

Then you may push your own image using:
```
$ docker push docker-registry-username/docker-image-name
```

To push the ```ubuntu-nodejs``` image to the **sammy** repository, the command would be:
```
$ docker push sammy/ubuntu-nodejs
```

The process may take some time to complete as it uploads the images, but when completed, the output will look like this:

```
Output
The push refers to a repository [docker.io/sammy/ubuntu-nodejs]
e3fbbfb44187: Pushed
5f70bf18a086: Pushed
a3b5c80a4eba: Pushed
7f18b442972b: Pushed
3ce512daaf78: Pushed
7aae4540b42d: Pushed

...
```

After pushing an image to a registry, it should be listed on your account’s dashboard, like that show in the image below.

![image](https://github.com/drivenbydesire/simple-docker-example/assets/9294879/805a7184-d4ec-45f9-897f-137493b53e0c)


If a push attempt results in an error of this sort, then you likely did not log in:

```
Output
The push refers to a repository [docker.io/sammy/ubuntu-nodejs]
e3fbbfb44187: Preparing
5f70bf18a086: Preparing
a3b5c80a4eba: Preparing
7f18b442972b: Preparing
3ce512daaf78: Preparing
7aae4540b42d: Waiting
unauthorized: authentication required
```

Log in with ```docker login``` and repeat the push attempt. Then verify that it exists on your Docker Hub repository page.

You can now use ```docker pull sammy/ubuntu-nodejs``` to pull the image to a new machine and use it to run a new container.
</details>

## Conclusion

In this tutorial you installed Docker, worked with images and containers, and pushed a modified image to Docker Hub. Now that you know the basics, explore the [other Docker tutorials](https://www.digitalocean.com/community/tags/docker?type=tutorials) in the DigitalOcean Community.


## Contributing

Thank you for considering contributing to the Simple-Docker-Example! I followed steps from [this article](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04) and modified some as of my own requirements to create this simple-docker-example.

Thanks to Digital Ocean Team for the article.

Earlier, I followed similar article for laradock but it didn't work due to php versions mismatch.
I will try my best to create this simple-docker-example to include as much as docker related actions in simplified form.

## License

The Simple-Docker-Example is open-sourced software licensed under the [MIT license](https://opensource.org/licenses/MIT).
