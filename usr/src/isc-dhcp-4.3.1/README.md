# isc-dhcp-client

## Problems

> dhclient process is listening on two randomly chosen udp ports in addition to the usual port 68.
this appears to be a bug in the discovery code for probing information on interfaces in the system.

-

> dhclient process is always listening udp6 when ipv6 is disable in /etc/dhcp/dhclient.conf

-
* https://bugs.launchpad.net/ubuntu/+source/isc-dhcp/+bug/1176046

## Workaround

need to re-compile from the source

* http://forums.debian.net/viewtopic.php?f=10&t=95273&p=495605#p495605


## Re-compile from source

```bash
pushd /usr/src
apt-get update
apt-get -y -qq install build-essential fakeroot devscripts < /dev/null
apt-get -y -qq build-dep isc-dhcp < /dev/null
apt-get -y source isc-dhcp-client < /dev/null
cd isc-dhcp*
grep "--disable-tracing" debian/rules &> /dev/null || NB='          ' &&
sed -i "/--sysconfdir=\/etc\/dhcp \\\\$/a \\${NB}\--disable-tracing \\\\\n${NB}--disable-failover \\\\" debian/rules
sed -i 's/^#define NSUPDATE$/\/* #define NSUPDATE *\//' includes/site.h
DEBEMAIL=devopstuff@gmail.com DEBFULLNAME=@devopstuff dch -l +local 'Rebuilt from sources'
debuild -b -uc -us
dpkg -i ../$(dpkg-query -W -f='${binary:Package}'_"$(head -1 debian/changelog | awk -F'[()]' '{print $2}')"_'${Architecture}' isc-dhcp-common).deb
dpkg -i ../$(dpkg-query -W -f='${binary:Package}'_"$(head -1 debian/changelog | awk -F'[()]' '{print $2}')"_'${Architecture}' isc-dhcp-client).deb
popd
/etc/init.d/networking restart
netstat -tulnp | grep dhclient
```

### Before
* netstat -tulnp | grep dhclient
```bash
root@srv-1:~# netstat -tulnp | grep dhclient
udp        0      0 0.0.0.0:68              0.0.0.0:*                           423/dhclient
udp        0      0 0.0.0.0:3332            0.0.0.0:*                           423/dhclient
udp6       0      0 :::36127                :::*                                423/dhclient
```

* debian/rules

```
CONFFLAGS=--prefix=/usr \
          --sysconfdir=/etc/dhcp \
          --with-srv-lease-file=/var/lib/dhcp/dhcpd.leases \
          --with-srv6-lease-file=/var/lib/dhcp/dhcpd6.leases \
          --with-cli-lease-file=/var/lib/dhcp/dhclient.leases \
          --with-cli6-lease-file=/var/lib/dhcp/dhclient6.leases \
```
* includes/site.h
```
#define NSUPDATE
```
### After
* netstat -tulnp | grep dhclient
```bash
root@srv-1:~# netstat -tulnp | grep dhclient
udp        0      0 0.0.0.0:68              0.0.0.0:*                           423/dhclient
```

* debian/rules
```
CONFFLAGS=--prefix=/usr \
          --sysconfdir=/etc/dhcp \
          --disable-tracing \
          --disable-failover \
          --with-srv-lease-file=/var/lib/dhcp/dhcpd.leases \
          --with-srv6-lease-file=/var/lib/dhcp/dhcpd6.leases \
          --with-cli-lease-file=/var/lib/dhcp/dhclient.leases \
          --with-cli6-lease-file=/var/lib/dhcp/dhclient6.leases \
```
* includes/site.h
```
/* #define NSUPDATE */
```

## Notes

apt-get need to be re-compiled if you want to copy/paste without redirect stdin to /dev/null

* http://serverfault.com/questions/342697/prevent-sudo-apt-get-etc-from-swallowing-pasted-input-to-stdin
* https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=728775

## Credits

* @devopstuff

## License

* nothing to report
