# SeedboxGuide
A step by step guide to setting up your unmananged VPS as a seedbox with Plex, Google Enterprise Drive for storage, qbittorrent for torrent client, Jackett for Indexing, Radarr for automating movie downloads, Sonarr for automating TV Shows downloads. (Docker to manage all the installations)

## Initial Setup

### Updating packages
- SSH into your VPS using `ssh username@hostname` and input your password when prompted.
- `apt update` to update and after the update is done `apt upgrade` to upgrade the updated packges.

### Setup fail2ban
- `apt install -y fail2ban` to install fail2ban on your vps.
- `systemctl enable fail2ban` to allow fail2ban to persist on boot.
- `systemctl start fail2ban` to start.

### Make a new user account (non-root)
- `adduser username` to create a new user. (Remember to replace username with your desired username)
- `usermod -aG sudo username` to allow the above user to add to the sudo group.
- `su username` to switch to the newly created user.
- `sudo usermod -m -d /home/new_directory username` to create a new home directory for the user. (Not always required)
  
## Installing Rclone & setting up Google Enterprise Drive for storage
- `sudo apt install rclone` to install rclone.
- To setup a remote, please refer to [Rclone Config Documentation](https://rclone.org/commands/rclone_config/) for more details.

## Mounting 
- Before mounting, you will need to create a folder for rclone to mount the remote to. `mkdir directoryname` to make a new directory. 
- To mount, create a new screen to ensure the rclone continues to run in the background using the command `screen -S rclone`
- `rclone mount --buffer-size=32M --dir-cache-time=84h --vfs-cache-mode=minimal --vfs-cache-max-age=6h remote: ~/directoryToMount/ -vv` to mount the remote.
- `df -h` to view the available space. You can check if the rclone has mounted successfully. `ls /directory` can be used as well to see if there are files loaded.

## Installing Docker
- `sudo apt install docker.io` to install docker.
- `docker run -d \
  --name=qbittorrent \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Etc/UTC \
  -e WEBUI_PORT=8080 \
  -p 8080:8080 \
  -p 6881:6881 \
  -p 6881:6881/udp \
  -v ~/data/appdata/qbittorrent/config:/config \
  -v ~/data/downloads:/downloads \
  --restart unless-stopped \
  lscr.io/linuxserver/qbittorrent:latest` to install qbittorrent.
- You can access it from `http://youripaddress:8080`.
- The username is `admin` and password is `adminadmin`. It is highly recommend to change the password in the setting.

## Installing Jackett
-`docker run -d \
  --name=jackett \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Etc/UTC \
  -e AUTO_UPDATE=true `#optional` \
  -e RUN_OPTS= `#optional` \
  -p 9117:9117 \
  -v ~/data/appdata/jackett/data:/config \
  -v ~/data/appdata/jackett/blackhole:/downloads \
  --restart unless-stopped \
  lscr.io/linuxserver/jackett:latest` to install jackett.
- You can access it from `http://youripaddress:9117`.

## Installing Radarr
-`docker run -d \
  --name=radarr \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Etc/UTC \
  -p 7878:7878 \
  -v ~/data/appdata/radarr/data:/config \
  -v ~/data/:/movies `#optional` \
  -v ~/data/downloads:/downloads `#optional` \
  --restart unless-stopped \
  lscr.io/linuxserver/radarr:latest` to install radarr.
- You can access it from `http://youripaddress:7878`.

## Install Sonarr
- `docker run -d \
  --name=sonarr \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Etc/UTC \
  -p 8989:8989 \
  -v ~/data/appdata/sonarr:/config \
  -v ~/data/:/tv `#optional` \
  -v ~/data/downloads:/downloads `#optional` \
  --restart unless-stopped \
  lscr.io/linuxserver/sonarr:latest` to install sonarr.
- You can access it from `http://youripaddress:8989`.
