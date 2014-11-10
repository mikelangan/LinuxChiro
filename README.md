LinuxChiro
==========

Posture checking and correcting for Linux

What it checks:
 - Any specified files/folders for permissions or ownership
 - It also checks config filed for correct option/value 
   - Ex: "PermitEmptyPasswords no" in sshd_config or "net.ipv4.tcp_syncookies = 1" in /etc/sysctl.conf
 - The following config files have working examples
    - /etc/sysctl.conf
    - /etc/ssh/ssh_config
    - /etc/ssh/sshd_config
    - /etc/resolv.conf
    - /etc/login.defs
    - /etc/sysconfig/init
    - /etc/pam.d/system-auth

Core functionality:
- Examining  - Audit things to be fixed, things already fixed or the status of all 
- Prescribing - Produces a list of what commands that can be run to resolve issues
- Perform adjustments - Fix issues interactively (locally) and non-interactively (locally/remotely) 
- Insurance - Backs up things before making each change
- Easily revert - Creates a list of commands to run to easily undo changes 

Other features:
- Runs remotely via a pipe (it won't take arguments when run remotely but you can hard code the option) 
- Runs as non root and performs rudimentary check for required sudo privs (better checks coming in future version).
  - It will run as root locally but it will warn you. 
- Checks for the availability of required commands before running 
- Changes are made immediately active and added to the appropriate files for the next service (ex sshd) restart 
- Descriptive/Custom exit codes for each unique exit situation
- Provides brief/simple reasons for making each change, ability to improve on this
- Runs on RHEL/Centos 6 (Rhel 5 mostly works - RHEL7 support soon) 

Limitations:
- Currently it will not create files that don't exist but should - it will only warn you in this situation. 
  - It will add options/values that don't exist in a (existing) config file
- Currently it will not delete options or files that should not exist. 
  - This isn't a big deal because we can usually just set them to an appropriate value but some things need to be removed. 

Major Feature additions in the works 
- Ability to load checks from separate config files
- Package and Service checks
- Custom white lists per hosts for decentralized management of hosts. 


Usage and Example:

[autobot@server ~]$ git clone https://github.com/johnculkin/LinuxChiro.git
Cloning into 'LinuxChiro'...
remote: Counting objects: 486, done.
remote: Compressing objects: 100% (96/96), done.
remote: Total 486 (delta 55), reused 0 (delta 0)
Receiving objects: 100% (486/486), 116.56 KiB, done.
Resolving deltas: 100% (317/317), done.
[autobot@server ~]$ cd LinuxChiro/
[autobot@server LinuxChiro]$ chmod +x checkup.sh
[autobot@server LinuxChiro]$

[autobot@server LinuxChiro]$ ./checkup.sh
You are running as a non-root user with the correct sudo privs.
OSVER is 5 - Supported
Usage: ./checkup.sh audit|audit-on|audit-off|fix-quiet|fix-notify|interactive

audit-all   - make no changes, notify status of all checks
audit-on    - make no changes, notify status of only implemented checks
audit-off   - make no changes, notify status of only non-implemented checks
fix-quiet   - make changes, no notifications
fix-notify  - make changes, with notifications of changes made
interactive - step through each check, give status of all and prompt to fix (if needed)

[autobot@server LinuxChiro]$


[autobot@server LinuxChiro]$ ./checkup.sh audit-off
You are running as a non-root user with the correct sudo privs.
OSVER is 5 - Supported

!!!Warning /etc/security/console.perms does not exist
!!!Warning /var/spool/cron/root does not exist
!!!Warning /root/.tcshrc does not exist
!!!Warning /root/.bashrc does not exist
!!!Warning /root/.cshrc does not exist
---Incorrect /var/log/wtmp is set to 664 should be 600
---Incorrect /var/log/rpmpkgs is set to 644 should be 640
---Incorrect /var/lib/nfs is set to 755 should be 750
---Incorrect /var/log/sa is set to 755 should be 600
!!!Warning /var/spool/cron/root does not exist
---Incorrect /etc/gshadow is set to 0 should be 0000
---Incorrect /etc/shadow is set to 0 should be 0000
!!!Warning /var/log/audit/audit.log does not exist
!!!Warning /var/spool/cron/root does not exist
---Incorrect /var/log/btmp is set to root:utmp should be root:root
---Incorrect /etc/pam.d/atd is set to root:daemon should be root:root
---Incorrect /var/log/wtmp is set to root:utmp should be root:root
!!!Warning /var/log/audit/audit.log does not exist
---Incorrect PASS_MIN_LEN in /etc/login.defs is set to 5, it should be 14
---Incorrect PASS_MIN_DAYS in /etc/login.defs is set to 0, it should be 1
---Incorrect PASS_MAX_DAYS in /etc/login.defs is set to 99999, it should be 60

[autobot@server LinuxChiro]$


[autobot@server LinuxChiro]$ ./checkup.sh fix-notify
You are running as a non-root user with the correct sudo privs.
OSVER is 5 - Supported

+++Changing  /var/log/wtmp to 600 to sec_logs
+++Changing  /var/log/rpmpkgs to 640 to sec_logs
+++Changing  /var/lib/nfs to 750 to secure_nfs_folder
+++Changing  /var/log/sa to 600 to secure_sar_files
+++Changing  /etc/gshadow to 0000 to secure_gshadow
+++Changing  /etc/shadow to 0000 to secure_shadow
+++Changing  /var/log/btmp to root:root to secure_logs
+++Changing  /etc/pam.d/atd to root:root to secure_pam_conf
+++Changing  /var/log/wtmp to root:root to secure_wtmp
+++Changing Changing existing entry - PASS_MIN_LEN 14 in /etc/login.defs
+++Changing Changing existing entry - PASS_MIN_DAYS 1 in /etc/login.defs
+++Changing Changing existing entry - PASS_MAX_DAYS 60 in /etc/login.defs

Based on the PRESCRIBECOMMANDSONLY variable, we will NOT run the following commands in /tmp/tmp.OwYMT15392/commandstorun.lis
#!/bin/bash
sudo chmod 600 /var/log/wtmp
sudo chmod 640 /var/log/rpmpkgs
sudo chmod 750 /var/lib/nfs
sudo chmod 600 /var/log/sa
sudo chmod 0000 /etc/gshadow
sudo chmod 0000 /etc/shadow
sudo chown root:root /var/log/btmp
sudo chown root:root /etc/pam.d/atd
sudo chown root:root /var/log/wtmp


Compressed the Prescribed commands to /tmp/tmp.OwYMT15392.tar.gz

Here are the commands to undo the changes if you made them:
#!/bin/bash
sudo chmod 664 /var/log/wtmp
sudo chmod 644 /var/log/rpmpkgs
sudo chmod 755 /var/lib/nfs
sudo chmod 755 /var/log/sa
sudo chmod 0 /etc/gshadow
sudo chmod 0 /etc/shadow
sudo chown root:utmp /var/log/btmp
sudo chown root:daemon /etc/pam.d/atd
sudo chown root:utmp /var/log/wtmp

Backups of files that were changed and commands to undo changes are available in /tmp/tmp.SnHUy15391.tar.gz

[autobot@server LinuxChiro]$



To Dos:

-- Put all text echos through the function (even errors) 

-- Put a catch all "else" in every if/case statement and custom error

-- Get arugments working via pipe (important for massh usage). Might be tough, may have to rename script and check $0

-- Get rid of all the exit code status checks in the if statements, put the greps right in the If block 

-- Ability to set max and min for config options, Ex: --Incorrect PASS_MAX_DAYS in /etc/login.defs is set to 99999, it should be 60

-- Create a way to remove options from config files and remove files that should not exist - Ex Rhost Auth from pam files and Cron log watch spam "/etc/cron.daily/0logwatch"

-- Set old password limits by outting "remember=24" at the correct place in the correct line of /etc/pam.d/system-auth

-- Check the order of tests on a fresh install - ex config network before repo and packages, config packages before options and files perms/ownership - or just recommend to rerun script when making certain changes?

-- Create white list of things that should not be changed on a host. Could this be easily tied to the remediation script? Just remove the lines from the remediation script that are in the white list file???

-- Break out config of checks to separate file - maybe not, its nice to have everything in one file - maybe just organize the file better. Config files could be centralized or local on host being checked. Set config file to be used via command line argument?

-- Allow for multiple occurrences of an option/values in a file (ex: two name servers in /etc/resolve.conf)

-- Plan for Apache Confs - do in same config loop? Ex ssl cipher strength and protocol (poodle) 

-- Add check for updates - #"yum check-update"

-- Package Checking - just run rpm once and store output in variable

-- Add a way to check for packages that should NOT be installed - combine with exisitng check 

-- Service check for things that should not be running

-- Add a way to check for services that should be running - combine with exisitng check 

-- Add execute bit on script in repo - maybe not?

-- Get it working with RHEL/Centos 7, give script the ability to check different things per OS version, maybe this is better handled with OS specific config files for the checks?

-- Allow command line changing of the backup option, currenlty its hard coded. 

-- Allow command line changing of the PRESCRIBECOMMANDSONLY option, currenlty its hard coded.

-- Get a better way to determine RedHat/Centos version

-- Create a file to test the script during development - it could just be a set of prescribe and undo scripts 

-- Improve the way that we check if the user running the script has the correct sudo privs, tirm white space etc

-- Check sysctl values before we set then in /etc/sysctl.conf? Maybe They are the defaults and already in place. We will want to add the values to /etc/sysctl but maybe we don't need to make the active change. Same with sshd? Low priority 

