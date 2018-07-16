# Networking 

## Create a Docker Bridge Network for a Developer to Use for Their Containers

### Docker Bridge
In general networking terms, a bridge network is a Link Layer device which forwards traffic between network segments.
IN terms of Docker, a bridge network uses a software bridge which allows containers on the same bridge to communicate, while providing isolation from containers that are not connected. 
Bridge Networks apply to containers running on the same Docker host. For communication between different docker hosts, routing is managed at the OS level or with an overlay network. By default, a default bridge network is automatically created, and all newly instantiated containers automatically connect to it.


Use the network ls command to look at our networks. 
We use the network command with create to create a new bridge network.
EG: docker network create --driver=bridge --subnet=192.168.1.0/24 --opt "com.docker.network.driver.mtu"="1501" testNetwork

To add a container that is already running to a network, use network connect.
EG, docker network connect -ip=192.168.1.10 testNetwork myContainer. You do not need to specify the IP. You cannot specify the IP address when connecting to the default network. 

When adding a network to a container, it doesn't remove it from it's other network. 
Use network disconnect to disconnect a container from a network. 
EG. docker network disconnect testNetwork myContainer

## Configure Docker for External DNS
By default DNS is passed through from the underlying host.

There are two ways to override this default setting.

1. When Instantiating the container
    Add the option when using the run command.
    EG docker run -d --name myTestContainer --dns=8.8.8.8 httpd
    This will instantiate the httpd image with the 8.8.8.8 for the DNS.
2. Change the default DNS Option
    You can change the default DNS so it doesn't pull the host's dns entry.
    This is done inside the daemon.json inside /etc/docker
    An empty daemon.json will look like this with modified dns.
        {
            "dns": ["8.8.8.8", "8.8.4.4"]
        }
    Please note you'll need to restart the docker daemon for these changes to take affect. 

## Publish a Port So That an Application Is Accessible Externally and Identify the Port and IP It Is On
We can expose ports when running our containers, with -p.
EG. 
docker run --name testWeb -p 80:80 httpd
This will map the left (the host) to the right.

If you use -P it will look at the image's metadata, and map all the used ports to a random port above 32768 on the host machine. 
When you run the docker ps command, you can see the ports and where they are mapped too

A simple way to get the IP address of a container is to run docker container inspect <container-name> | grep IPAddr

## Deploy a Service on a Docker Overlay Network
The overlay network is a network that allows containers within a swarm to communicate across it. 
Ingress is the default overlay network.

You can create an overlay network in the same fashion you create a bridge, albeit changing the driver option to overlay.
When you create an overlay network on the manager, it does not automatically populate to the worker nodes.
To do so, when creating a service, use the --network option to set the new network. After this first service creation (with replicas on that host), it will be referenced on that host.
On an overlay network, the underlying host has no access to the overlay network, including any IP addresses, however, inter container communication is available on it. 

To remove a service from a network, run docker service update --network-rm=<network-name> <service-name>

## Describe the Built In Network Drivers and Use Cases for Each and Detail the Difference Between Host and Ingress Network Port Publishing Mode
Drivers for your hosts and containers determine their behavior on each host as well as accessability and routing between them.

Network Driver Types
* Bridge
    - Simple to understand and troubleshoot. Is the default on stand-alone docker hosts.
    - Consists of private network that is internal to the host, all containers implemented on this host can communicate with one another.
    - External access is granted by port exposure of the container by the host, or static routes that are added with the host as the gateway for the network. 
* None
    - When containers need no networking.
    - Can only be accessed by the host they're running on.
    - Containers are attached directly. 
* Host
    - Only accessabile by the host.
    - Access to services can only be provided by exposing ports. 
* Overlay
    - Allows communication between all daemons in a swarm.
    - Extends itself by building previously non-existent networks on workers if needed to all daemons in the cluster.
    - Allows multiple communication of multiple services in a swarm that may have multiple replicas.
    - Default mode of Swarm communication
* Ingress
    - Special overlay network that load balances network traffic amongst a given service's working nodes
    - Maintains a list of IP's from nodes that participate in that services, and when a request comes in, routes to one of them for the indicated service.
    - Provides the routing mesh, allowing services to be exposed to the external network without having a replica on every node. 
* Docker Gateway Bridge
    - Special overlay network that allows overlay networks (including ingress) access to an individual docker daemon's physical network.
    - Every container run within a service is connected to the local Docker daemon's host network
    - Automatically created across nodes when a node joins


There are two types of port publishing modes
1. Host
    - Ports for containers are only available on the underlying host system and are not available for services outside of the host where that instance exists.
    - Used mainly in mainly multi-host environments where you need complete control over routing.
    - You are responsible for knowing where all instances are in the swarm at all times, and is controlled with mode=host in deployment.
2. Ingress
    - Since it is responsible for routing mesh, it makes all published ports available on all hosts participating in the cluster. 


##  Troubleshoot Container and Engine Logs to Understand Connectivity Issues Between Containers
You can look at these issues from three levels. System, Container, and Service.

1. System
    The system contains logs on the Docker daemon inside /var/log/messages (for RHEL based systems). Note, some things are written as Docker or docker.
    There is a lot of information on joining nodes to a swarm, both from managers, and workers themselves. 
2. Container
    We can get the logs of a container by running docker container logs <container-name>.
3. Service
    To see the service logs, run docker service logs <service-name> from the manager. It is the aggregate of all the individual container/hosts logs. 

## Understanding the Container Network Model
The container network model is built on three main components. 

1. Sandbox - encompasses the network stack configuration, including management of interfaces, routing, and DNS of 1-N endpoints on 1-N network on a container.
A sandbox contains the configuration of a container's network stack.
2. Endpoint - interfaces, switches, ports, etc and belong to only one network at a time
3. Network - a collection of endpoints that communication directly and can consist of 1-N endpoints.

The container network model is managed by Internet Protocol Address Management (IPAM).
On single host implementations, IPAM is not an issue, and routing is generally handled manually or through port exposure. Each network is specific to the host system.
However, on multiple host environments that may contain separate physical networks, providing routing to the underlying swarm networks is the "IPAM Problem".
Network drivers enable IPAM through DHCP or plugins so that complex implementations that would normally have overlapping IP address issues.

## Understand and Describe the Traffic Types that flow between the Docker Engine, Registry, and UCP components.

UCP Agents on worker nodes in a swarm communicate with the UCP manager on the swarm manager.