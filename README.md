# ICA0006-ha-storage
Distributed storage solution for TalTech ICA0006 class
Authors: 

- Karl-Andreas Turvas (team lead, debugger + advisor)
- Katariina Maribel Vink (main admin)
- Angelina Vinogradova (admin apprentice and documentor)
- ~~Mari-Ann Veende~~ (did not show up)

 ## ILO is configured for all servers with the following IP addresses:
| ILO             | SERVER OS IP    |
|-----------------|-----------------|
| 192.168.161.203 | 192.168.161.207 |
| 192.168.161.204 | 192.168.161.208 |
| 192.168.161.205 | 192.168.161.209 |

VM IP  192.168.180.73 

OS: Ubuntu 22.04.3 LTS

## 3 servers running Ubuntu 22.04.3 LTS, 8 drives set up with RAID 1+0
| Name      | IP address      | User    | Password    |
|-----------|-----------------|---------|-------------|
| Server207 | 192.168.161.207 | student | student1234 |
| Server208 | 192.168.161.208 | student | student1234 |
| Server209 | 192.168.161.209 | student | student1234 |

We are using <a href="https://docs.ansible.com/" target="_blank">Ansible</a> for automation.


<a href="https://docs.gluster.org/en/main/" target="_blank">GlusterFS</a> is our choice of network filesystem.

## Ansible usage

Install dependencies: `ansible-galaxy install -r requirements.yml` (force update with `--force`)

Run playbooks:

```bash
ansible-playbook playbooks/group_gluster_client/main.yml
ansible-playbook playbooks/group_gluster_server/main.yml
```

## Installing cli tool for hardware RAID

In /etc/apt/sources.list.d/hpe.list and add the following:

```
deb http://downloads.linux.hpe.com/SDR/repo/mcp bionic/current non-free
```

Install the ssacli packages:

```
apt update
apt install ssa ssacli ssaducli
```

Show config:

```
root@server209: ssacli ctrl all show config
Smart Array P410i in Slot 0 (Embedded)    (sn: 5001438001ACF8F0)
   Internal Drive Cage at Port 1I, Box 1 (Index 0), OK
   Internal Drive Cage at Port 2I, Box 1 (Index 1), OK
   Port Name: 1I
   Port Name: 2I
   Array A (SAS, Unused Space: 0  MB)
      logicaldrive 1 (1.09 TB, RAID 1+0, OK)
      physicaldrive 1I:1:1 (port 1I:box 1:bay 1, SAS HDD, 300 GB, OK)
      physicaldrive 1I:1:2 (port 1I:box 1:bay 2, SAS HDD, 300 GB, OK)
      physicaldrive 1I:1:3 (port 1I:box 1:bay 3, SAS HDD, 300 GB, OK)
      physicaldrive 1I:1:4 (port 1I:box 1:bay 4, SAS HDD, 300 GB, OK)
      physicaldrive 2I:1:5 (port 2I:box 1:bay 5, SAS HDD, 300 GB, OK)
      physicaldrive 2I:1:6 (port 2I:box 1:bay 6, SAS HDD, 300 GB, OK)
      physicaldrive 2I:1:7 (port 2I:box 1:bay 7, SAS HDD, 300 GB, OK)
      physicaldrive 2I:1:8 (port 2I:box 1:bay 8, SAS HDD, 300 GB, OK)
   SEP (Vendor ID PMCSIERA, Model  SRC 8x6G) 250  (WWID: 5001438001ACF8FF)

root@server209: ssacli ctrl all show status
Smart Array P410i in Slot 0 (Embedded)
   Controller Status: OK
   Cache Status: OK
   Battery/Capacitor Status: OK
```

Try to remove disk to test redundancy:

```
root@server209: ssacli controller slot=0 array A remove drives=1:8
Error: This operation is not supported with the current configuration. Use the
       "show" command on devices to show additional details about the
       configuration.
       Reason: Feature requires license key
```

## Installing Gluster Server:

Add repo:

    sudo apt-get install software-properties-common -y
    sudo add-apt-repository ppa:gluster/glusterfs-11
    sudo apt update

Install gluster:

    sudo apt install glusterfs-server

Start and enable gluster daemon:
 
    sudo systemctl start glusterd
    sudo systemctl enable glusterd

