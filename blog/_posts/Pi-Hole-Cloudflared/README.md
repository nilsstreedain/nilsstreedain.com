# cloudflared + pi-hole + AutoUpdating BlockLists

Before getting started:
- Create a fresh install of Raspbian (or your prefered distro) with ssh enabled
- Connect your Raspberry Pi (or whatever computer you're using) to your network
- ssh into the Pi

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

## Setup Cloudflared

Install Cloudflared

```bash
wget https://bin.equinox.io/c/VdrWdbjqyF/cloudflared-stable-linux-arm.tgz
tar -xvzf cloudflared-stable-linux-arm.tgz
sudo cp ./cloudflared /usr/local/bin
sudo chmod +x /usr/local/bin/cloudflared
cloudflared -v
```

### Configure Cloudflared

Create/open Cloudflared configuration

```bash
sudo mkdir /etc/cloudflared/
sudo nano /etc/cloudflared/config.yml
```

Paste the following:

```yml
proxy-dns: true
proxy-dns-port: 5053
proxy-dns-upstream:
  - https://1.1.1.1/dns-query
  - https://1.0.0.1/dns-query
  #Uncomment following if you want to also want to use IPv6 for  external DOH lookups
  #- https://[2606:4700:4700::1111]/dns-query
  #- https://[2606:4700:4700::1001]/dns-query
```

Finally, install and start Cloudflared Service

```bash
sudo cloudflared service install --legacy
sudo systemctl start cloudflared
sudo systemctl status cloudflared
```

### Update Cloudflared weekly

Create a monthly cron job called updatecloudflared

```bash
sudo nano /etc/cron.weekly/updatecloudflared
```

Paste the following:

```bash
#!/bin/sh

# update Cloudflared root list
sudo cloudflared update
sudo systemctl restart cloudflared
```

Make it executable

```bash
sudo chmod +x /etc/cron.weekly/updatecloudflared
```

## Setup Pi-Hole

Install Pi-Hole and follow the steps in the user interface

Make sure to set the upstream DNS to 127.0.0.1#5053 for IPv4 and ::1#5053 for IPv6

```bash
sudo curl -sSL https://install.pi-hole.net | bash
```

Change default Pi-Hole password

```bash
sudo pihole -a -p
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

## Pi-Hole Beta

```bash
pihole checkout ftl release/v5.9
```

```bash
pihole checkout core release/v5.4
```

```bash
pihole checkout web release/v5.6
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
