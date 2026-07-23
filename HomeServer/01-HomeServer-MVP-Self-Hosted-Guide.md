# My MVP Self-Hosted Guide

## Scope

This document is the practical companion to my main plan:

- [HomeServer-Development-Plan.md](C:/Users/chg_i/Documents/The-Everything-shelf/HomeServer/HomeServer-Development-Plan.md)

This guide is focused on:

- building the **MVP first**
- choosing the right **self-hosted services**
- installing things in the right order
- protecting my data from day one
- avoiding homelab mistakes that create noise without creating value

This guide assumes:

- the date is **May 15, 2026**
- my first server is a **ZimaBoard 2 16GB**
- I start with **2 x 16TB CMR HDDs**
- I want a **Google replacement first**
- I want a path toward **TrueNAS SCALE**, **GPU compute**, and **10GbE** later

## The MVP Philosophy

My MVP is not supposed to be impressive.

It is supposed to be:

- reliable
- quiet
- easy to recover
- remotely accessible
- useful every day

If I get those five things right, the rest of the project becomes much easier.

The MVP is not the place for:

- Kubernetes
- multi-node clustering
- public reverse proxies
- complicated SSO
- experimental storage layering
- mixing router, NAS, and AI workloads in one fragile box

## What I Am Actually Building First

For the MVP, I want one small box that gives me immediate value:

- automatic phone photo backup
- private file sync
- remote access
- simple SMB shares
- DNS filtering for the house later
- basic monitoring
- basic backup discipline

I do **not** need to deploy every app I have ever heard of.

## My Recommended MVP Stack

### Install now

- **Ubuntu Server 24.04 LTS** on the ZimaBoard 2
- **ZFS mirror** for the two 16TB drives
- **Docker Engine + Compose plugin**
- **Tailscale**
- **Immich**
- **Syncthing**
- **File Browser**
- **Uptime Kuma**
- **Vaultwarden**

### Add soon after the MVP is stable

- **AdGuard Home**
- **n8n**
- **Paperless-ngx**
- **Plex** if the box still has enough headroom

### Add later on better hardware

- **Home Assistant** on a dedicated box or mini PC
- **TrueNAS SCALE**
- **Ollama**
- **Open WebUI**
- **ComfyUI**
- **Nextcloud AIO** if I truly need its collaboration features

## Why I Am Not Starting With Nextcloud

This is important.

A lot of people say "Google replacement" and immediately install Nextcloud.

I think that is often the wrong first step.

For my MVP:

- **Immich** is better for photos
- **Syncthing** is lighter and simpler for file sync
- **File Browser** is lighter for quick web file access

I should add **Nextcloud** later only if I specifically want:

- calendars
- contacts
- web-based file management
- easy share links
- document collaboration
- a more integrated "suite" feel

That keeps the MVP smaller, faster, and easier to debug.

## Hardware Checklist for the MVP

### Required

- ZimaBoard 2 16GB
- 2 x 16TB **CMR** drives
- stable 12V power supply that can handle dual 3.5" drive spin-up
- Ethernet cable
- small UPS
- USB keyboard and display for initial setup if needed
- another machine to prepare the installer USB

### Strongly recommended

- drive labels or tape labels
- one external USB drive for backups
- spare SATA/data cable if not already included
- notebook or markdown file with every password and network choice

## Recommended Host OS

I am choosing **Ubuntu Server 24.04 LTS** for the MVP.

### Why not keep the default appliance OS?

The ZimaBoard 2 ships with ZimaOS by default, and that is a valid option if I want the appliance experience.

I am still choosing Ubuntu Server for this guide because:

- Docker’s official Linux installation path is straightforward on Ubuntu
- most self-hosted documentation assumes Ubuntu or Debian
- I get maximum control
- I learn the stack I will actually rely on later

### Why not install TrueNAS first?

Because the ZimaBoard MVP is my learning and service box, not the final storage platform.

I want to keep the MVP simple and flexible.

## Network and Naming Plan Before I Start

Before I install anything, I want to decide a few names.

### Hostname

I will call the MVP host:

```text
homeserver-mvp
```

### Local domain

I will use:

```text
home.arpa
```

This is the reserved domain for home networks.

### Service names

I will keep service names predictable:

```text
immich.home.arpa
sync.home.arpa
files.home.arpa
kuma.home.arpa
vault.home.arpa
adguard.home.arpa
n8n.home.arpa
```

