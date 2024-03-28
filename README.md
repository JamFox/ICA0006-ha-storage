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
