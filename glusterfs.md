# what is GlusterFS?
- GlusterFS (Gluster File System) is an open source distributed file system that can scale out in building-block fashion to store multiple petabytes of data. ... The vendor supports a commercial version of GlusterFS, which ships with its Red Hat Gluster Storage product.

- GlusterFS is a free, open source and scalable network filesystem specially designed for data-intensive tasks such as cloud storage and media streaming. GlusterFS made up of two components a server and a client. The server runs glusterfsd and the client used to mount the exported filesystem. You can achieve high availability by distributing the data across the multiple volumes/nodes using GlusterFS. GlusterFS client can access the storage like local storage. GlusterFS is a file-based scale-out storage that allows you to combine large numbers of commodity storage and compute resources into a high performance and virtualized pool. You can scale both capacity and performance on demand from terabytes to petabytes.

# Features
1. Global namespace and Clustered storage.
2. Modular and stackable.
3. Highly available storage.
4. Built-in replication and geo-replication.
5. Self-healing and ability to re-balance data.
6. Software only, runs on commodity hardware.
7. Multi-brick Block Device volumes and Quota Scalability.

# Few helping links:
1. https://www.youtube.com/watch?v=OI153oBeiCI
2. https://www.scaleway.com/en/docs/how-to-configure-storage-with-glusterfs-on-ubuntu/
3. Configure a High-Availability Storage with GlusterFS on Ubuntu 18.04 LTS:
    https://www.scaleway.com/en/docs/how-to-configure-storage-with-glusterfs-on-ubuntu/ 
4. Install and Setup GlusterFS on Ubuntu 18.04:
    https://kifarunix.com/install-and-setup-glusterfs-on-ubuntu-18-04/
5. High-Availability Storage with GlusterFS on Ubuntu 18.04 LTS:
    https://www.howtoforge.com/tutorial/high-availability-storage-with-glusterfs-on-ubuntu-1804/

6. Installing GlusterFS - a Quick Start Guide: 
   https://docs.gluster.org/en/latest/Quick-Start-Guide/Quickstart/
7. GlusterFS Documentation:
   https://docs.gluster.org/en/latest/
8. https://www.gluster.org/



# create 3 machines with OS: ubuntu 18.04 server with following names:
- storage nodes:
  1. gfsnode01
  2. gfsnode02
- GlusterFS Client  
  3. gfsclient


# To run on all nodes
$ sudo apt-get update
$ sudo apt-get upgrade -y
$ sudo nano /etc/hosts
-> add this to hosts file:
``` 10.10.1.192  gfsnode01 ```
``` 10.10.2.42   gfsnode02  ```
``` 10.10.2.81   gfsclient ```

$ ping -c 3 gfsnode01
$ ping -c 3 gfsnode02
$ ping -c 3 gfsclient

$ sudo apt install software-properties-common -y 
$ sudo apt-get install software-properties-common

# gfsnode1 password = node123
# gfsnode2 password = node123
# gfsclient password = node123

# $ wget -O- https://download.gluster.org/pub/gluster/glusterfs/3.12/rsa.pub | apt-key add -
# $ sudo add-apt-repository ppa:gluster/glusterfs-3.12

# to run on both gfsnodes:
$ sudo add-apt-repository ppa:gluster/glusterfs-5
$ sudo apt install glusterfs-server
$ sudo systemctl enable glusterd
$ sudo systemctl status glusterd
$ sudo glusterfsd --version


# to run on gfsclient:
$ sudo bash
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:gluster/glusterfs-5
$ sudo apt install glusterfs-client

# Setup Distributed GlusterFS Volume on Ubuntu 18.04
- Configure Firewall
- If UFW is running, run the command below to allow the storage nodes to communicate with each other.
#  run on node1
$ sudo ufw allow from 10.10.2.42
#  run on node2
$ sudo ufw allow from 10.10.1.192

# Configure GlusterFS Trusted Pool
- To create a trusted storage pool between the nodes, run the probe from Storage gfsnode01 as shown below;
$ sudo gluster peer probe gfsnode02

# To check the status of the trusted pool just created above, execute the command below on both storage nodes; 
$ sudo gluster peer status

- To list the storage pools;
$ sudo gluster pool list

`root@gfsnode01:~#` gluster peer status
Number of Peers: 1

