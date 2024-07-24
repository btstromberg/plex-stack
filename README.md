# Plex Stack

## Intruduction

This is a complete Plex setup guide using Docker, with all the necessary tools for automating download/organization of libraries etc.

#### Why use Docker?

Docker runs software in containers, each container has it's own file system where the software is installed, together with any dependencies, services etc in a separate standardized environment that can be run on basically any server. 

If I want to migrate my containers to a new server, all I have to do is copy a single folder (containing the compose config file and the data directory for the containers) to the new server, install docker on it and run the compose commmand, and I don't have to install anything at all really.

### Software Used
- [Plex Media Server](https://github.com/linuxserver/docker-plex)
- [Sonarr](https://github.com/Sonarr/Sonarr) - TV Shows automation/organizing/file management
- [Radarr](https://github.com/Radarr/Radarr) - Movies automation/organizing/file management
- [Tautulli](https://github.com/Tautulli/Tautulli) - Plex usage/performance monitoring system
- [Organizr](https://github.com/causefx/Organizr) - Single web portal that embeds other web UI's, such as Sonarr, Radarr etc
- [Portainer](https://github.com/portainer/portainer) - Graphical Docker container management tool
- [Qbittorrent](https://github.com/qbittorrent/qBittorrent) - Torrent client
- [Overseerr](https://github.com/sct/overseerr) - Web portal where users can log in with their Plex account and request movies/shows. Plex/Radarr/Sonarr integration
- [Bazarr](https://github.com/morpheus65535/bazarr) - Automatic subtitle management/download
- [Prowlarr](https://github.com/Prowlarr/Prowlarr) - Integrates with Radarr/Sonarr to manage their indexer/client settings in one place.
- [Kometa](https://github.com/Kometa-Team/Kometa) (aka Plex Meta Manager) - Tool for customizing Plex library metadata. For example custom posters, collections, etc.


## Requirements

* Linux
* Docker (Engine)
* Public IP address (for external access to Plex)

## Preparation

### Network

You must have a public IP address in order to access Plex over the internet using this guide. The easiest way to check, is to log in to your router and look at it's WAN/Internet address. If it starts with anything <b>other</b> than these, you have a public IP address:

* 10.X.X.X
* 172.16.X.X - 172.31.X.X
* 192.168.X.X
* 100.X.X.X

Once you know you have a public IP, you need to open a port to your servers local IP (port forwarding). Plex uses port 32400 TCP. This is an example how to do it in an Asus router:

<img>![portforward](./docs/assets/pf.png)</img>

### Software installation

#### Docker Engine
Visit https://docs.docker.com/engine/install/ for details on how to install Docker on your distribution. If you are using Debian, take a look at the TLDR guide at the end.

This configuration uses extension fields and environment variables which can be a bit confusing if you are new to Docker, but it really helps keeping everything organized and standardized, especially when expanding with more containers in the future.

1. Clone the repo:
<br><br>
```
cd /opt
git clone https://github.com/btstromberg/plex-stack.git
cd plex-stack
```

2. Now make a copy of the example env file and open it in a text editor:

```
cp .env.example .env
nano .env
```

3. Change the config according to your needs. The last line is for the Plex token, keep in mind that it is only valid for 4 minutes after generation (you can generate a new one if it expires too quickly)

4. Start the stack.:

```
docker compose -f /opt/plex-stack/docker-compose.yml up --detach
```

And remove the included example env file:

```
rm .env.example
```

5. Go to `http://localipofserver:32400/web` and log in with your Plex account to claim the server

6. Go to Settings > Remote Access and verify connectivity (green lock)

## TLDR installation guide (Debian 12)

```
su root
apt update
apt install sudo
sudo usermod -aG sudo username
su username

sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc


echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo usermod -aG docker $USER

cd /opt
sudo git clone https://github.com/btstromberg/plex-stack.git
cd plex-stack
sudo mv .env.example .env
sudo vi .env
sudo docker compose up --detach
```

### Configuration

All of these programs needs to be configured, which is outside the scope of this guide. [TRaSH-Guides](https://trash-guides.info/) has great detailed guides for setting everything up.

Almost all of them are configured via web GUIs. Organizr is a great tool that can be used to organize all links into a single web portal. Each web gui is bound to it's own port on the host in this example, using unencrypted http. Therefore they should not be reachable from the internet via port forwarding. A reverse proxy can be used to proxy all traffic to each service with domain names, HTTPS, additional authentication etc. Traefik is the proxy I prefer, but setting it up is something that is also outside the scope of this guide at the moment.

### Troubleshooting

Portainer is a great graphical tool that you can use to manage/monitor your containers. It can be reached via http://ipofserver:9000/. You will have to create a password for the admin account, then you should see your environment under home. Here you can read logs, restart containers, recreate, remove etc.

Indentation is also important when working with .yml files, usually docker compose will be able to tell you which line is wrong but incorrect indentation can lead to unexpected/weird errors so watch out for that when copying/pasting stuff for example.

If your server says it's unavailable remotely, it's probably due to one of these things:

* You are behind a CGNAT connection, which means you share one public IP with many others, and can't forward ports yourself. CGNAT addresses start with 100. Some ISPs can give you a real public IP if you request it, often for free.

* You have not forwarded ports correctly. Check your routers port forwarding settings.

* You are double NATed. You sit behind a router, which has it's WAN port connected to another router (sometimes the ISPs depending on country).

You can easily test port connectivity using telnet. [Here](https://kb.synology.com/vi-vn/DSM/tutorial/Whether_TCP_port_is_open_or_closed) is a link explaining how to do it.

If telnet can connect directly to the servers local IP on TCP 32400, the issue is probably the router settings, either your own or some upstream ISP router.