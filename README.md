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
</details>









## Contributing

Thank you for considering contributing to the Laravel framework! The contribution guide can be found in the [Laravel documentation](https://laravel.com/docs/contributions).

## Code of Conduct

In order to ensure that the Laravel community is welcoming to all, please review and abide by the [Code of Conduct](https://laravel.com/docs/contributions#code-of-conduct).

## Security Vulnerabilities

If you discover a security vulnerability within Laravel, please send an e-mail to Taylor Otwell via [taylor@laravel.com](mailto:taylor@laravel.com). All security vulnerabilities will be promptly addressed.

## License

The Laravel framework is open-sourced software licensed under the [MIT license](https://opensource.org/licenses/MIT).
