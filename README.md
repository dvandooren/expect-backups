# Expect scripts to backup network device configs

## Backup Nexus configs (`grab-n5k-configs.exp`)
### Create user and ssk keys
On the server that is going to run the Expect scripts to perform the backups, add a new user (ie. `backup-configs`). Change to that user and create ssh keys (`ssh-keygen -t rsa -b 4096`) so that it can login to the switches with it's public key.

### Create a role for the backup user
On the Nexus switches you must create a user and ideally only allow it to run a few commands to display the config

Create a *role* with the following commands:
```
role name config-backup
  rule 3 permit command terminal length *
  rule 2 permit command show running-config
  rule 1 permit read  
```

Create a new user that authenticates with an ssh-key with the following commands:
```
username <backup-user> password 5 !  role config-backup
username <backup-user> sshkey ssh-rsa <users_ssh_publickey>
```
> **NOTE:** The public key should be all on one line and end with `==`. It shouldn't include the *identifier* after the `==`.

### Config the grab-n5k-configs.exp script
Edit the file `grab-n5k-configs.exp` and change the following lines appropiately:
```expect
set username backup-configs
set devicelist "switch-a switch-b"
set backupdir "/config-backups/N5K/"
```
> **NOTE:** The names used the `deviceList` are used as part of the backup file name so I prefer to use just the hostname of the device and ensure it's in my /etc/hosts file so it can always resolve.

### Add cron jobs to run the script
Add cron job to delete configs older than 120 days
```bash
0 4 * * * /bin/find /config-backups/N5K -type f -mtime +120 -delete
```

Add cron job to run the `expect` script
```bash
5 4 * * * /opt/grab-configs/grab-n5k-configs.exp
```

> **NOTE:** add the cron jobs as the user you want the script to run as. I create a local `backup-configs` user. Also ensure this user has the proper permissions to the `backupdir` assigned in the script and cron entries.

---

## Backup ASA configs (`grab-asa-configs.exp`)
*Based on Cisco Adaptive Security Appliance Software Version 9.2(2)4 which doesn't support ssh-keys for users*

### Create a *role* for the backup user
On the ASA firewalls you must create a user and ideally only allow it to run a few commands to display the config

Create a *role* with the following commands:
```
privilege cmd level 4 mode exec command terminal
privilege show level 4 mode exec command running-config
privilege show level 4 mode exec command terminal
```

Create a new user with a password with the following commands:
```
username <backup-user> password <password> privilege 4
```

### Config the grab-asa-configs.exp script
Edit the file `grab-asa-configs.exp` and change the following lines appropiately:
```expect
set username backup-user
set password "somepassword"
set devicelist "device1 device2"
set backupdir "/config-backups/ASA/"
```
> **NOTE:** The names used the `deviceList` are used as part of the backup file name so I prefer to use just the hostname of the device and ensure it's in my /etc/hosts file so it can always resolve.

### Add cron jobs to run the script
Add cron job to delete configs older than 120 days
```bash
0 4 * * * /bin/find /config-backups/ASA -type f -mtime +120 -delete
```

Add cron job to run the `expect` script
```bash
5 4 * * * /opt/grab-configs/grab-asa-configs.exp
```

> **NOTE:** add the cron jobs as the user you want the script to run as. I create a local `backup-configs` user. Also ensure this user has the proper permissions to the `backupdir` assigned in the script and cron entries.
