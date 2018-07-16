# Orchestration

## State the Difference Between Running a Container and Running a Service
Containers use the docker run (or create) command. The command is responsible for the container revolution, but times are changing.

Containers us give us flexibility, portability and a whole host of other advantages, but they're quickly becoming too limited.

We need an easier way to deploy complex configurations in highly available and easily scalable implementations. This requires the development of cluster management and control software (Swarm or Kubernetes)
to work directly with our containers. As a result, the new paradigm is needed to address the requirements of these clustered environments. 

The service command is the solution to managing containers deployed in cluster implementations. We can now think of a service as something that is consumed within the context of a larger application (which can include other Docker Services in its makeup).

Containers are limited to the single host they are started on, services are containers that live on a scalable number of 'workers' in a cluster of systems. Docker Swarm handles access to and the availability of, that service across the worker nodes, eliminating the challenges of routing and accessing individual containers. The scalability services allow you to take granular control of system resources. 

Containers 
- Encapsulate an application or function
- Run on Single Host
- Require manual steps to expose functionality outside of the host system (ports, network, and or volumes)
- Require more complex configuration to use multiple instances ( such as proxies)
- Not highly available
- Not easily scalable. 

Services
- Encapsulate an application or function
- Can run on 1 to N nodes at any time
- Functionality is easily accessed using features such as routing mesh outside the worker nodes. No need for complex network configuration
- Multiple instances are set to launch in a single command 
- Highly available clusters are available
- Easily scalable (both up and down)

## Demonstrate Steps to Lock (and Unlock) a Cluster
When you init a swarm, you can pass the --auto-lock command to set it up to auto lock on start/restart. It also passes back a key to unlock it. If you do not have this key or access to at least one other manager, then you will lose access to the node.

If you have already configured your swarm, you can run docker swarm update --autolock=true. Make sure you backup the key you receive. 

When you stop docker, upon a restart, you need to unlock the swarm with docker swarm unlock.

If you have the key, but you have access to an unlocked manager, you can use docker swarm unlock -key to get the key.

To get a new key run docker swarm unlock-key --rotate.
You should never overwrite old keys, and keep them backed up. This is because if you have many managers nodes, and you rotate your key, but there is a malfunction and the key doesn't get propagated to all the managers, you will lose access. Make sure you keep at least one key back at least for a few hours. 

## Extend the Instructions to Run Individual Containers into Running Services Under Swarm and Manipulate a Running Stack of Services
The command service ls lists services running in a swarm (assuming you are on a manager).
A replica is the number of threads you want to run on a given swarm. Default = 1.
A service uses mesh means you can access any container running in the swarm. (Even if that container is not running on that node). All nodes on a swarm can answer a service.

You can scale service on the fly up or down.

eg 
docker service update --replicas <new number of replicas> <service name>

There is currently an option called --detach = false, it will be the default in the future. This will show the update to us on our terminal, running the command in the foreground.

### Reserve vs Limit
There are two types or resource limit options, reservations and limits. 

Reserve holds those resources on the host so they are always available for the container. Think dedicated resources.
Limit prevents the binary inside the container from using more than that. Think of controlling runaway processes in container.

Reservations are hard limits, and must be higher than the soft limit.

You can use the--limit-cpu or --limit-memory to change the limits. They can also be fractions for cpu. EG. --reserve-cpu .75

If you change the limits for cpu or memory, they have to be applied to a new service. It will shutdown the service, and startup a new instance of that service.

## Increase and Decrease the Number of Replicas in a Service
We can run a service update passing in replicas with the new numbers up our down.
eg docker service update --replicas 3.

Docker service update requires one argument, so you can only update a single service with it.

However, docker service scale, allows us to scale (Change the number of replicas) for multiple services. 
EG
docker service scale testweb=2 testnginx=5

## Running Replicated vs. Global Services
Replicas mean that that service will be replicated to n nodes. 
Replicated mode means I control how many instances are running of that service across my node.

Global mode is the alternative. It means you want to run your service across all nodes. You will lose the granular control over your replicas.

When creating the service, you need to pass the --mode global flag.

When you join a node to a cluster that has a service in global mode, it will automatically deploy an instance(replica) to the new nod

You cannot change the mode of a service. You cannot run docker service --mode.

## Demonstrate the Usage of Templates with 'docker service create'
Templates can only be used for some flags related to service creation. Specifically, host names, mounts, and environment variables. 
This allows us to dynamically set things. 

EG, we can use the node ID and the Service Name to name the hostname of the docker container instance. 
docker service create --name web --hostname="{{.Node.ID}}-{{.Service.Name}} httpd

## Apply Node Labels for Task Placement
Node labels are a way to control how and where services run in your swarm.
The --pretty flag passed to inspect prints the inspect output in a easily readable format. 
You can update nodes with a label to designate services later on to run oun.

EG add a label to a node

docker node update --label-add mynodelabel=testnode <node id>

You can use the --contstraint to constrain a service to specific node.
eg