### Static IP

I will choose one fixed address on my LAN, for example:

```text
192.168.1.20
```

I should either:

- reserve it on the router, or
- set it statically on the server

I should not leave my main server on a random DHCP lease and then build everything around a moving IP.

## Storage Design for the MVP

### My rule

The two 16TB drives will be a **mirror**, not two separate disks.

### Why a mirror

Because I want:

- simple recovery
- protection from one drive failing
- straightforward replacement workflow

### Critical warning

A mirror is **not** a backup.

If I:

- delete a file
- encrypt my data with malware
- misconfigure an app
- overwrite something by mistake

the mirror will happily preserve the mistake on both drives.

That is why I also need:

- snapshots
- an external backup target
- later, an offsite copy for my irreplaceable data

## Filesystem Strategy

I will use **ZFS on Ubuntu** for the data pool.

### Why ZFS here

- it aligns with my future TrueNAS direction
- snapshots are excellent
- data integrity features are worth it
- the dataset model makes organization easier

### My pool design

Pool name:

```text
tank
```

Datasets:

```text
tank/appdata
tank/media
tank/photos
tank/documents
tank/backups
tank/shares
tank/docker
tank/archives
```

## Step 1: Assemble the Hardware

### What I do physically

1. Mount or place the ZimaBoard where it has airflow.
2. Connect both 16TB drives.
3. Connect Ethernet.
4. Connect display and keyboard for first boot if needed.
5. Plug the server into the UPS.
6. Plug the UPS into mains power.

### Before I go further

I should label the drives physically:

```text
drive-a
drive-b
```

This sounds trivial until one of them fails two months later.

## Step 2: Prepare the Ubuntu Server Installer

### What I need

- Ubuntu Server 24.04 LTS ISO
- a USB installer
- a tool like Rufus or Balena Etcher on another machine

### BIOS / firmware settings I want

- UEFI boot enabled
- boot from USB first for installation
- restore on AC power loss enabled if available
- secure boot left on only if everything works cleanly

If secure boot causes trouble, I will disable it and document that choice.

## Step 3: Install Ubuntu Server

### Installation choices I want

- install to the internal eMMC or dedicated boot storage
- do **not** install the OS onto the 16TB data drives
- install OpenSSH server during setup
- create a normal admin user
- use a strong password
- if possible, add my SSH public key at install time

### My naming choices during install

```text
hostname: homeserver-mvp
username: myadmin
```

### After first boot

I log in and run:

```bash
sudo apt update && sudo apt full-upgrade -y
sudo reboot
```

## Step 4: Baseline Host Hardening

After the first reboot, I do basic hygiene before I install applications.

### Create or verify my SSH key workflow

From my laptop or desktop:

```bash
ssh-copy-id myadmin@192.168.1.20
```

Then I confirm key-based login works.

### Optional but recommended: disable password SSH later

After I confirm SSH keys work, I can harden SSH by editing:

```text
/etc/ssh/sshd_config
```

Key lines:

```text
PasswordAuthentication no
PermitRootLogin no
```

Then:

```bash
sudo systemctl restart ssh
```

I should only do this after confirming my SSH key login actually works.

### Install useful base packages

```bash
sudo apt install -y \
  curl wget git vim tmux htop smartmontools \
  nvme-cli lsof jq ca-certificates unzip \
  zfsutils-linux samba
```

### Enable SMART monitoring service

```bash
sudo systemctl enable --now smartmontools
```

## Step 5: Set a Static IP

I want the host IP to stay stable.

### Check network interface names

```bash
ip a
```

I note the interface name, often something like:

```text
enp1s0
```

### Edit Netplan

Example file:

```text
/etc/netplan/01-netcfg.yaml
```

Example content:

```yaml
network:
  version: 2
  ethernets:
    enp1s0:
      dhcp4: no
      addresses:
        - 192.168.1.20/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 1.1.1.1
          - 8.8.8.8
```

Then apply it:

```bash
sudo netplan try
sudo netplan apply
```

If my router later runs AdGuard Home, I will point DNS at that instead.

## Step 6: Identify the Data Drives Correctly

Before touching ZFS, I must identify the correct disks.

### List disks

```bash
lsblk -o NAME,SIZE,MODEL,SERIAL
```