Hostname: gfsnode02
Uuid: 2e83959f-a21b-4fab-9062-c6a41721a452
State: Peer in Cluster (Connected)
`root@gfsnode01:~#` gluster pool list
UUID					Hostname 	State
2e83959f-a21b-4fab-9062-c6a41721a452	gfsnode02	Connected 
9766544f-f220-4066-b346-eac5211399c6	localhost	Connected 


# Create Distributed GlisterFS Volume
- Create a new directory /glusterfs/distributed on each storage servers.Create a brick directory for GlusterFS volumes on the GlusterFS storage device mount point on both storage nodes.
$ mkdir -p /glusterfs/distributed

- From the gfsnodeO1 server, create the distributed glusterfs volume named ‘vol01’ with 2 replicas ‘gfsnode01’ and ‘gfsnode02’
$ sudo gluster volume create vol01 transport tcp gfsnode01:/glusterfs/distributed gfsnode02:/glusterfs/distributed force

- start the ‘vol01’ and check the volume info
$ gluster volume start vol01
$ gluster volume info vol01
$ gluster volume info 


```root@gfsnode01:~#``` gluster volume start vol01
volume start: vol01: success

```root@gfsnode01:~#``` gluster volume info vol01
 
Volume Name: vol01
Type: Distribute
Volume ID: fd4d3d06-a721-4359-9708-40a0b39a2289
Status: Started
Snapshot Count: 0
Number of Bricks: 2
Transport-type: tcp
Bricks:
Brick1: gfsnode01:/glusterfs/distributed
Brick2: gfsnode02:/glusterfs/distributed
Options Reconfigured:
transport.address-family: inet
nfs.disable: on

# on client machine:
- Now create a new directory '/mnt/glusterfs' when the glusterfs-client installation is complete.
$ sudo mkdir -p /mnt/glusterfs

- Mount the distributed glusterfs volume ‘vol01’ to the ‘/mnt/glusterfs’ directory.
$ mount -t glusterfs gfsnode01:/vol01 /mnt/glusterfs

- Now check the available volume on the system.
$ sudo df -h /mnt/glusterfs

```root@gfsclient:~#``` df -h /mnt/glusterfs
Filesystem        Size  Used Avail Use% Mounted on
gfsnode01:/vol01   78G  3.2G   75G   5% /mnt/glusterfs


# Additional: To mount glusterfs permanently to the Ubuntu client system, we can add the volume to the '/etc/fstab'.
```root@gfsclient:~#``` nano /etc/fstab
```root@gfsclient:~#``` cat /etc/fstab
LABEL=cloudimg-rootfs	/	 ext4	defaults	0 0
LABEL=UEFI	/boot/efi	vfat	defaults	0 0
gfsnode01:/vol01 /mnt/glusterfs glusterfs defaults,_netdev 0 0

- Reboot the server. When online, the glusterfs volume ‘vol01’ is mounted automatically through the fstab.

# Testing Replication & Mirroring
- Mount the glusterfs volume ‘vol01’ to each glusterfs servers.
$ mount -t glusterfs gfsnode01:/vol01 /mnt
$ mount -t glusterfs gfsnode02:/vol01 /mnt

# Now back to the Ubuntu client and go to the '/mnt/glusterfs' directory.
$ cd /mnt/glusterfs

- Create three files using touch command
$ touch file01 file02 file03

- Check on each ‘gfsnode01’ and ‘gfsnode02’ that the files that we’ve created from the client machine are displayed.
$ cd /mnt/
$ ls -lah

# gfsnode01
```root@gfsnode01:~#``` cd /mnt/
```root@gfsnode01:/mnt#``` ls -lah
total 8.0K
drwxr-xr-x  3 root root 4.0K Jan  5 16:25 .
drwxr-xr-x 25 root root 4.0K Jan  5 14:56 ..
-rw-r--r--  1 root root    0 Jan  5 16:25 file01
-rw-r--r--  1 root root    0 Jan  5 16:25 file02
-rw-r--r--  1 root root    0 Jan  5 16:25 file03

# gfsnode02

```root@gfsnode02:~#``` cd /mnt/
```root@gfsnode02:/mnt#``` ls -lah
total 8.0K
drwxr-xr-x  3 root root 4.0K Jan  5 16:25 .
drwxr-xr-x 24 root root 4.0K Jan  5 15:47 ..
-rw-r--r--  1 root root    0 Jan  5 16:25 file01
-rw-r--r--  1 root root    0 Jan  5 16:25 file02
-rw-r--r--  1 root root    0 Jan  5 16:25 file03

# results: All files that we created from the client machine will be distributed to all the glusterfs volume node servers.



