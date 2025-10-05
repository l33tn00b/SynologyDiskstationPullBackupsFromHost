# SynologyDiskstationPullBackupsFromHost
Using a Diskstation to Pull Backups From a Remote Host

This is based on https://guficulo.blogspot.com/2015/04/backup-your-raspberry-pi-automatically.html but modified for newer versions of DSM.


# Basic Idea
Use the Diskstation to pull a backup from a remote linux host. 
We could also use the host to rsync to the Diskstation. Why not? Because if the host is broken into, we'll expose Diskstation login data. Well, we could mitigate by having a separate backup user for each host. 

# Prepare the host that is to be backed up
Assuming it's a Raspberry Pi:
- enable root password login:
  - login to raspberry with regular user account
  - `sudo passwd root`, set password (remember it!)
  - `nano /etc/ssh/sshd_config`, allow root login with password
- check if ssh root login with password works
 
# On Diskstation
- login via ssh with admin account
- be root: `sudo -i`
- create key pair: `ssh-keygen -t ed25519 -f /root/.ssh/id_ed25519_diskstation -N ""`
- copy keys to client (ssh-copy-id doesn't work on Synology): ` cat ~/.ssh/id__ed25519_diskstation.pub | ssh root@<your client's IP address> 'cat >> .ssh/authorized_keys'`
- make directory for keys: `/var/services/homes/<your admin username>/.ssh`
- copy keys to your admin user's directory: `cp /root/.ssh/id_ed25519_diskstation* /var/services/homes/<your admin username>/.ssh`
- adjust permissions:
  ```
  chown -R <admin username>:users /var/services/homes/<admin username>/.ssh
  chmod 700 /var/services/homes/<admin username>/.ssh
  chmod 600 /var/services/homes/<admin username>/.ssh/id_ed25519_diskstation
  chmod 644 /var/services/homes/<admin username>/.ssh/id_ed25519_diskstation.pub
  ```
- check passwordless login