### Prefer stable disk IDs

```bash
ls -l /dev/disk/by-id/
```

I want to use the stable `/dev/disk/by-id/...` paths, not `/dev/sda` and `/dev/sdb`, because Linux can reorder drive letters.

### Example

```text
/dev/disk/by-id/ata-WDC_WD161KFGX_XXXX
/dev/disk/by-id/ata-WDC_WD161KFGX_YYYY
```

## Step 7: Create the ZFS Mirror

### Important warning

This destroys data on the selected drives.

I should only continue after triple-checking the drive IDs.

### Optional cleanup of old signatures

```bash
sudo wipefs -a /dev/disk/by-id/ata-WDC_WD161KFGX_XXXX
sudo wipefs -a /dev/disk/by-id/ata-WDC_WD161KFGX_YYYY
```

### Create the pool

```bash
sudo zpool create \
  -o ashift=12 \
  -O acltype=posixacl \
  -O atime=off \
  -O compression=lz4 \
  -O dnodesize=auto \
  -O normalization=formD \
  -O relatime=on \
  -O xattr=sa \
  -O mountpoint=/tank \
  tank mirror \
  /dev/disk/by-id/ata-WDC_WD161KFGX_XXXX \
  /dev/disk/by-id/ata-WDC_WD161KFGX_YYYY
```

### Verify the pool

```bash
zpool status
zfs list
```

## Step 8: Create Datasets

I want clean separation between app data, user files, media, and backup content.

### Create datasets

```bash
sudo zfs create tank/appdata
sudo zfs create tank/docker
sudo zfs create tank/photos
sudo zfs create tank/documents
sudo zfs create tank/media
sudo zfs create tank/backups
sudo zfs create tank/shares
sudo zfs create tank/archives
```

### Optional quotas

If I want to stop one app from consuming the whole pool, I can add quotas.

Example:

```bash
sudo zfs set quota=4T tank/photos
sudo zfs set quota=4T tank/media
```

### Useful mountpoints

I can keep the default mountpoints or define clearer paths:

```bash
sudo zfs set mountpoint=/srv/appdata tank/appdata
sudo zfs set mountpoint=/srv/docker tank/docker
sudo zfs set mountpoint=/srv/photos tank/photos
sudo zfs set mountpoint=/srv/documents tank/documents
sudo zfs set mountpoint=/srv/media tank/media
sudo zfs set mountpoint=/srv/backups tank/backups
sudo zfs set mountpoint=/srv/shares tank/shares
sudo zfs set mountpoint=/srv/archives tank/archives
```

I will use those mountpoints in the rest of this guide.

## Step 9: Create Base Directories

```bash
sudo mkdir -p /srv/appdata/{immich,adguard,syncthing,vaultwarden,uptime-kuma,filebrowser,n8n}
sudo mkdir -p /srv/docker/{stacks,env}
sudo mkdir -p /srv/shares/{family,exchange,projects}
sudo mkdir -p /srv/backups/{usb,pc-clients,server}
sudo mkdir -p /srv/photos/library
sudo mkdir -p /srv/media/{movies,shows,music}
```

### Ownership strategy

I do not want to `chmod 777` everything.

Start with:

```bash
sudo chown -R myadmin:myadmin /srv/appdata /srv/docker
sudo chown -R nobody:nogroup /srv/shares
sudo chmod -R 775 /srv/appdata /srv/docker /srv/shares
```

I can refine per-service permissions later.

## Step 10: Configure Samba Shares

This gives me easy file access from Windows, macOS, and Linux desktops.

### Backup the config

```bash
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
```

### Add shares

Append these blocks to:

```text
/etc/samba/smb.conf
```

```ini
[family]
   path = /srv/shares/family
   browseable = yes
   writable = yes
   guest ok = no
   valid users = myadmin
   create mask = 0664
   directory mask = 0775

[projects]
   path = /srv/shares/projects
   browseable = yes
   writable = yes
   guest ok = no
   valid users = myadmin
   create mask = 0664
   directory mask = 0775

[exchange]
   path = /srv/shares/exchange
   browseable = yes
   writable = yes
   guest ok = no
   valid users = myadmin
   create mask = 0664
   directory mask = 0775
```

### Add Samba password

```bash
sudo smbpasswd -a myadmin
```

### Restart Samba