docker service create --name testService -p 80:80 --constraint 'node.labels.mynodelabel == testnode" --replicas 3
It will only deploy onto nodes that have the label mynodelabel and is equal to testnode 
There are many other constraints besides labels (node.labels) that can be used.

If there is only one (or any number less than the total number of nodes in the swarm) node, but your replica count is hire, it will replicate multiple times on a single node.

## Convert an Application Deployment into a Stack File Using a YAML Compose File with 'docker stack deploy'
Docker compose files are written in YAML. Docker Compose does not come with docker, and needs to be installed. Do not install it from the default repositories. Get it from  pip (python). Docker Compose 3 relies on some python libraries. At time of writing, 3 is the latest version.

### YAML
YAML (YAML Ain't Markup Language) is a human-readable data serialization language. It is commonly used for configuration files, but could be used in many applications where data is being stored (e.g. debugging output) or transmitted (e.g. document headers).

Using Compose is basically a three-step process:
1. Define your app's environment with a Dockerfile so it can be reproduced anywhere (create an image)
2. Define the services that make up your app in docker-compose.yml so they can run together in an isolated environment.
3. Run docker-compose up and Compose starts and runs your entire app.

Deploy a stack from a compose file on a SINGLE host: docker-compose up
Using docker-compose ps you can see the containers started with docker-compose. 
Use docker-compose down --volumes to stop the services started with docker compose. 

When we want to deploy to a swarm, we use docker stack deploy. EG
docker stack deploy --compose-file myfile.yml myCustomStackName

Docker stack deploy does not support dynamic building when deploying, so you cannot use the build command inside the compose yaml file. 

Each service from your compose file is a separate service, and can be managed separately. 

## Understanding the 'docker inspect' Output
Whilst you can just pass the ID of something you want to inspect, you can also use docker container inspect <id> or docker image inspect <id> etc.

## Identify the Steps Needed to Troubleshoot a Service Not Deploying
There are several steps and commands to run when a service will not deploy.

- Docker node ls: Make sure all expected nodes are connected to the swarm
- Docker service ps [service name]: Is the service partially running? Is the problem specific to the number of replicas?
- Docker service inspect [service name]: Did you deploy with a constraint but have mismatched the values. 
- Docker ps: Is your cluster locked?
- Did you restore a swarm?: Make sure you re-initialize the new manager (docker swarm innit -force-new-cluster)
- Have you recently updated Docker? Make sure you are using the official Docker repositories. 

SELinux can cause Docker issues. Try putting SELinux into passive mode to test issues that you think may be caused by SELinux.

Make sure your User/Group ID have access to resources your service may need. 

Make sure the host has the necessary resource requirements. 
Be sure routing, network, and firewall is configured correctly. 

## How Dockerized Apps Communicate with Legacy Systems
Side by side, the architecture of a Docker Containerized application externally does not differ from what you might encounter in traditional environments, however, there are a couple of caveats to keep in mind so all your applications know how to find one another.

A few items to consider with Docker apps when putting them into your IT Ecosystem.
- Routing: You need to expose your application/service with port redirection or provide a static routing mechanism to the container network on their hosts. 
- Port Redirection: As above, this will affect how they behave in your environment. What if you have multiple ports that operate on port 80?
- Portability: Making sure that data is external, and will not affect the performance to the container.

Containers Should Be
- Abstract: You containerize an application so it remains removed from the other components in the stack.
- Portable: Containers can be picked up and put anywhere will be consistent in their content and behavior. However, re-establishing communications and data can be tricky.
- Flexible: They must provide almost limitless flexibility. They shouldn't be tied closely to external and hardcoded variables. 

## Paraphrase the Importance of Quorum in a Swarm Cluster
Every Swarm will have 1 to N manager nodes. They're responsible for managing directing, logging and reporting on the swarms lifecycle. You need to have multiple manager nodes if you desire high availability.

### Raft Consensus Algorithm
Is used to managed the swarm state. Using this method amongst manager nodes is designed to ensure that in the failure of any manager, any other manager will have enough information to continue the operation of the swarm.
Raft tolerates up N-1/2 failures and requires a majority (quorum) of N/2 + 1 to agree on any new instructions that are proposed to the cluster for execution.

For example, a Swarm of containing two managers has no fault tolerance, as if one manager goes down, then the quorum size is 1, so 50%, and therefore not a majority. 

Be careful when adding more managers, as communication between multiple managers can impact performance.

Requirements/Considerations
- Use Static IPs
- Immediately replace failed managers
- Distribute Management Nodes for High Availability
- Monitor Swarm health.
- Have a backup and recovery plan


When distributing nodes, use the rule of thirds to maintain a quorum incase an entire location down. Add manager nodes increasingly. Do not add them all in one location

Requirements and Considerations for Management Node Datacentre Distributions
- Use a minimum of 3 availability zones
- Run Manager only nodes
- Force Rebalance (docker service update --force)


docker node update ---availability drain [node] will drain all services from a node, and mark it as unavailable. It will only function as a manager.