On each server the ones that isn't the server itself:

    iptables -I INPUT -p all -s 192.168.180.73 -j ACCEPT

    iptables -I INPUT -p all -s 192.168.161.207 -j ACCEPT

    iptables -I INPUT -p all -s 192.168.161.208 -j ACCEPT

    iptables -I INPUT -p all -s 192.168.161.209 -j ACCEPT

Or use:

    ufw allow 24009 

192.168.161.207:

```
sudo mkdir -p /mnt/glusterfs
sudo mount /dev/sda2 /mnt/glusterfs
sudo mkdir -p /mnt/glusterfs/brick1
echo "/dev/sda2  /mnt/glusterfs  ext4  defaults  0  0" | sudo tee -a /etc/fstab
```

192.168.161.208:

```
sudo mkdir -p /mnt/glusterfs
sudo mount /dev/sda2 /mnt/glusterfs
sudo mkdir -p /mnt/glusterfs/brick2
echo "/dev/sda2  /mnt/glusterfs  ext4  defaults  0  0" | sudo tee -a /etc/fstab
```

192.168.161.209:

```
sudo mkdir -p /mnt/glusterfs
sudo mount /dev/sda2 /mnt/glusterfs
sudo mkdir -p /mnt/glusterfs/brick3
echo "/dev/sda2  /mnt/glusterfs  ext4  defaults  0  0" | sudo tee -a /etc/fstab
```

Volume creation:

```
sudo gluster volume create volume1 replica 3 transport tcp 192.168.161.207:/mnt/glusterfs/brick1 192.168.161.208:/mnt/glusterfs/brick2 192.168.161.209:/mnt/glusterfs/brick3 force
```

Result: 

```
student@lab:~$ sudo gluster volume create volume1 replica 3 transport tcp 192.168.161.207:/mnt/glusterfs/brick1 192.168.161.208:/mnt/glusterfs/brick2 192.168.161.209:/mnt/glusterfs/brick3 force
volume create: volume1: success: please start the volume to access data
```

Start the volume: 

    sudo gluster volume start volume1

Result: 

```
student@lab:~$ sudo gluster volume start volume1
volume start: volume1: success
```

## Test server status from client VM 192.168.180.73

Probe peers: 

    gluster peer probe 192.168.161.207

    gluster peer probe 192.168.161.208

    gluster peer probe 192.168.161.209
    
Result: 

```
root@lab:~# gluster peer probe 192.168.161.207
peer probe: success

root@lab:~# gluster peer probe 192.168.161.208
peer probe: success

root@lab:~# gluster peer probe 192.168.161.209
peer probe: success
```

Check peer status: 

    gluster peer status

Result: 

```
root@lab:~# gluster peer status
Number of Peers: 3

Hostname: 192.168.161.207
Uuid: b0853efc-92c3-4c3b-99d1-b468d3816f69
State: Peer in Cluster (Connected)

Hostname: 192.168.161.208
Uuid: 690298e5-15d8-4223-a7eb-5e545a5afca9
State: Peer in Cluster (Connected)

Hostname: 192.168.161.209
Uuid: 3f74dbd2-9b68-4add-ad24-13c0e2f24bad
State: Peer in Cluster (Connected)
```

## Client setup

```
gluster volume info all
apt update && sudo apt install --assume-yes glusterfs-client
mkdir -p /mnt/glusterfs
mount -t glusterfs 192.168.161.207:/volume1 /mnt/glusterfs
```

Add to /etc/fstab:

    192.168.161.207:/volume1 /mnt/glusterfs glusterfs defaults,_netdev 0 0

Check Disk Space Usage:

    df /mnt/glusterfs/

Result:

```
root@lab:~# df /mnt/glusterfs/
Filesystem                1K-blocks     Used  Available Use% Mounted on
192.168.161.207:/volume1 1152222916 24638648 1080503080   3% /mnt/glusterfs
```

## Test replication

On client add a file:

```
root@lab: touch /mnt/glusterfs/testfile.txt
```

NOTE: Replication only works from clients, if adding files onto bricks from servers, files will not be replicated!

On servers check to see file:

```
root@192.168.161.208: ls /mnt/glusterfs/brick2
testfile.txt

root@192.168.161.209: ls /mnt/glusterfs/brick3
testfile.txt

root@192.168.161.207: ls /mnt/glusterfs/brick1
testfile.txt
```

Disable one server to check replication:

```
root@192.168.161.209: sudo systemctl stop glusterd
root@lab: ls /mnt/glusterfs/
testfile.txt
# Yay, the file still exists!
```
