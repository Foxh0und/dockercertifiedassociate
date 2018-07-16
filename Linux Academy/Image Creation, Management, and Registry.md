# Image Creation, Management, and Registry

## Pull an Image from a Registry (Using Docker Pull and Docker Images)
The DockerHub is the default repository for Docker CE. 
With Enterprise Edition, you can use UCP to change the default repository to a custom one.
To pull an image use docker pull.

Each image has a name and a tag, which designate version. 
Use -a to pull all versions of an image.
If you wish to pull a specific tag, use this format : image:tag.
Use --disable-content-trust to ignore repository checking.

The command images lists images on the disk.
Images that use the v2 or later format have a content-addressable identifier called a digest. As long as the input used to generate the image is unchanged, the digest value is predictable. To list image digest values, use the --digests flag.

We can search through our images with filtering, and can filter at labels in images.
We can filter on the CREATED field using before or after.

We can also do them after events. EG"

docker images --filter "before=centos:6" returns all images we downloaded before we pulled centos:6.

The quiet flag just shows the image id's. This is useful for feeding ID's one at a time into another command.

## Searching an Image Repository
You can search a repository if you do not know the exact image name.
You use the search command, eg docker search apache. 
The search command returns a few fields, including stars (similar to github).
Filter is available in search.
There are also official and automated fields.
Automated images get updated automatically.
The --limit flag limits the top n results. 

## Tag an Image
You can create tags to build upon an image whilst leaving the original image alone.
We use docker tag.
EG docker tag centos:6 mycentos:v1
Creates a new tagged image based on centos:6 named mycentos:v1. 
It will have the same Image ID, as it hasn't been changed yet.

Use rmi to remove an image. If I didn't change mycentos:v1, and ran docker rmi centos:6, centos:6 would be untagged, but nothing would be deleted, as the same filesystem still points to mycentos:v1. 

The tag is just a reference to the file system. If you remove a tag, it won't remove the filesystem unless it is the last one.

## Use CLI Commands to Manage Images (List, Delete, Prune, RMI, etc)
The image command typically allows us to list one image at a time.

Images just lists all images(all tags for duplicate IDs) on the system.

Use the history argument with image to see the history of an image, eg docker image history myrepo:2. It only shows us what events happened on the system.

Use image save to save an image. Eg, docker image save myrepo:2 > myrepo.tar.
It will contain all data and metadata required to launch the image. You can use the import command to import the tar.
EG docker import myrepo.tar localimport:newimage name.

Load, which redirects too and from a stream. EG docker load --input myrepo.tar.

Docker load and import are not the same commands, and they handle things differently. 

The command rmi is the same as docker image remove.

Prune removes all dangling images.

### Dangling Images


## Inspect Images and Report Specific Attributes Using Filter and Format
Inspect gives a lot of information about our images. 
The default output is formatted in JSON.

We can use the format command to get to things inside the output json.

EG to get the hostname inside the container config:
docker image inspect centos:6 --format '{{.ContainerConfig.HostName}}.

EG to get everything inside Container Config, not just the hostname:
docker image inspect centos:6 --format '{{json .ContainerConfig}}.


## Container Basics - Running, Attaching to, and Executing Commands in Containers
Not exactly on the exam, but more like assume knowledge.

When you run something on an image it instantiates a container on image. 
When we make changes to that container it modifies it's own filesystem, not the base image's underlying storage that it inherits from. This is why you cannot delete from it.

The ps command lists running containers.
The run command is the simplest way of instantiating an image.

Useful flags:
- i: Interactive mode, be attached to the standard input.
- t: Terminal, attach it to my current terminal.

Unless specify a name (use --name), it will assign one for your container automatically.

You can execute commands that you would like it to start up with.
Use the restart command to restart a container that is shutdown.
Use the rm command to remove containers. 

You can join commands, eg, pass all the container IDs (obtained with the quiet flag) to the rm command
docker rm `docker ps -a -q`

Temporary containers are known as nebulas containers. You can specify this when instantiating a container with the -rm flag, which removes the container when it is stopped.

You can pass environment variables with --env variableName = variableValue.

When we want a container to effectively run in background, we want it to be detached. To do this we pass the d flag.
This will launch it in detached mode.

To connect/attach to a running container use the exec command. EG If you have a container named httpd:

docker exec -it httpd /bin/bash

This states you want to connect in interactive mode with your current terminal and start by executing the /bin/bash command.

The attach command is not as versatile. It will attach you to the exact command and instance, and if you exit it will close the container. Exec allows us to execute a new command.

## Create an Image with Dockerfile
When building an image, it assumes a docker file is called 'Dockerfile' (case sensitive).
Hash for comments.

We use the build command to build an image,

eg docker build -t customimage:v1 .

The . refers to the current directory

If you rebuild an image (run build again) from a dockerfile it will use a cache if the steps have not changed in any form or fashion.
If you change the steps, it will only build that one again, the other unchanged steps will use the cache. Use --no-cache to avoid this.

