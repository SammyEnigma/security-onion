# Introduction #

The following are posts from Martin Holste, the author of ELSA, extracted from the Security Onion and Security Onion Testing mailing lists, that provide insight and working examples of the power of ELSA's query capabilities.


# 12/11/12 - [security-onion-testing] #

In any case, be sure to check out the documentation for subsearches (https://code.google.com/p/enterprise-log-search-and-archive/wiki/Documentation#Subsearches) as that's where you get the really powerful queries from.

The queries from today were like this (note that you can always replace "groupby" with using the "Report On" menu button):

Find today's summary of alerts for "current events" and "trojan" alerts
```
   sig_msg:current_events sig_msg:trojan groupby:sig_msg
```
Choose "ET POLICY Proxy Judge Discovery/Evasion (prxjdg.cgi)" from the signature list, which executes:
```
   sig_msg:"ET POLICY Proxy Judge Discovery/Evasion (prxjdg.cgi)"
```
Get a sense for the distribution of the alert over different IP's by pivoting on dstip (or look at the field summary at the top and notice that there is just one unique entry for srcip)
```
   sig_msg:"ET POLICY Proxy Judge Discovery/Evasion (prxjdg.cgi)" groupby:dstip
```
Notice that there's just one entry for srcip and drill down on it
```
   10.0.1.1 class:snort groupby:sig_msg
```
Find some other alerts but nothing concrete, mine for data in URL's (via Bro)
```
   10.0.1.1 class:bro_http groupby:site
```
see the Polish site and drill-down
```
   site:www.aksarat.pl
```
Click "Info" and choose the getPcap plugin to see the content of the request

As a follow-up, look at the URI structure and see if other sites are being used for this checkin
uri:check.rsp groupby:site

Get a feel for how often the check-in occurs
```
   uri:check.rsp groupby:hour
```
Change granularity to per-minute
```
   uri:check.rsp groupby:minute limit:1000
```
Be alerted in the future by clicking "Results..." and choosing "Create alert" so anytime uri:check.rsp shows up you get an email.

# 1/12/13 - [security-onion] #

Once you start looking at connections in ELSA with geoip, you can also use the "whois" plugin in the same way to see a description of the destination network.  So, from Brad's dashboard, you could run this query in the ELSA query box:
```
   icmp udp tcp class=BRO_CONN groupby:BRO_CONN.dstip | whois
```
Then, you can do some post-search filters to remove any well-known hosts like this:
```
   icmp udp tcp class=BRO_CONN groupby:BRO_CONN.dstip | whois | filter(descr,google)
```
Or maybe you want to see TCP traffic not to port 80/443:
```
   tcp class=BRO_CONN groupby:BRO_CONN.dstip -dstport:80 -dstport:443 | whois
```
Or, find only things destined for a high port with a certain byte count (you'll need to wait for the latest ELSA version to make it into SecurityOnion to get the bytes\_in field):
```
   +tcp class=BRO_CONN groupby:BRO_CONN.dstip +dstport>=1000 +bytes_in>1000000 | whois
```
Now for something a bit more advanced.  Using the latest version of ELSA, you can check for any high bytes transferred to the outside from hosts that recently had a "TROJAN" Snort alert:
```
   sig_msg:trojan class:snort groupby:srcip | remote > foreach(${remote} class=BRO_CONN groupby:BRO_CONN.dstip +dstport>=1000 +bytes_in>1000000 | whois | filter descr,google)
```
So now we're checking any remote host (as in not on the home network) involved in a Trojan incident for transferring more than a megabyte of data to an IP not owned by Google.

# 1/22/13 - [security-onion] Is it possible to launch ELSA from command line? #

You can use the command-line version of ELSA by navigating to /opt/elsa/web/ (I think that's the right directory on SO) and using the "cli.pl" script.  The "-q" parameter is for query, so it would look like perl cli.pl -q "example.com" and you can use "-f" to change the result format from TSV to JSON.

# 4/16/2013 - [ELSA](ELSA.md) What is the best way to query for a list of all internal RFC1918 hosts sending / receiving traffic outside the US? #

Great question!

If you're using a firewall to create the flow records:
```
host:<my firewall> class:firewall_connection_end srcip>=10.0.0.0 srcip<=10.255.255.255 srcip>=192.168.0.0 srcip<=192.168.255.255 srcip>=172.16.0.0 srcip<=172.16.255.255 groupby:dstip | geoip | filter(cc,us)
```
You will probably want to run a prior search for just
```
 srcip>=10.0.0.0 srcip<=10.255.255.255 srcip>=192.168.0.0 srcip<=192.168.255.255 srcip>=172.16.0.0 srcip<=172.16.255.255
```
Then save that as a saved search and give it the name rfc1918.  Then, you can reduce the above query to this:
```
host:<my firewall> class:firewall_connection_end $rfc1918 groupby:dstip | geoip | filter(cc,us)
```
If you leave off the "host:<my firewall>" term, then it will still work but only search temp indexes.

For data extrusion, one of my favorites is to do something similar, but with HTTP POST's:
```
+method:post $rfc1918 groupby:dstip | geoip | filter(cc,us)
```
Or Java user agents:
```
+user_agent:java $rfc1918 groupby:dstip | geoip | filter(cc,us)
```