# ICA0006-ha-storage
Distributed storage solution for TalTech ICA0006 class
Authors: Karl-Andreas Turvas, Katariina Maribel Vink, Angelina Vinogradova, Mari-Ann Veende 

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


## Installing glusterfs-server:

###  install dependency and add repo
    sudo apt-get install software-properties-common -y
    sudo add-apt-repository ppa:gluster/glusterfs-11
    sudo apt update
### install
    sudo apt install glusterfs-server
### start and enable
    sudo systemctl start glusterd
    sudo systemctl enable glusterd

## On each server the ones that isn't the server itself:

    iptables -I INPUT -p all -s 192.168.180.73 -j ACCEPT

    iptables -I INPUT -p all -s 192.168.161.207 -j ACCEPT

    iptables -I INPUT -p all -s 192.168.161.208 -j ACCEPT

    iptables -I INPUT -p all -s 192.168.161.209 -j ACCEPT

## On 192.168.180.73:

### Probe peers: 

    gluster peer probe 192.168.161.207

    gluster peer probe 192.168.161.208

    gluster peer probe 192.168.161.209

### Result: 

```
root@lab:~# gluster peer probe 192.168.161.207
peer probe: success

root@lab:~# gluster peer probe 192.168.161.208
peer probe: success

root@lab:~# gluster peer probe 192.168.161.209
peer probe: success
```

### Check peer status: 
    gluster peer status
### Result: 

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

### Command (kas see oli vajalik? ja kus?): 
    ufw allow 24009 

### 192.168.161.207:
```
sudo mkdir -p /mnt/glusterfs
sudo mount /dev/sda2 /mnt/glusterfs
sudo mkdir -p /mnt/glusterfs/brick1
echo "/dev/sda2  /mnt/glusterfs  ext4  defaults  0  0" | sudo tee -a /etc/fstab
```

### 192.168.161.208:
```
sudo mkdir -p /mnt/glusterfs
sudo mount /dev/sda2 /mnt/glusterfs
sudo mkdir -p /mnt/glusterfs/brick2
echo "/dev/sda2  /mnt/glusterfs  ext4  defaults  0  0" | sudo tee -a /etc/fstab
```

### 192.168.161.209:
```
sudo mkdir -p /mnt/glusterfs
sudo mount /dev/sda2 /mnt/glusterfs
sudo mkdir -p /mnt/glusterfs/brick3
echo "/dev/sda2  /mnt/glusterfs  ext4  defaults  0  0" | sudo tee -a /etc/fstab
```

## Volume creation:
```
sudo gluster volume create volume1 replica 3 transport tcp 192.168.161.207:/mnt/glusterfs/brick1 192.168.161.208:/mnt/glusterfs/brick2 192.168.161.209:/mnt/glusterfs/brick3 force
```
### Result: 
```
student@lab:~$ sudo gluster volume create volume1 replica 3 transport tcp 192.168.161.207:/mnt/glusterfs/brick1 192.168.161.208:/mnt/glusterfs/brick2 192.168.161.209:/mnt/glusterfs/brick3 force
volume create: volume1: success: please start the volume to access data
```
### Start the volume: 
    sudo gluster volume start volume1
### Result: 
```
student@lab:~$ sudo gluster volume start volume1
volume start: volume1: success
```

## Client:
```
gluster volume info all
apt update && sudo apt install --assume-yes glusterfs-client
mkdir -p /mnt/glusterfs
mount -t glusterfs 192.168.161.207:/volume1 /mnt/glusterfs
```
### Into vim /etc/fstab:

    192.168.161.207:/volume1 /mnt/glusterfs glusterfs defaults,_netdev 0 0

### Check Disk Space Usage: 
    df /mnt/glusterfs/
### Result: 
```
root@lab:~# df /mnt/glusterfs/
Filesystem                1K-blocks     Used  Available Use% Mounted on
192.168.161.207:/volume1 1152222916 24638648 1080503080   3% /mnt/glusterfs
```

