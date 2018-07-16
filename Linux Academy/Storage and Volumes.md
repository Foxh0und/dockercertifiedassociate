# Storage and Volumes

## State Which Graph Driver Should Be Used on Which OS
If you use are using containers the way they're intended, you will be storing little to no data in a container's writeable layer. Docker volumes are the more appropriate destination so that the container remains portable. However, there are some cases when writing to the container storage layer is a necessity and you will need to choose an appropriate storage driver that is supported on your implementation platform. 

1. Always choose from the prioritized list published by docker for your platform.
2. EE/CE affects what choices are available.
3. Some Storage Drivers are not available for all file systems. 

Overlay2 is recommended for CE.

Each driver has performance characteristics for each workloads.

- Aufs, Overlay, Overlay2: Operates at the file level, is more effecient in memory utilization but container layer can grow quickly.
- devicemapper, btrfs,zfs: Operates at block level. Allows better formance in heavy write workloads at the expense of more memory.

Overlay performs better than overlay2 with containers with many layers or deep filesystem. 

Remember you need to reimport images (And containers) if you change the Storage Driver.

## Summarize How an Image Is Composed of Multiple Layers on the Filesystem
A Docker image is built up from a series of layers, each representing a single instruction in the image's Dockerfile. Every layer except for the last, is read only.

Remember, some commands of the same type (eg: Run) can be run as a single instruction, and thus create only another layer.

The Storage Driver handles the details about how image layers interact. 

With each container having its own writeable container layer, each image can maintain a 1-N ratio of image storage to container storage. 

As a result of Docker's storage strategy of 'Copy-On-Write', files and directories that exist in lower layers that are not needed in higher layers can be provided read access to them to avoid duplication. If that a file needs to be modified, only then is it copied to the higher layer where the changes are stored. 

Anytime a container is deleted, any data that is written to the containers writeable layer that is not stored in a data volume will be deleted along with the container. 

Data Volumes represent a file/directory on the host filesystem in the /var/lib/docker directory. They are not controlled by the storage driver, and allow containers to remain portable.

## How Storage and Volumes Can Be Used Across Cluster Nodes for Persistent Storage
We use docker volume create to create a new docker volume. By default that will be /var/lib/docker/volumes/<name of new mountpoint>

eg docker volume create myNewMount

Volumes can be mounted to your container instances from your underlying host systems. In the case of clusters, this can be limiting because there is no built-in mechanism for your swarm to share a single filesystem/docker volume. 

However, you can use --mount to add a mount point. However the workers and other managers won't have access to the mount, as that mount won't exist on their system.

Azure and AWS allow for cloud store options to share volumes. 

## Identify the Steps You Would Take to Clean Up Unused Images (and Other Resources) On a File System (CLI)
When dealing with images (and other resources) on your Docker systems, outside of DTR, you can use the 'docker prune' command to control resources that are not attached to Docker objects and whether they get to 'hang around' or not. 

The system prune command allows you prune images. The flags --all/-a removes all, and -f forces the operation. You can add a bunch of other things, eg, add --volumes to delete all unused volumes. 

Using purely docker system prune removes all stopped containers, all networks used by at least one container, all dangling images, and all build caches. 