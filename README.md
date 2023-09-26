# PiQTree2 Development 

The easiest way to develop for PiQTree2 is to build in a docker container, which provides an isolated dedicated linux environment hosted by your host OS (Windows, MacOS, or Linux).

## PiQTree2 Docker Container

### Setting up the docker engine

The docker engine is a hypervisor application that hosts docker containers. On Windows and MacOS, you can install Docker Desktop application from [here](https://www.docker.com/products/docker-desktop) 

Alternatively, on macOS, you can use Colima which is a lightweight Docker-compatible environment on macOS without the need for a hypervisor.

<details><summary>Installing Colima on macOS</summary>

1. Install Colima using Homebrew:

    ```sh
    brew install colima
    ```

2. Initialize Colima with the Docker runtime:

    ```sh
    colima start --runtime docker
    ```

3. Verify that Colima is running:

    ```sh
    colima status
    ```

Now you can use Docker commands as you normally would, and Colima will handle the container runtime on macOS.

</details>

On Linux, you will need to install the docker engine. 

Instructions for installing for your specific distribution of linux are available [here](https://docs.docker.com/engine/install/)

### Configuring the PiQTree2 docker container

The container will mount a local directory that you have clones the PiQTree2 repository into on your host machine, to a working directory inside the container.

This allows you to edit files on your host machine using your favorite editor, and build and run the project inside the container.  The container will also mount your SSH private key into the container if you have one set up on your host machine.  This allows you to push and pull from your fork of the PiQTtree2 repository from inside the container.

<details><summary><strong>PiQTree2 Docker Image Contents</strong></summary>
This container downloads and installs the following dependencies:

- Eigen library: Used for linear algebra.
- TERRAPHAST library: Used for computing the likelihood of a tree.
- Boost libraries: Used for various utility functions and data structures.
- zlib: Compression library.
- libbz2: Library for high-quality data compression.
- liblzma: Compression library.
- TBB (Threading Building Blocks): Used for parallel programming.
- Google Performance Tools: Contains TCMalloc, heap-checker, heap-profiler, and cpu-profiler.
- OpenMPI: Message Passing Interface library for parallel programming.
- libpll: Library for phylogenetic analysis.
- Clang: C language family frontend for LLVM.
- CMake: Cross-platform build system.
- Git: Distributed version control system.
- wget: Network utility to retrieve files from the web.
- plf_nanotimer: A high-resolution, cross-platform timer library for precise time measurement in nanoseconds.
- plf_colony: A container library optimized for frequent insertions and erasures while maintaining cache-friendliness and iterator stability.
- LSD2 (Least Squares Dating 2) is a phylogenetic dating library and tool that estimates divergence times and substitution rates on a phylogenetic tree. 

The container creates a working directory named PiQTtree2 in the root directory of the container.  The `entrypoint.sh` script is run on start up, which checks to see if you passed a named variable `SSH_PRIVATE_KEY` in when you built the container.  If you do, it copies the SSH private key into the container and sets the permissions on the file to 600.  This allows you to push and pull from your fork of the PiQTtree2 repository from inside the container.

</details>
<details open><summary><strong>Building the PiQTree2 Docker Image</strong></summary>

To build the Docker image clone this PiQTree2dev repository to a directory on your local machine (eg:/source/PiQTree2dev) so the docker build command can find teh `DockerFile` and `entrypoint.sh` script.  Then navigate to the root of your local clone of the PiQTtree2 repository and run the following command to build a docker image named `PiQTtree2dev` using the Dockerfile in this repository 

`docker build --tag PiQTtree2dev -f /source/PiQTree2dev/Dockerfile .`

*assuming that you cloned this PiQTree2dev repository into `/source/PiQTree2dev`*
</details>

### Using the container
<details><summary><strong>Running the container</strong></summary>
To start a Docker container using the image you just built and run PiQTtree2 interactively in a terminal session, run the following command:

`docker run -it --rm -v ${PWD}:/PiQTtree2 PiQTtree2dev`

This command does the following:

- `run`: Runs a command in a new container.
- `-v ${PWD}:/PiQTtree2`: Mounts the current directory on the host to `/PiQTtree2` in the container.
- `PiQTtree2dev`: The name of the image to use.

<details><summary><strong>Running the container in terminal mode</strong></summary>
    To run the container in terminal mode, add the following argument to the `docker run` command:

    `-it -rm`

    - `-it`: Allocates a pseudo-TTY connected to the container’s stdin and stdout.
    - `--rm`: Automatically removes the container when it exits.
</details>
<details><summary><strong>Running the container in detached mode</strong></summary>

Most developers will likely run a container persistently (in detached mode) across multiple terminal sessions. To run the container in detached mode, add the following argument to the `docker run` command:

`-d`

NB: If you have the flag -rm in your docker run command, the container will be removed when it exits.  This is not what you want if you want the container to persist.  So, if you are running the container in detached mode, you should remove the -rm flag from the docker run command.

You will need to find the container ID to subsequently attach to the container. To do this, run the following command:

`docker ps -all`

This will list all running/stopped containers.  Find the container ID for the container you want to attach to.  

If it is stopped, you will need to start it using the following command:

`docker start <container_id>`

Then, run the following command to attach to the container:

`docker attach <container_id>`

</details>

---
</details>
<details><summary><strong>Stopping the Docker Container</strong></summary>
In interactive mode simply enter `exit`.

In detached mode use the following command:

`docker stop <container_id>`

</details>
<details><summary><strong>Naming containers</strong></summary>

By default containers are given random names, like `flamboyant_badger`.  To explicitly name a container, add the following argument to the `docker run` command:

`--name <container_name>`

when referring to a container using docker command, you can use either the container ID or the container name.

</details>
<details><summary><strong>How to configure your SSH keys into the Docker container</strong></summary>

If you intend to contribute to a private fork of the iqtree2 repository, and you have an SSH private key set up on your host machine, and you have added your public SSH key to your GitHub account (https://github.com/settings/keys), then you can mount your SSH private key into the Docker container so that you can push and pull from your fork from inside the container. To do this, add the following argument to the `docker run` command:

### for macOS/Linux

`-v ~/.ssh/PRIVATE_KEY:/root/.ssh/id_rsa`

replace `PRIVATE_KEY` with the name of your private key file.

### for Windows (powershell)

`-v $env:USERPROFILE/.ssh/PRIVATE_KEY:/root/.ssh/id_rsa`

replace `PRIVATE_KEY` with the name of your private key file.

### Checking that you can authenticate with Github.com from inside the container

Run the following command from an interactive terminal session inside the container:

```sh
ssh -T git@github.com
```

This will prompt you to add github.com to your list of known hosts.  Type `yes` to add github.com to your list of known hosts.  You should see the following message:

`Hi USER_NAME! You've successfully authenticated, but GitHub does not provide shell access.` 

</details>
<details><summary><strong>Configuring the container for debugging using Xcode</strong></summary>

To configure the container for debugging using Xcode, add the following arguments to the `docker run` command:

`-p 1234:1234 -e DISPLAY=host.docker.internal:0`

- `-p 1234:1234`: Maps port 1234 on the host to port 1234 in the container to allow Xcode debuugging.
- `-e DISPLAY=host.docker.internal:0`: Specifies the X11 display to use will be on the host machine.

</details>
<details><summary><strong>Configuring the container for debugging using VScode</strong></summary>


To configure the container for debugging using VScode, add the following arguments to the `docker run` command:

`-p 3000:3000`

- `-p 3000:3000`: Maps port 3000 on the host to port 3000 in the container to allow VS Code debugging.

</details>
<details><summary><strong>Running a file in the container</strong></summary>
To run a file in the container, add the command and it's arguments to the end of the `docker run` command:

`docker run PiQTree2dev <command> <arguments>`

eg: To run the iqtree executable in the container:

`docker run PiQTree2dev iqtree -s example.phy`

or to run pytest in the \PiQTree2\test directory:

`docker run PiQTree2dev cd \PiQTree2\test && pytest`

</details>
<details><summary><strong>Running a file in the container</strong></summary>
## Sample docker run command

A docker run command for developing in detached mode from VS Code with the ability to check in code to a private fork of the iqtree2 repository on GitHub would look like this:

`docker run -it -d -v ${PWD}:/PiQTree2 -v $env:USERPROFILE/.ssh/github:/root/.ssh/id_rsa -p 3000:3000 --name PiQTree2dev PiQTree2dev /bin/bash`

</details>
<details><summary><strong>Inside the Docker Container</strong></summary>

Once inside the Docker container, you will be in the `/PiQTree2` directory where you can find the PiQTree2 project files. You can perform git operations, build the project, and run tests as you would in a regular development environment.

To build the project 

```sh
cd /PiQTree2
rm -rf build
mkdir -p build
cd build
cmake ..
make
```
</details>
<details><summary><strong>Cleaning Up</strong></summary>

To remove the Docker image you created, first find the image ID using:

`docker images`

Then, remove the image using:

`docker rmi <image_id>`

Replace `<IMAGE_ID>` with the ID of the image you want to remove.
</details>

## References

- [Docker Documentation](https://docs.docker.com/)
- [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)
- [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)