When pulling, if it does not have the image in the local repository, it will try and pull from the configured online repository. 

You can build straight from a git repository by pointing it towards the dockerfile.

FYI, squash is only on experimental features. It allows you to build images as a single layer.

## Dockerfile Options, Structure, and Efficiencies
DockerFiles have many instructions. Each instruction creates in intermediate layer.

- FROM: image to build from (Scratch for empty image)
- LABEL: Any Labels you would like
- RUN: Commands to run when building the image. Executes command in a brand new layer. Can be in JSON form, but command form is preferred. 
- MAINTAINER: The name of the maintainer. Is now deprecated, and MAINTAINER should be a label.
- CMD: Default command run on image instantiated. Only one can exist in a DockerFile, and if multiple are specified; only the last one will take effect.
- ENV: Environment variables that the container will have.
- EXPOSE: Exposes ports. Does not publish port, but really serves as documentation. 
- ENTRYPOINT: Entry point for the container. This cannot be overwritten. Makes a container executable 
- COPY: Takes files from the local context/path you indicate, and put them inside the image (where you specify)
- VOLUME: Create a mount that containers will have available to it. There is no way to tie a container to mount point in the host system mount. This is so images remain portable.
- ADD: Similar to COPY, but also supports URLs.
- ARG: If it precedes the FROM command, it is only passed to the FROM command.
- STOPSIGNAL: Special signal when a container is stopped
- SHELL: Override default shell (/bin/sh) with another.
From an efficiency standpoint, you want to combine as many commands(of the same time) on a single line in a DockerFile. You can do this with a "\" at the end of lines. 

You can see all the layers with the history command. EG docker history <image>. For the exact count, pipe this to wc -l.

## Describe and Display How Image Layers Work
The Dockerfile determines the number of intermediate layers of an image.
The history command shows the history of an image. It shows the command that gets executed that build up each layer of that image.
If the IMAGE ID in history is missing, it means that image came from somewhere else.

Docker uses a union file system, which means files and directories of separate file systems or branches be overlaid so they form a single filesystem.
Each image layer has it's own information in each one. 
WE can also see the size of each layer. The aggregate size of each layer should be the size of the image

## Modify an Image to a Single Layer
There is no way officially to flatten or squash layers into a single image, though Docker Squash is graining traction, but is still experimental.
You can export containers and reimport it as an image. (Container > Tar File > Image). This can often save space.
Now, we cannot see the other layers, as it's from the one container. We have lost all the intermediary images The only layer will be the import command. 
If you check your images into git repository, history can also be seen here.

## Deploy, Configure, Log Into, Push, and Pull an Image in a Registry
You need at minimum two environment variables when running a registry that is both secure and you can authenticate against.
- REGISTRY_HTTP_TLS_CERTIFICATE
- REGISTRY_HTTP_TLS_KEY

You can push to a repository with the push command. 
eg
docker push reponame/httpd:latest

However, you may need to use authentication, and login to the registry. This is done before a push. EG
docker login localhost:8000

Use the pull command to pull an image from a registry. When using a custom registry, you have to specify it. As before, you may have to authenticate the repo by logging into it.
eg docker pull mycustomrepo.com/httpd

## Selecting a Docker Storage Driver
If you write image inside a container, it makes it less portable. 
Docker uses a pluggable architecture, meaning many different types of storage drivers are useable. We looked at this in depth in the introduction. refer to those notes.

To look at storage driver information, use the info command and grep for "Storage".
Overlay is generally the default. 
We can specify the storage driver in the configuration file (/etc/docker/daemon.json).

Remember, if you change the storage driver, you need to export and reimport your images. Images won't be available if you change, even if you change back.

Inside /var/lib/docker there will be a folder for your storage driver. Eg devicemapper, or overlay.

## Prepare for a Docker Secure Registry
A docker registry is like the docker hub. Before you use a private repo, you need to secure and it set up authentication.

Docker Registry uses port 5000 on the underlying host by default.

You need to make a directory with the domain of your private repo.
EG: /etc/docker/certs.d/myregistrydomain.com:5000
Insider this directory is where you need to place your certificates. They must be owned by root.

There is a special docker container named registry that is used to create basic user authentication. 

You want to output the results of the run command into the auth directory that was created, or to a file.

docker run --entrypoint htpasswd registry:2 -Bbn test password > auth/htpasswd

## Deploy, Configure, Log Into, Push, and Pull an Image in a Registry

## Managing Images in Your Private Repository

NEED TO DO THESE ABOVE TOO.

## Container Lifecycles - Setting the Restart Policies
You can set restart policies to containers.

There is no automatic container restart of the container by default. 

You can specify the restart policy when instantiating one. EG

docker container run -d -name myname --restart <policy state>

There are several states
- on-failure: On a container failure
- none: Will not restart
- always: Will always restart
- unless-stopped: 

If you stop a container manually, the container will not restart unless started or the docker service restarts. This is because sometimes you may want to stop a container.

If unless-stopped is selected, it will ignore the docker service restart, and will not start unless manually started.