```bash
sudo systemctl restart smbd
sudo systemctl enable smbd
```

### Test from another machine

On Windows:

```text
\\192.168.1.20\family
```

## Step 11: Install Docker Engine the Official Way

This guide assumes I follow Docker’s official Ubuntu installation path.

### Remove conflicting packages

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do
  sudo apt remove -y $pkg
done
```

### Add Docker’s official repository

```bash
sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Let my user run Docker

```bash
sudo usermod -aG docker myadmin
newgrp docker
```

### Verify install

```bash
docker run hello-world
docker compose version
```

## Step 12: Move Docker Data Off the Boot Disk

I do not want container layers and volumes filling the small eMMC boot storage.

### Create Docker data path

```bash
sudo mkdir -p /srv/docker/data
```

### Configure Docker daemon

Create or edit:

```text
/etc/docker/daemon.json
```

```json
{
  "data-root": "/srv/docker/data",
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

### Restart Docker

```bash
sudo systemctl restart docker
docker info | grep "Docker Root Dir"
```

It should show:

```text
/srv/docker/data
```

## Step 13: Install Tailscale

I want remote access before I start exposing services to myself.

### Install

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

### Bring the node online

```bash
sudo tailscale up --ssh
```

### What I expect

The command will give me a login URL.

I sign in, authorize the device, and then I should be able to connect using the Tailnet IP or MagicDNS.

### Validate

From another Tailscale device:

```bash
ping homeserver-mvp
ssh myadmin@homeserver-mvp
```

### Important rule

For the MVP, I should prefer:

- local LAN access
- Tailscale access

I should **not** port-forward my apps to the public internet just because I can.

## Step 14: Create a Simple Stack Layout

I want every service in its own folder.

### Example layout

```text
/srv/docker/stacks/immich
/srv/docker/stacks/syncthing
/srv/docker/stacks/filebrowser
/srv/docker/stacks/vaultwarden
/srv/docker/stacks/uptime-kuma
/srv/docker/stacks/adguard
/srv/docker/stacks/n8n
```

## Step 15: Deploy Immich

Immich is my photo platform.

### Why Immich is in the MVP

- mobile auto-upload
- timeline browsing
- face/object search
- private Google Photos replacement

### Use the official Compose release files

```bash
mkdir -p /srv/docker/stacks/immich
cd /srv/docker/stacks/immich
wget -O docker-compose.yml https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env
```

### Edit the `.env` file

At minimum, I want to set:

```text
UPLOAD_LOCATION=/srv/photos/library
DB_DATA_LOCATION=/srv/appdata/immich/postgres
TZ=America/Mexico_City
```

I should also review the generated passwords and not leave placeholders.

### Start Immich

```bash
cd /srv/docker/stacks/immich
docker compose up -d
```

### First login

Open:

```text
http://192.168.1.20:2283
```

Then:

1. create the admin account
2. install the Immich mobile app on my phone
3. point the app to the local or Tailscale-accessible URL
4. enable background upload

### Immediate post-install tasks

- create a second user for my wife
- decide whether libraries are shared or separate
- test upload from both Wi-Fi and Tailscale
- verify photos land in `/srv/photos/library`

## Step 16: Deploy Syncthing

Syncthing is my lightweight file sync tool.

### Why Syncthing is in the MVP

- extremely good at simple file replication
- very fast to set up
- excellent for project folders and device sync
- lighter than Nextcloud

### Compose file

Create:

```text
/srv/docker/stacks/syncthing/compose.yaml
```

```yaml
services:
  syncthing:
    image: lscr.io/linuxserver/syncthing:latest
    container_name: syncthing
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Mexico_City
    ports:
      - "8384:8384"
      - "22000:22000/tcp"
      - "22000:22000/udp"
      - "21027:21027/udp"
    volumes:
      - /srv/appdata/syncthing:/config
      - /srv/shares/family:/data/family
      - /srv/shares/projects:/data/projects
