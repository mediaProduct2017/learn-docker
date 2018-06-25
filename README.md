# learn-docker

[docker usage guide -- Nvidia](https://www.nvidia.co.kr/content/apac/event/kr/deep-learning-day-2017/dli-1/Docker-User-Guide-17-08_v1_NOV01_Joshpark.pdf)

[A Primer on Nvidia-Docker — Where Containers Meet GPUs](https://thenewstack.io/primer-nvidia-docker-containers-meet-gpus/)

[Usage - nvidia-docker](https://github.com/NVIDIA/nvidia-docker/wiki/Usage)

    docker version
    nvidia-docker version

docker images and containers: images are installed docker programs which can be activated and become contrainers by docker command, containers are running progresses or runned progresses.

    docker info
    docker images # for images
    
    docker ps # LIST OF CONTAINERS, for running containers
    # Options
    # -a: all (including Exited)
    # -f name=[]: name filter
    
    docker ps -a # for containers including exited containers
    
    docker stats
    # RESOURCE UTILIZATION MONITORING
    # to quit docker stats, you have to use ctrl+c
    
CONTAINER CREATION from docker images

    docker run [OPTIONS] IMAGE[:TAG] [COMMAND] [ARG...]
    
NVIDIA runtime

nvidia-docker registers a new container runtime to the Docker daemon.
You must select the nvidia runtime when using docker run:

    docker run --runtime=nvidia --rm nvidia/cuda nvidia-smi
    docker run --runtime=nvidia --rm nvidia/cuda nvcc --version
    # nvidia/cuda is the image
    
    nvidia-docker run --rm nvidia/caffe nvidia-smi
    # use once when container create initially
    
    Options:
    --rm: Automated container remove when it exits # it never shows in docker ps -a 
    -v: volume mount
    -u: set user UID
    
    INTERACTIVE TERMINAL OPTION:
    -t option
    Enables shell can show container default shell’s stdout
    -i option
    enables shell input key put to the launched container default shell
    -ti or -it option
    enables interactive terminal to the container default shell
    
Enables GPU selection (with NV_GPU option)    
    
    NV_GPU=1,3 nvidia-docker run --rm nvidia/caffe nvidia-smi
    
container remove w/ or w/o force option

    docker rm {container-name}
    docker rm –f {container-name}

Container naming

    docker run --name cuda -v $(pwd):/workspace -v /mnt/vol:/data image-name
    
User forwarding

    docker run -u $(id -u):$(id -g) image-name
    
MOUNT FOR HOST RESOURCES

Volume mount:

    -v {host-volume}:{container-volume}[:ro]
    
Port forwarding:

     -p {host-port}:{container-port}
     
DOCKER PULL

Loading docker image to the host

    docker pull ubuntu
    
EMBEDED IMAGE PULLING: Retrieves docker images registry when no image found on the host

GETTING ADDITIONAL DOCKER IMAGE

From docker hub

    docker pull [image name][:tag]
    
From NVIDIA DGX registry

    docker pull nvcr.io/nvidia/[framework]:[tag]
    
From local registry

    docker pull [local-registry addr]/[group name]/[image name][:tag]
    
From file

    docker load --input {file-name}.tar.bz2
    
BACKUP DOCKER IMAGE

To docker hub

    docker push [docker hub id]/[image name][:tag]
    
To NVIDIA DGX registry

    docker push nvcr.io/[group name]/[image name][:tag]
    
To local registry

    docker push [local-registry]/[group name]/[image name][:tag]
    
To file

    docker save [image name] | bzip2 > {file-name}.tar.bz2
    
DOCKER IMAGE REMOVE

    docker rmi [image-name][:tag
    
CONTAINER BACKUP

    docker commit [container-name] [image-name][:tag]
    
Container will loose all works when exit    

Solution
1. Code & dataset
   USE volume mount to host volume
2. Environment work
   docker commit

BUILDING NEW IMAGE

Docker image build with script description

DOCKERFILE

docker image build script file

Any image can be customized from base image

Use provided CUDA docker image to build custom GPU accelerated image

WRITING DOCKERFILE

Description of building development environment

DOCKER IMAGE CREATION

     docker build -t IMAGE[:TAG] –f Dockerfile {dockerfile path}
     
SOME USERFUL CLEANUP COMMANDS:

Docker volume clean up

    docker volume rm $(docker volume ls -qf dangling=true)
    
Docker image clean up

    docker rmi -f $(docker images -q)
    
Docker images clean up which name is \<none\>

    docker rmi -f $(docker images | grep "<none>" | awk "{print \$3}") 
    
Docker container clean up which is Exited

    docker rm $(docker ps -a -f status=exited)
    
Ubuntu 14.04/16.04/18.04, Debian Jessie/Stretch

    # If you have nvidia-docker 1.0 installed: we need to remove it and all existing GPU containers
    docker volume ls -q -f driver=nvidia-docker | xargs -r -I{} -n1 docker ps -q -a -f volume={} | xargs -r docker rm -f
    sudo apt-get purge -y nvidia-docker

    # Add the package repositories
    curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
      sudo apt-key add -
    distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
    curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
      sudo tee /etc/apt/sources.list.d/nvidia-docker.list
    sudo apt-get update

    # Install nvidia-docker2 and reload the Docker daemon configuration
    sudo apt-get install -y nvidia-docker2
    sudo pkill -SIGHUP dockerd

    # Test nvidia-smi with the latest official CUDA image
    docker run --runtime=nvidia --rm nvidia/cuda nvidia-smi
    
    docker run --runtime=nvidia --rm nvidia/cuda nvcc --version
    
    