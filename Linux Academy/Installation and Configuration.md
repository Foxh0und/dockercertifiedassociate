# Installation and Configuration

## Complete Docker Installation on Multiple Platforms (CentOS/Red Hat)
You need to know how to install Docker for the exam.

Several packages are required
- device-mapper-persistent-data
- lvm2

You do not want to install Docker from the central repositories, the Docker repo is required.
The repo is: https://download.docker.com/linux/centos/docker-ce.repo
Naturally once this is done we need to update, and then install (docker-ce).

You need to use docker-ce or -ee, as it will be the latest version.
With SE linux, it can take some time to apply all the required changes and policy.

It is a service, and as such, we need to use systemctl to enable and start it.

When you install it, docker.sock (var/run/docker.sock) is owned by root. Changing ownership and permissions,
allows any user to do it. Or you can add it to the newly created docker group.
  usermod -aG docker <username>

## Selecting a Storage Driver
Normally Docker volumes are used to write data.
Docker uses a pluggable architecture that supports multiple storage drivers that control how images and containers are stored and controlled on the host.

On CentOS and RHEL, device mapper is used is the storage driver.
It can be used as a block storage on the disk, or as a brand new disk.

### Storage Drivers
Ideally, very little data is written to a containers writeable layer. However some applications require you
write to the container. This is where storage drivers come in. The files won’t be persisted after the container stops, and both read and write speeds are low. The data can also be difficult to extract from the container, and is heavily coupled with the host machine and is difficult to move elsewhere.

Docker has two options for containers to store files in the host machine, so the files are persistent events on
container stop. They are volumes and bind mounts.

The info command is very useful, and we can grep this to find the storage driver currently employed.

The default is often the overlay driver. On EE, device-mapper is only supported by Docker for CentOS and RHEL, but we suspect overlay will be supported in new incarnations. Overlay is only supported in Fedora.

For Docker, there is a configuration file in /etc/docker/ named 'key.json'. It maps to other configuration files.

If the storage driver is changed, any docker images need to be exported and reimported.

Most of the docker files are in /var/lib/docker.

### Images vs Containers
An instance of an image is called a container. We start with images, and if started, we have a running container of this image. We can have many running containers of the same image.

## Configuring Logging Drivers (Syslog, JSON-File, etc.)
Setting up logging drivers is similar to setting up Storage Drivers.
By default it uses the JSON driver for logging.

There are several third party logging drivers, such as AWS, GCP (google), and Splunk.

Similarly to how we were able to search for the storage drivers with the info command and grep, we can search for the logging driver.

The logs command followed by the name of the container allows us to view our log files for that particular container.

To modify the logging driver, an entry needs to be added to daemon.json.
Options also needed to be added.
Example: Using Syslog as the Logging Driver.
  {
    "log-driver":"syslog",
    "log-opts": {
      "syslog-address":"udp://172.31.125.216:514"
    }
  }

This will change the global option and define syslog as the logging driver.

When we launch a container, with the flag --log-driver we can specify the logging driver on a container bases when we run it. For example:
docker container run -d --name example --log-driver json-file httpd

## Setting up Swarm (Configure Managers)
A Swarm contains one or more nodes, and one or more managers.

To start up a Swarm, we need to run swarm init, and specify where you would like to "advertise" it from, which is usually the IP address.

You'll receive a message with a command that contains a special token. This command needs to be used to add workers to the Swarm. If this token is lost, use docker swarm join-token <worker/manager>.


## Setting Up Swarm (Add Nodes)
To join a Swarm as a worker, we use the command that we received from our manager.
docker swarm join --token

Remember, if we have lost the token, we can get it again on our Manager machine with
docker swarm join join-token worker

Unlike on managers, the node ls command returns nothing, as only managers have knowledge of the overall Swarm. Workers are only passive receivers of instructions.

## Setting Up a Swarm (Backup and Restore)

### Service
A service is a group of containers of the same image. Services make it simpler to scale applications. Docker starts services straight away.


Inside /var/lib/docker is a folder called swarm.

There are several files, including the state.json and certificates. When backing up a swarm these are all required to be backed up.
We can then use this to overwrite another computers swarm directory in /var/lib/docker to restore it.

If we are restoring on a new manager, we need to run docker swarm join --force-new-cluster.
If you notice closely, the join tokens are exactly the same as the machine we backed up from.

Nothing else besides the swarm folder is required, no images are required on the restoration targets.

## Outline the Sizing Requirements Prior to Installation
Sizing your environment for a Swarm Cluster is going to be generally the same you would for any applications.

- CPU, Memory and Disk
  Your containerized application will have the same requirements if it were installing it on any other infrastructure, and you have to be sure your underlying host(s) have the required specifications.
- Concurrency
  What are the load requirements of the application at peak and in total? These determine the optimal placement and the hardware resources you need to allocate.


### Universal Control Plane (UCP)
This is Docker's cluster management system, developed by Docker. It allows users to manage everything associated with the Docker swarm cluster from a web console.

UCP Minimum Requirements
- 8GB RAM (Managers or DTR Node)
- 4GB (Workers)
- 3GB free disk space
UCP Recommended Requirements
- 16GB Ram (Managers or DTR Nodes)  
- 4vCPU's (Workers or DTR Nodes)
- 25GB - 100GB Space

There are other timeout specifications that are not configurable, such as Raft consensus between manager nodes, and Gossip protocol.


Recommendations
- Plan for Load Balancing
- Use External Certificate Authority for Production (for protecting UCP) so no internal certificate errors and warnings.



## Complete Backups for UCP and DTR
To backup UCP, we want to run a container of docker/ucp and back it up to a tar file.

We run a similar command, with docker run [options] docker/dtr backup. You will also need to pass the UCP server URL and the UCP password.

During a backup, DTR will not be available.

## Managing Images in Your Private Repository
You can use the provided REST API to query a registry.
Simply, you can use the curl command to inspect it. It responds with JSON

EG, to look at a catalogue of the images on the registry, run the following:

curl --insecure -u "user:password" https://mydomain.com:5000/v2/_catalogue

Use insecure mode if using a self signed certificate.

If you use WGET, you can download the response as a file (still in JSON format).

You can use the API to do many things, such as looking at tags, images themselves, you can even delete images.

DTR is different, as there is a web console.

## Create and Manage UCP Users and Teams
You can create and manage users within UCP, it provides a very friendly user interface to manage many things across your Docker stack. 


## Namespaces and CGroups
Namespaces provide isolation so that other pieces of the system remain unaffected by whatever is within that namespace. Docker uses namespaces of various kinds to provide the isolation that containers need in order to remain portable and refrain from affecting the remainder of the host system.

Namespace Types
- Process ID
What allows the container to encapsulate everything within the one process ID.
- Mount
Mounts to a lesser extend can be namespaced and isolated from the host system. Can be used as a volume mount tied to the host system, but is not a traditional system mount.
- Interproccess Communication (IPC)
Allows containers and services in a swarm to communicate with each other without affecting the host system
- User (currently experimental)
The ability to remap users from a container to the underlying host. At current stage, it can break the mount and process ID namespaces, as it requires permissions.
- Network
Defines how networking works.

### Control Groups (cgroups)
Control Groups provide resource limitation and reporting capability within the container space. They allow granular control over what host resources are allocated to the containers and when they are allocated.

Common Control CGroups
- CPU
- Memory
- Network Bandwidth
- Disk
- Priority

Whereas namespaces provide the isolation, cgroups allow you to allocate resources to said container.

You can allocate resources based on reservation or limit.
