# Security Onion 12.04 Update Procedure #

## Initiating an update over SSH ##

If you're updating your Security Onion box over an SSH connection and your connection drops, then your update process may be left in an inconsistent state.  It is therefore recommended to run byobu so that your session will continue to run on the Security Onion box even if your connection drops.  Byobu is very handy and we recommend running it all the time to avoid forgetting about it before an update.
```
# install byobu
sudo apt-get install byobu

# enable byobu
byobu-enable

# you're now ready to update
```

For more information about byobu, please see:
https://help.ubuntu.com/community/Byobu

## soup - Security Onion UPdate ##

We recommend using the "soup" script to automatically install updates while avoiding the MySQL/PF\_RING issues described below.
```
sudo soup
```

Please pay attention to the output as it may request that you take specific action, such as manually restarting services.  Also refer to the relevant blog entry for the update as there may be additional information there:  http://blog.securityonion.net

If you get the following error:
```
sudo: soup: command not found
```

then do the following:
```
sudo apt-get update && sudo apt-get install securityonion-sostat
```

For more information, please see:
<a href='http://blog.securityonion.net/2013/08/new-securityonion-packages.html'><a href='http://blog.securityonion.net/2013/08/new-securityonion-packages.html'>http://blog.securityonion.net/2013/08/new-securityonion-packages.html</a></a>

## Distributed deployments ##

Always update the master server first before updating sensors.

## Using salt and soup to Update your entire Deployment ##
https://code.google.com/p/security-onion/wiki/Salt#Using_Salt_to_Install_Updates_Across_Your_Entire_Deployment

## Standard Ubuntu package management tools ##

The "soup" command described above is the recommended method to install updates.  However, you can use standard Ubuntu package management tools to update ALL packages (Ubuntu and Security Onion), but there are some caveats to be aware of:

  * MySQL - if you've already run Setup, please see the [recommended procedure for updating the MySQL packages](MySQLUpdates.md).

  * PF\_RING and new kernel packages
You may be prompted to update your kernel packages and PF\_RING at the same time.  If you do so, the PF\_RING kernel module may get built for your current kernel and not for the newly installed kernel and upon reboot services will fail.  To avoid this, you should install just the PF\_RING kernel module by itself and then install the kernel and any other remaining package updates.  Here's a one-liner that will do that:
```
sudo apt-get update ; sudo apt-get install securityonion-pfring-module ; sudo apt-get dist-upgrade
```
If you accidentally install both the kernel and PF\_RING packages at the same time and then reboot and find out that PF\_RING services (Snort and Suricata) are failing, you can reinstall the securityonion-pfring-module package:
```
sudo apt-get install --reinstall securityonion-pfring-module
```