```

### Start it

```bash
cd /srv/docker/stacks/syncthing
docker compose up -d
```

### First setup

Open:

```text
http://192.168.1.20:8384
```

Then:

1. set GUI authentication
2. add my laptop as a device
3. add my desktop as a device
4. create one shared folder first, such as `projects`
5. validate sync behavior before adding more data

## Step 17: Deploy File Browser

This gives me a simple web-based file manager without the weight of Nextcloud.

### Compose file

Create:

```text
/srv/docker/stacks/filebrowser/compose.yaml
```

```yaml
services:
  filebrowser:
    image: filebrowser/filebrowser:latest
    container_name: filebrowser
    restart: unless-stopped
    user: "1000:1000"
    ports:
      - "8081:80"
    volumes:
      - /srv/shares:/srv
      - /srv/appdata/filebrowser/database.db:/database.db
      - /srv/appdata/filebrowser/.filebrowser.json:/.filebrowser.json
```

### Bootstrap files

```bash
touch /srv/appdata/filebrowser/database.db
cat > /srv/appdata/filebrowser/.filebrowser.json <<'EOF'
{
  "address": "",
  "port": 80,
  "baseURL": "",
  "database": "/database.db",
  "log": "stdout",
  "root": "/srv"
}
EOF
```

### Start it

```bash
cd /srv/docker/stacks/filebrowser
docker compose up -d
```

### First login

Open:

```text
http://192.168.1.20:8081
```

The default login is usually:

```text
admin / admin
```

I should change that immediately.

## Step 18: Deploy Vaultwarden

If I want password self-hosting later, Vaultwarden is one of the highest-value additions.

### Why it matters

- secure family password storage
- easy Bitwarden-compatible clients
- very high usefulness per watt

### Compose file

Create:

```text
/srv/docker/stacks/vaultwarden/compose.yaml
```

```yaml
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: unless-stopped
    environment:
      - DOMAIN=http://vault.home.arpa
      - SIGNUPS_ALLOWED=false
      - TZ=America/Mexico_City
    ports:
      - "8082:80"
    volumes:
      - /srv/appdata/vaultwarden:/data
```

### Start it

```bash
cd /srv/docker/stacks/vaultwarden
docker compose up -d
```

### First tasks

1. create admin account
2. disable public signups
3. export old passwords from existing manager if needed
4. enable 2FA

## Step 19: Deploy Uptime Kuma

I want to know when things break.

### Why it belongs early

- helps me spot service instability
- helps me detect storage or app crashes early
- low overhead

### Compose file

Create:

```text
/srv/docker/stacks/uptime-kuma/compose.yaml
```

```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    restart: unless-stopped
    ports:
      - "3001:3001"
    volumes:
      - /srv/appdata/uptime-kuma:/app/data
```

### Start it

```bash
cd /srv/docker/stacks/uptime-kuma
docker compose up -d
```

### First monitors to add

- `http://192.168.1.20:2283` for Immich
- `http://192.168.1.20:8384` for Syncthing
- `http://192.168.1.20:8081` for File Browser
- `http://192.168.1.20:8082` for Vaultwarden
- ping the router
- ping the server itself

## Step 20: Add AdGuard Home

I should add this only after the core services are stable.

### Why it matters

- DNS filtering
- safe search options
- basic parental controls
- better visibility into device queries

### Compose file

Create:

```text
/srv/docker/stacks/adguard/compose.yaml
```

```yaml
services:
  adguardhome:
    image: adguard/adguardhome:latest
    container_name: adguardhome
    restart: unless-stopped
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "3000:3000/tcp"
      - "80:80/tcp"
    volumes:
      - /srv/appdata/adguard/work:/opt/adguardhome/work
      - /srv/appdata/adguard/conf:/opt/adguardhome/conf
```

### Important note

Port 53 is sensitive.

I should deploy AdGuard Home only when:

- the host IP is stable
- I understand which device is handing out DNS on the network
- I am ready to change router DNS settings carefully

### Start it

```bash
cd /srv/docker/stacks/adguard
docker compose up -d
```

### First setup

Open:

```text
http://192.168.1.20:3000
```

Then:

1. complete initial setup
2. point a single test device to AdGuard Home
3. verify browsing still works
4. only then update router DNS for the full house

## Step 21: Add n8n

I do not need n8n on day one, but it fits the end goal.

### Why add it

- automations
- webhook-driven workflows
- media processing workflows later
- integrations between local apps and cloud apps

### My rule

I will add n8n only after:

- storage is stable
- backups exist
- I understand my base services

### Compose file

Create:

```text
/srv/docker/stacks/n8n/compose.yaml
```

