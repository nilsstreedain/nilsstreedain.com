# unbound + pi-hole + AutoUpdating BlockLists

Before getting started:
- Create a fresh install of Raspbian (or your prefered distro) with ssh enabled
- Connect your Raspberry Pi (or whatever computer you're using) to your network
- ssh in the Pi

## Update Raspberry Pi

```bash
sudo apt update
```

```bash
sudo apt full-upgrade
```

## Change Raspberry Pi Password

```bash
passwd
```

### Optional:

```bash
sudo raspi-config
```

Set Raspberry Pi Country (raspi-config > Localisation Options > WLAN Country)

Change Raspberry Pi Hostname (raspi-config > System Options > Hostname)

## Setup Pi-Hole

Install Pi-Hole and follow the steps in the user interface

```bash
sudo curl -sSL https://install.pi-hole.net | bash
```

Change default Pi-Hole password

```bash
sudo pihole -a -p
```

## Setup Unbound

Install Unbound

```bash
sudo apt install unbound
```
Update the list of primary root servers

```bash
wget https://www.internic.net/domain/named.root -qO- | sudo tee /var/lib/unbound/root.hints
```

### Configure unbound

Open unbound configuration

```bash
sudo nano /etc/unbound/unbound.conf.d/pi-hole.conf
```

Paste the following:

```yml
server:
    # If no logfile is specified, syslog is used
    # logfile: "/var/log/unbound/unbound.log"
    verbosity: 0

    interface: 127.0.0.1
    port: 5335
    do-ip4: yes
    do-udp: yes
    do-tcp: yes

    # May be set to yes if you have IPv6 connectivity
    do-ip6: no

    # You want to leave this to no unless you have *native* IPv6. With 6to4 and
    # Terredo tunnels your web browser should favor IPv4 for the same reasons
    prefer-ip6: no

    # Use this only when you downloaded the list of primary root servers!
    # If you use the default dns-root-data package, unbound will find it automatically
    #root-hints: "/var/lib/unbound/root.hints"

    # Trust glue only if it is within the server's authority
    harden-glue: yes

    # Require DNSSEC data for trust-anchored zones, if such data is absent, the zone becomes BOGUS
    harden-dnssec-stripped: yes

    # Don't use Capitalization randomization as it known to cause DNSSEC issues sometimes
    # see https://discourse.pi-hole.net/t/unbound-stubby-or-dnscrypt-proxy/9378 for further details
    use-caps-for-id: no

    # Reduce EDNS reassembly buffer size.
    # Suggested by the unbound man page to reduce fragmentation reassembly problems
    edns-buffer-size: 1472

    # Perform prefetching of close to expired message cache entries
    # This only applies to domains that have been frequently queried
    prefetch: yes

    # One thread should be sufficient, can be increased on beefy machines. In reality for most users running on small networks or on a single machine, it should be unnecessary to seek performance enhancement by increasing num-threads above 1.
    num-threads: 1

    # Ensure kernel buffer is large enough to not lose messages in traffic spikes
    so-rcvbuf: 1m

    # Ensure privacy of local IP ranges
    private-address: 192.168.0.0/16
    private-address: 169.254.0.0/16
    private-address: 172.16.0.0/12
    private-address: 10.0.0.0/8
    private-address: fd00::/8
    private-address: fe80::/10
```

<!-- Unbound can come with a service enabled by default that uses 127.0.0.1 instead of 127.0.0.1#5335 when writing into a file that is used for local services. To disable it, do the following:

```bash
sudo systemctl disable unbound-resolvconf.service
sudo systemctl stop unbound-resolvconf.service
sudo systemctl restart dhcpcd
```

-->

Finally, restart unbound

```bash
sudo service unbound restart
```

Use the web admin panel to change the Pi-Hole upstream DNS to 127.0.0.1#5335 for IPv4 and ::1#5335 for IPv6

### Update root.hints monthly (Not Really Needed)

Create a monthly cron job called updateroothints

```bash
sudo nano /etc/cron.monthly/updateroothints
```

Paste the following:

```bash
#!/bin/sh

# update unbound root list
wget https://www.internic.net/domain/named.root -qO- | sudo tee /var/lib/unbound/root.hints
```

Make it executable

```bash
sudo chmod +x /etc/cron.monthly/updateroothints
```

## Setup Auto-Updating BlockLists

Install pihole-updatelists and it's dependacies

```bash
sudo apt-get install php-cli php-sqlite3 php-intl php-curl
```

```bash
wget -O - https://raw.githubusercontent.com/jacklul/pihole-updatelists/master/install.sh | sudo bash
```

Configure pihole-updatelists

```bash
sudo nano /etc/pihole-updatelists.conf
```

Blacklists (exact):
- Very Safe - No false positive (What I Recommend): `https://v.firebog.net/hosts/lists.php?type=tick`
- Somewhat Safe - Rare false positives (What I use): `https://v.firebog.net/hosts/lists.php?type=nocross`

Blacklists (regex):
- Some false positives, whitelist recommended: `https://raw.githubusercontent.com/mmotti/pihole-regex/master/regex.list`
- Blocks TikTok domains: `https://raw.githubusercontent.com/llacb47/mischosts/master/social/tiktok-regex.list`

Whitelist (exact):
- Recommended Whitelist: `https://raw.githubusercontent.com/anudeepND/whitelist/master/domains/whitelist.txt`
- My Whitelist:
`https://raw.githubusercontent.com/nilsstreedain/pihole-whitelist/main/exact.txt`

Whitelist (regex):
- My Whitelist:
`https://raw.githubusercontent.com/nilsstreedain/pihole-whitelist/main/regex.txt`

Update pi-hole lists

```bash
sudo pihole-updatelists
```

### Update pi-hole lists daily

Create a daily cron job called updatelists

```bash
sudo nano /etc/cron.daily/updatelists
```

Paste the following:

```bash
#!/bin/sh

# update Pi-Hole lists
sudo pihole-updatelists
```

Make it executable

```bash
sudo chmod +x /etc/cron.daily/updatelists
```

<!-- ### Update pi-hole itself weekly (Not Recommended)

Create a weekly cron job called updatepihole

```bash
sudo nano /etc/cron.weekly/updatepihole
```

Paste the following:

```bash
#!/bin/sh

# update Pi-Hole
pihole -up
```

Make it executable

```bash
sudo chmod +x /etc/cron.weekly/updatepihole
```

-->
