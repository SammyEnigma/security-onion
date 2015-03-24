# Introduction #

How to install and get started with Ntopng on a Security Onion box

# Author #

This page was written by Kevin Branch.

# Testing #

This procedure was last successfully tested by Kevin on 1/30/2015 when the latest stable ntopng was at version 1.2.2, rev `8661`.  At that time, the newer nightly build revision `8884` was broken, so this article now references the stable repo instead of the nightly build repo.

# No Support #

Since ntopng is not an official part of Security Onion, we don't provide official support for it.

# About the ntopng apt repository #

Don't add that repo to your Security Onion system.  As nice as it would be to have ntopng updated as part of the "soup" process, adding the ntopng apt repository to your apt sources appears to break Security Onion because it will cause the pfring package from the ntopng repo to be installed in parallel to the Security Onion securityonion-pfring-`*` packages, which generally aren't on the same PFRING version, causing various applications to fail.

If you've already been broken by this, I believe that after removing the ntop repository from your apt sources, the following will put things back to normal:
```
apt-get purge pfring
apt-get --reinstall install securityonion-pfring-*
```

# Are your http requests for ntop getting redirected to https? #

Something in Security Onion appear to use HSTS to mark the SO host name for HTTPS-only use, so when you use a browser that honors HSTS -- like Chrome -- to access your ntop instance via the same host name as your other SO apps, it will redirect you to https which isn't what ntop is listening on.  The ideal solution is to enable https support in ntop, but I've not had luck with that yet.  The other options are to access ntop via the raw IP number of the SO box, or to set up an additional DNS name that points at your SO system, and use that name exclusively for reaching ntop.

# Installation Details #

```
# Ntopng depends on this package
apt-get install redis-server libhiredis0.10 rrdtool libnl1

# Fetch and install the latest stable build of the ntopng and ntopng-data deb packages
DEB_FILE_1=`curl -s http://www.nmon.net/apt-stable/12.04/x64/ 2>&1 | grep ntopng_ | tail -n1 | cut -d\" -f8`
DEB_FILE_2=`curl -s http://www.nmon.net/apt-stable/12.04/all/ 2>&1 | grep ntopng-data | tail -n1 | cut -d\" -f8`
wget http://www.nmon.net/apt-stable/12.04/x64/$DEB_FILE_1
wget http://www.nmon.net/apt-stable/12.04/all/$DEB_FILE_2
dpkg -i $DEB_FILE_1 
dpkg -i $DEB_FILE_2

# Make ntopng start at boot
touch /etc/ntopng/ntopng.start

# Set up an ntopng data directory
mkdir /usr/local/ntopng
chown nobody:root /usr/local/ntopng

# Edit /etc/ntopng/ntopng.conf, using the ntopng man page to figure out what options you want.
# You might start with something like this:
	--data-dir=/usr/local/ntopng
	--local-networks="192.168.0.0/16,10.0.0.0/8"
	--interface=eth1
	--dns-mode=1
	--disable-login
	--packet-filter="ip and not proto ipv6 and not ether host ff:ff:ff:ff:ff:ff and not net (224.0.0.0/8 or 239.0.0.0/8)"
	--daemon
	-G=/var/tmp/ntopng.pid
# Make sure to include at least the -G line in the above example exactly as seen, as the ntopng control script will fail without it.

# Open the ntopng web listener port in iptables
ufw allow 3000/tcp

# Manually start the service
service ntopng start

# Surf to http://NSM_HOST_NAME:3000 and try out your new ntopng installation!
```