```yaml
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:stable
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - TZ=America/Mexico_City
      - GENERIC_TIMEZONE=America/Mexico_City
      - N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
      - N8N_RUNNERS_ENABLED=true
    volumes:
      - /srv/appdata/n8n:/home/node/.n8n
```

### Start it

```bash
cd /srv/docker/stacks/n8n
docker compose up -d
```

### First automations I should build

- alert me when a backup fails
- notify me when disk free space drops below a threshold
- move files from `exchange` into categorized folders
- run a recurring health report

## Service Update Routine

I want a standard update workflow.

For each service stack:

```bash
cd /srv/docker/stacks/<service>
docker compose pull
docker compose up -d
docker image prune -f
```

For Immich, I should read release notes more carefully before major upgrades.

## MVP Backup Strategy

This section matters more than any application.

## My backup layers

### Layer 1: ZFS mirror

This protects me from a single drive failure.

It is **not** enough by itself.

### Layer 2: ZFS snapshots

This protects me from:

- accidental deletion
- accidental overwrite
- some forms of app-level mistakes

### Layer 3: external backup target

This protects me when:

- I destroy the pool
- the host dies
- I lose both pool drives
- malware affects the main storage

### Layer 4: offsite copy later

This protects me from:

- theft
- fire
- flood
- electrical disasters

## Step 22: Enable Basic ZFS Health Checks

### Monthly scrub

I want ZFS to scrub monthly.

Check current status:

```bash
zpool status
```

Run a manual scrub:

```bash
sudo zpool scrub tank
```

Check progress:

```bash
zpool status tank
```

I can schedule this monthly with cron.

Edit root crontab:

```bash
sudo crontab -e
```

Add:

```cron
0 3 1 * * /sbin/zpool scrub tank
```

## Step 23: Create Snapshot Scripts

I want lightweight snapshot automation immediately.

### Script path

Create:

```text
/usr/local/sbin/zfs-snapshot-mvp.sh
```

### Script content

```bash
#!/usr/bin/env bash
set -euo pipefail

STAMP="$(date +%Y-%m-%d-%H%M)"
DATASETS=(
  tank/appdata
  tank/photos
  tank/documents
  tank/shares
)

for ds in "${DATASETS[@]}"; do
  zfs snapshot -r "${ds}@auto-${STAMP}"
done
```

### Make executable

```bash
sudo chmod +x /usr/local/sbin/zfs-snapshot-mvp.sh
```

### Create cleanup script

Create:

```text
/usr/local/sbin/zfs-snapshot-prune-mvp.sh
```

```bash
#!/usr/bin/env bash
set -euo pipefail

KEEP=30

for ds in tank/appdata tank/photos tank/documents tank/shares; do
  zfs list -H -t snapshot -o name -s creation | grep "^${ds}@auto-" | head -n -"${KEEP}" | while read -r snap; do
    zfs destroy -r "$snap"
  done
done
```

### Make executable

```bash
sudo chmod +x /usr/local/sbin/zfs-snapshot-prune-mvp.sh
```

### Add cron entries

```bash
sudo crontab -e
```

Add:

```cron
15 * * * * /usr/local/sbin/zfs-snapshot-mvp.sh
45 3 * * * /usr/local/sbin/zfs-snapshot-prune-mvp.sh
```

### Validate snapshots

```bash
zfs list -t snapshot
```

## Step 24: Add a Real External Backup

This is where I stop pretending snapshots are enough.

### My first backup target

I should use:

- a USB external HDD
- or a second backup disk dock

I should keep this separate from the main pool.

### Mount the external backup drive

Example mountpoint:

```bash
sudo mkdir -p /mnt/usb-backup
```

Format it if it is new and I accept destroying existing data:

```bash
sudo mkfs.ext4 /dev/sdX1
```

Mount it:

```bash
sudo mount /dev/sdX1 /mnt/usb-backup
```

Check it:

```bash
df -h /mnt/usb-backup
```

## Step 25: Create a Simple Server Backup Script

This is my practical first backup for the server itself.

### Create script

```text
/usr/local/sbin/mvp-rsync-backup.sh
```

