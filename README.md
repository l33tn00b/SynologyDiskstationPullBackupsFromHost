# SynologyDiskstationPullBackupsFromHost
Using a Diskstation to Pull Backups From a Remote Host

This is based on https://guficulo.blogspot.com/2015/04/backup-your-raspberry-pi-automatically.html but modified for newer versions of DSM.


# Basic Idea
Use the Diskstation to pull a backup from a remote linux host. 
We could also use the host to rsync to the Diskstation. Why not? Because if the host is broken into, we'll expose Diskstation login data. Well, we could mitigate by having a separate backup user for each host. 

# Prepare the host that is to be backed up
Assuming it's a Raspberry Pi:
- enable root password login:
  -   
