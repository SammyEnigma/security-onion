# UFW - Uncomplicated Firewall #
The default firewall configuration tool for Ubuntu is ufw. By default UFW is enabled on Security Onion.

## Enable/Disable ##
```
sudo ufw enable
```

```
sudo ufw disable
```
## Allow/Deny ##

**example:** allow port 9876 for Xplico
```
sudo ufw allow 9876/tcp
```
**example:** allow irc port range 6667 - 7000
```
sudo ufw allow 6667:7000
```
**example:** deny https
```
sudo ufw deny 443
```

## What Ports Are Opened/Listening ##
**example**
```
sudo ufw status
```
**example output**
```
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
8000/tcp                   ALLOW       Anywhere
7734/tcp                   ALLOW       Anywhere
7736/tcp                   ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
3000/tcp                   ALLOW       Anywhere
172.30.15.16 80/tcp        ALLOW       172.30.15.10
3154/tcp                   ALLOW       Anywhere
```

Above you can see that there is one rule that restricts traffic to the source ip, destination ip and tcp port. This rule was added after installing [DVWA](http://code.google.com/p/dvwa/) on the same VM as Security Onion in order to test the detection of some basic web application attacks. The rule looks like so -

```
sudo ufw allow proto tcp from 172.30.15.10/32 to 172.30.15.16 port 80
```

## Tightening the firewall on a master server ##

By default, a master server allows connections to the following ports from **any** IP address:
  * 22 - SSH
  * 443 - Squert/CapMe
  * 444 - Snorby
  * 514 - Syslog
  * 1514/udp - OSSEC
  * 3154 - ELSA
  * 7734 - Sguil client
  * 7736 - sensor connection to sguild

You may want to restrict those ports to only accepting connections from a subset of IP addresses.

NOTE! Before attempting any firewall changes, you should always ensure you have a backup plan should you accidentally block your own connection.  So make sure that you have DRAC/KVM/physical or some other form of access.

### Firewall rules to allow sensors to connect to master ###
First, add a rule like the following for each of your sensors (replacing a.b.c.d with the actual IP address of the sensor) to connect to ports 22 (SSH) and 7736 (sguild):
```
sudo ufw allow proto tcp from a.b.c.d to any port 22,7736
```

-OR-

If you're running [Salt](Salt.md), then sensors need to connect to ports 4505/tcp and 4506/tcp:
```
sudo ufw allow proto tcp from a.b.c.d to any port 22,4505,4506,7736
```


### Firewall rules to allow syslog devices ###
Next add a rule like the following for the IP addresses that will be sending syslog (port 514 tcp and udp):
```
sudo ufw allow from a.b.c.d to any port 514
```

### Firewall rules to allow OSSEC agents ###
Next add a rule like the following for the IP addresses that will be running the OSSEC agent (port 1514 udp):
```
sudo ufw allow proto udp from a.b.c.d to any port 1514
```

### Firewall rules to allow analysts/administrators to connect to master ###
Then add a rule like the following for the IP addresses or subnet that you'll be using to connect to the master as an analyst/administrator to ports 22 (SSH), 443 (Squert/CapMe), 444 (Snorby), 3154 (ELSA), 7734 (Sguil client), 9876 (Xplico):
```
sudo ufw allow proto tcp from a.b.c.d to any port 22,443,444,3154,7734,9876
```

### Remove default "allow from Anywhere" rules ###
Once you've added these new rules, then you can remove the default "allow from Anywhere" rules.
```
sudo ufw delete allow 22/tcp 
sudo ufw delete allow 443/tcp
sudo ufw delete allow 444/tcp
sudo ufw delete allow 514
sudo ufw delete allow 1514/udp
sudo ufw delete allow 3154/tcp
sudo ufw delete allow 7734/tcp
sudo ufw delete allow 7736/tcp
```

## Tightening the firewall on a sensor ##

By default, a sensor allows connections to the following ports from **any** IP address:
  * 22 - SSH
  * 514 - Syslog
  * 1514/udp - OSSEC

You may want to restrict those ports to only accepting connections from a subset of IP addresses.

NOTE! Before attempting any firewall changes, you should always ensure you have a backup plan should you accidentally block your own connection.  So make sure that you have DRAC/KVM/physical or some other form of access.

### Firewall rules to allow syslog devices ###
First, add a rule like the following for the IP addresses that will be sending syslog (port 514 tcp and udp):
```
sudo ufw allow from a.b.c.d to any port 514
```

### Firewall rules to allow OSSEC agents ###
Next add a rule like the following for the IP addresses that will be running the OSSEC agent (port 1514 udp):
```
sudo ufw allow proto udp from a.b.c.d to any port 1514
```

### Firewall rules to allow analysts/administrators to connect to master ###
Then add a rule like the following for the IP addresses or subnet that you'll be using to connect to the sensor as an analyst/administrator to ports 22:
```
sudo ufw allow proto tcp from a.b.c.d to any port 22
```

### Remove default "allow from Anywhere" rules ###
Once you've added these new rules, then you can remove the default "allow from Anywhere" rules.
```
sudo ufw delete allow 22/tcp 
sudo ufw delete allow 514
sudo ufw delete allow 1514/udp
```

## More Info ##
For more info you can visit the UFW documentation [site](https://help.ubuntu.com/community/UFW)

Note: [Gufw](https://help.ubuntu.com/community/Gufw) is a GUI front end for the ufw.