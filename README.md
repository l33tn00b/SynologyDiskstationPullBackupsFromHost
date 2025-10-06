t# SynologyDiskstationPullBackupsFromHost
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

# On Host
- disable password login via ssh
- make absolutely sure you're NOT on your Diskstation's console
- remove root passwd: `sudo passwd root -d`


# On Diskstation
- make directory `backups` in your admin home
- make directory `backups/logs` in your admin home
- make directory `backups/<your host name>` in your admin home
- make directory `backup/_scripts` in your admin home
- place file `backup_target.sh` in `backup/_scripts` of your admin home:
  ```
  SERVER=$1
  ADDRESS=$2
  NOW=$(date +"%Y-%m-%d")
  LOGFILE="$SERVER-$NOW.log"
  ping $ADDRESS -c 5 >> /volume1/homes/<your admin username>/backups/logs/$LOGFILE
  /usr/bin/rsync -av --delete --exclude-from=/volume1/homes/<your admin name>/backups/_scripts/rsync-exclude.txt -e "ssh -o IdentitiesOnly=yes -i ~/.ssh/id_ed25519_diskstation" root@$ADDRESS:/ /volume1/homes/<your admin username>/backups/$SERVER/ >> /volume1/homes/<your admin username>/backups/logs/$LOGFILE 2>&1
  ```
- place file `rsync-exclude.txt` in `backup/_scripts` of your admin home (change contents according to your requirements, this will not backup home directory of user pi):
  ```
  *.tmp
  /tmp/*
  /home/pi/
  /var/*
  /mnt/*
  /media/*
  /sys/*
  /proc/*
  ```
- set up task for regular execution

# ToDo:
- make ping work (requires sudo privileges) or throw it out