```bash
#!/usr/bin/env bash
set -euo pipefail

TARGET="/mnt/usb-backup/server-backup"
STAMP="$(date +%Y-%m-%d-%H%M)"

mkdir -p "${TARGET}/latest"
mkdir -p "${TARGET}/snapshots/${STAMP}"

rsync -aHAX --delete \
  /srv/photos/ \
  "${TARGET}/latest/photos/"

rsync -aHAX --delete \
  /srv/documents/ \
  "${TARGET}/latest/documents/"

rsync -aHAX --delete \
  /srv/shares/ \
  "${TARGET}/latest/shares/"

rsync -aHAX --delete \
  /srv/appdata/ \
  "${TARGET}/latest/appdata/"

cp -al "${TARGET}/latest" "${TARGET}/snapshots/${STAMP}" || true
```

### Make executable

```bash
sudo chmod +x /usr/local/sbin/mvp-rsync-backup.sh
```

### Run it manually once

```bash
sudo /usr/local/sbin/mvp-rsync-backup.sh
```

### Schedule it

```bash
sudo crontab -e
```

Add:

```cron
30 2 * * * /usr/local/sbin/mvp-rsync-backup.sh >> /var/log/mvp-backup.log 2>&1
```

### Important note

This is a practical first backup, not my final enterprise-grade design.

It is still much better than having no separate backup at all.

## Step 26: Add Kopia Later for Better Backup Workflows

Once the MVP is stable, I want Kopia in my toolbox.

### Why Kopia is worth adding

- encrypted backups
- deduplication
- versioned snapshots
- good endpoint backup story
- works with local or network-attached storage

### Best use for me

I should use Kopia for:

- laptop backups
- desktop backups
- important project folders
- offsite backup workflows later

### Simple repository idea

On the server:

```text
/srv/backups/pc-clients/kopia-repo
```

Then endpoint devices can back up into that repository over SMB or another mounted path.

### Later, I can move Kopia to:

- the TrueNAS box
- a second backup server
- an offsite target

## Step 27: Test Restore Workflows

Backups I have never restored are not yet trustworthy.

### Test 1: restore from snapshot

1. create a test file in `/srv/shares/family`
2. confirm an hourly snapshot happens
3. delete the file
4. browse snapshot metadata
5. restore the file

Helpful commands:

```bash
zfs list -t snapshot
```

### Test 2: restore from USB backup

1. copy a file into `/srv/documents`
2. run the backup script
3. delete the file from the main pool
4. restore it from `/mnt/usb-backup/server-backup/latest/documents`

### Test 3: simulate appdata recovery

1. stop a small container
2. copy back its config from the backup
3. restart the container
4. verify it works

This is the kind of drill that prevents future panic.

## Step 28: Add Notifications

I do not want silent failures.

### Minimum alerting

- Uptime Kuma monitor notifications
- email or Telegram alerts for backup script failure
- SMART health checks reviewed weekly

### Easy first approach

Use n8n or Uptime Kuma notification integrations once the stack is stable.

## My Recommended Self-Hosted Services by Goal

This section is a map of what to add and when.

## Goal: Google Photos replacement

### Best fit

- **Immich**

### Why

- best photo-first experience
- mobile upload
- strong search and timeline features

### Add now or later

- **Now**

## Goal: Google Drive replacement

### Best fit for MVP

- **Syncthing**
- **File Browser**

### Better full-suite fit later

- **Nextcloud AIO**

### My practical recommendation

Start with Syncthing and File Browser.

Add Nextcloud only when I genuinely need the suite behavior.

## Goal: family documents and paper archive

### Best fit

- **Paperless-ngx**

### Why

- scanned document management
- OCR
- tags and archive workflow

### When to add

- after backups and base services are working

## Goal: passwords and family credentials

### Best fit

- **Vaultwarden**

### When to add

- early, after Tailscale and backups exist

## Goal: personal app hosting

### Best fit

- Docker Compose on the MVP now
- dedicated app host or TrueNAS apps later

### Good app candidates

- Uptime Kuma
- File Browser
- Vaultwarden
- n8n
- Paperless-ngx

## Goal: automations

### Best fit

- **n8n**

### Good first workflows

- backup notifications
- media processing
- upload organization
- smart home alerts later

## Goal: Google Home integration

### Best fit

- **Home Assistant**

### Recommendation

I should not force Home Assistant onto the MVP if the box already handles photo and file workflows.

When I add Home Assistant, I should strongly consider giving it:

- its own mini PC
- or its own small dedicated system

## Goal: media server

### Best fit

- **Plex**

### Recommendation

Add Plex only if:

