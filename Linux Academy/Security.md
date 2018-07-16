# Security

## Describe the Process of Signing an Image and Enable Docker Content Trust
All images are implicitly trusted by Docker however, you may want to only look at signing image. You sign images with Docker Content Trust.
If you set the environment variable (with export) "DOCKER_CONTENT_TRUST=1", it will try and sign each images when pushing to a repository.

Images are signed on the push process with the certificate on the registry.

## Demonstrate That an Image Passes a Security Scan
Docker Cloud and Docker Hub offer you the ability to scan your private repository images.

Running docker login without a repository, you will login to Docker Hub.

From here, you can go to the Docker Hub and scan your images.

## Identity Roles
WWithin the UCP Docker has two types of users.

- Admins
    Can make changes to the UCP swarm
- Regular Users
    - Have permissions that range from full control to no access

There are different types of roles user can have. A role is a permission

- None
- View Only
- Restricted Control
- Scheduler
    Only one with Node Permissions
- Full Control

You can create custom roles on top of this.

## Configure RBAC and Enable LDAP in UCP
Enabling LDAP for authentication allows you to integrate user management and authorization. 

You enable LDAP from the UCP control plane. It uses a typical LDAP server address.  

Roles can be applied to organisations with Grants. A Grant applies a role to orangisation or team. 

Remember, UCP rules on apply to clusters and containers controlled by that instance of UCP.

## Demonstrate Creation and Use of UCP Client Bundles and Protect the Docker Daemon With Certificates
UCP Client Bundles allow you to provide a preconfigured setup for user accounts set up in UCP, along with the necessary trusted certificates to connect to and use the UCP cluster. You can then control what that user can do by granting roles to the account in UCP.

A client bundle is a group of certificates downloadable directly from the Docker Universal Control Plane (UCP) user interface within the admin section for “My Profile”. This allows you to authorize a remote Docker engine to a specific user account managed in Docker EE, absorbing all associated RBAC controls in the process. You can now execute docker swarm commands from your remote machine that take effect on the remote cluster. 

Inside the client bundle comes with an env.sh script that will add the client bundle. It's .ps for Windows.

When you update the roles that user has, or change grants, you do not need to redownload the client bundle.

## Describe the Process to Use External Certificates with UCP and DTR
You shouldn't use self-signed certificates in production. You can enable certificates and change them in the admin panel in UCP. When changing certificates, you will need to update the client bundle.

## Describe Default Docker Swarm and Engine Security
Docker Engine security involves the considerationof four areas.

1. Host kernel support of namespaces and cgroups.
    - Namespace provides isolation to running containers so they cannot access the underlying host.
    - Control groups implement resource management to minimize effect on a host.
2. Limited attack surface of the Docker daemon.
    - The attack surface is affected by the fact that the daemon requires ROOT account privileges, so containers that may interact with this can make changes  to configuration. UCP, DTR, and Docker Content Trust can mitigate this.
3. Customization of container configuration profiles.
4. Hardening features of the kernel and their interaction with underlying containers.


## Describe MTLS
Mutual Authenticated TLS (MTLS) supporters Docker Swarm's goal of being secure from the start. 

Any time a swarm is initialized, a self-signed CA is generated and issues certificates to every node that registers within the Swarm. Using TLS with this allows for communication between nodes. The TLS transactinon is a two layer protocol with a record and a handshake. It provides authentication and security.