- the ZimaBoard still has headroom
- the content library is organized
- I am willing to tune transcoding expectations

For a family media setup, Plex may eventually belong on the TrueNAS box if I choose an Intel CPU with iGPU.

## Goal: local AI and LLMs

### Best fit later

- **Ollama**
- **Open WebUI**
- **ComfyUI**

### Recommendation

Do not attempt serious local AI on the MVP ZimaBoard.

Keep AI for the future GPU compute node.

## Goal: monitoring and operational visibility

### Best fit

- **Uptime Kuma**
- optional later: **Beszel**, **Grafana**, **Prometheus**

### My recommendation

Start with Uptime Kuma.

Do not introduce a full metrics stack until I actually need it.

## Goal: identity and access control

### Best fit later

- **Authentik**
- or **Authelia**

### Recommendation

This is valuable, but not MVP material.

I should only add it after I already have multiple mature services and understand their access model.

## Services I Recommend Avoiding in the MVP

- Kubernetes
- Ceph
- Gluster
- public internet exposure of every app
- email self-hosting
- multiple reverse proxies at once
- a random pile of dashboards that solve no problem

## My Suggested Order of Operations

If I want the best chance of success, I should install in this order:

1. Ubuntu Server
2. updates, SSH, static IP
3. ZFS mirror
4. Samba
5. Docker
6. move Docker root to the data pool
7. Tailscale
8. Immich
9. Syncthing
10. File Browser
11. Uptime Kuma
12. snapshot scripts
13. USB backup workflow
14. Vaultwarden
15. AdGuard Home
16. n8n
17. Paperless-ngx
18. Plex if justified

## Expansion Path From MVP to Main Plan

The MVP should eventually hand off responsibilities, not try to keep them forever.

### What stays valid later

- Tailscale
- Immich
- Syncthing
- Vaultwarden
- Uptime Kuma
- backup discipline

### What may migrate later

- photo library
- media library
- SMB shares
- n8n
- document archive

### Best migration target

- dedicated **TrueNAS SCALE** storage box for data
- separate **GPU node** for AI and rendering
- separate **router/firewall** for network control

## Recommended Documentation I Should Keep As I Build

I should maintain one markdown file or notebook with:

- hostnames
- IP addresses
- drive serial numbers
- which SATA cable goes to which drive
- what is stored where
- what my backup schedule is
- restore procedure notes
- admin passwords and 2FA recovery details stored safely

This sounds boring, but it is what turns a project into infrastructure.

## My Final MVP Definition of Done

My MVP is complete when all of these are true:

- the server has a stable hostname and static IP
- I can SSH into it reliably
- the 2 x 16TB mirror is healthy
- Immich uploads from my phone automatically
- Syncthing replicates at least one important folder
- I have simple SMB shares
- Tailscale remote access works
- Uptime Kuma watches the important services
- snapshots run automatically
- a separate USB backup completes successfully
- I have already performed at least one restore test

If I do all of that, I will already have a serious and useful home server.

## Source Links

These are the main official references I used to shape this guide:

- Docker Engine on Ubuntu  
  https://docs.docker.com/engine/install/ubuntu/

- Docker Compose install overview  
  https://docs.docker.com/compose/install/

- Tailscale install  
  https://tailscale.com/docs/install

- Tailscale install on Linux  
  https://tailscale.com/docs/install/linux

- Immich install  
  https://docs.immich.app/install

- ZimaBoard 2 getting started  
  https://www.zimaspace.com/docs/zimaboard/power-on-zimaboard-2

- ZimaBoard 2 product page  
  https://www.zimaspace.com/en/products/single-board2-server

- n8n Docker install  
  https://docs.n8n.io/hosting/installation/docker/

- n8n Docker Compose setup  
  https://docs.n8n.io/hosting/installation/server-setups/docker-compose/

- Nextcloud All-in-One  
  https://github.com/nextcloud/all-in-one

- Home Assistant generic x86-64 install  
  https://www.home-assistant.io/installation/generic-x86-64/

- Syncthing docs  
  https://docs.syncthing.net/

- Uptime Kuma  
  https://github.com/louislam/uptime-kuma

- Kopia docs  
  https://kopia.io/docs/

- Kopia installation  
  https://kopia.io/docs/installation/

- Kopia repositories  
  https://kopia.io/docs/repositories/
