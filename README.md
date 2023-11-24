# Servarr Sidekick

This is a fork of [Docker Compose NAS](https://github.com/AdrienPoupa/docker-compose-nas) and aims to support Plex and other tweaks
spcific for my use-cases. This is designed to run on a Raspberry Pi 4, with storage and Plex being provided separately due to the 
relatively low horsepower of the Raspberry Pi 4.

Requirements: Any Docker-capable recent Linux box with Docker Engine and Docker Compose V2.
Tested on Raspberry Pi 4 Ubuntu Server 22.04;

![Docker-Compose NAS Homepage](https://github.com/AdrienPoupa/docker-compose-nas/assets/15086425/3492a9f6-3779-49a5-b052-4193844f16f0)

## Table of Contents

<!-- TOC -->
* [Docker Compose NAS](#docker-compose-nas)
  * [Table of Contents](#table-of-contents)
  * [Applications](#applications)
  * [Quick Start](#quick-start)
  * [Environment Variables](#environment-variables)
  * [PIA WireGuard VPN](#pia-wireguard-vpn)
  * [Sonarr, Radarr & Lidarr](#sonarr-radarr--lidarr)
    * [File Structure](#file-structure)
    * [Download Client](#download-client)
  * [Prowlarr](#prowlarr)
  * [qBittorrent](#qbittorrent)
  * [Homepage](#homepage)
  * [Overseerr](#overseerr)
  * [Traefik and SSL Certificates](#traefik-and-ssl-certificates)
  * [Optional Services](#optional-services)
    * [FlareSolverr](#flaresolverr)
    * [SABnzbd](#sabnzbd)
  * [Customization](#customization)
    * [Optional: Using the VPN for *arr apps](#optional-using-the-vpn-for-arr-apps)
  * [Use Separate Paths for Torrents and Storage](#use-separate-paths-for-torrents-and-storage)
<!-- TOC -->

## Applications

| **Application**                                                      | **Description**                                                                                                                                      | **Image**                                                                                | **URL**      |
|----------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------|--------------|
| [Sonarr](https://sonarr.tv)                                          | PVR for newsgroup and bittorrent users                                                                                                               | [linuxserver/sonarr](https://hub.docker.com/r/linuxserver/sonarr)                        | /sonarr      |
| [Radarr](https://radarr.video)                                       | Movie collection manager for Usenet and BitTorrent users                                                                                             | [linuxserver/radarr](https://hub.docker.com/r/linuxserver/radarr)                        | /radarr      |
| [Lidarr](https://lidarr.audio)                                       | Music collection manager for Usenet and BitTorrent users                                                                                             | [linuxserver/lidarr](https://hub.docker.com/r/linuxserver/lidarr)                        | /lidarr      |
| [Prowlarr](https://github.com/Prowlarr/Prowlarr)                     | Indexer aggregator for Sonarr and Radarr                                                                                                             | [linuxserver/prowlarr:latest](https://hub.docker.com/r/linuxserver/prowlarr)             | /prowlarr    |
| [PIA WireGuard VPN](https://github.com/thrnz/docker-wireguard-pia)   | Encapsulate qBittorrent traffic in [PIA](https://www.privateinternetaccess.com/) using [WireGuard](https://www.wireguard.com/) with port forwarding. | [thrnz/docker-wireguard-pia](https://hub.docker.com/r/thrnz/docker-wireguard-pia)        |              |
| [qBittorrent](https://www.qbittorrent.org)                           | Bittorrent client with a complete web UI<br/>Uses VPN network<br/>Using Libtorrent 1.x                                                               | [linuxserver/qbittorrent:libtorrentv1](https://hub.docker.com/r/linuxserver/qbittorrent) | /qbittorrent |
| [Overseerr](https://overseerr.dev/)                                  | Manages requests for your media library                                                                                                              | [sctx/overseerr](https://hub.docker.com/r/sctx/overseerr)                                | /overseerr   |
| [Homepage](https://gethomepage.dev)                                  | Application dashboard                                                                                                                                | [gethomepage/homepage](https://github.com/gethomepage/homepage/pkgs/container/homepage)  | /            |
| [Traefik](https://traefik.io)                                        | Reverse proxy                                                                                                                                        | [traefik](https://hub.docker.com/_/traefik)                                              |              |
| [Watchtower](https://containrrr.dev/watchtower/)                     | Automated Docker images update                                                                                                                       | [containrrr/watchtower](https://hub.docker.com/r/containrrr/watchtower)                  |              |
| [Autoheal](https://github.com/willfarrell/docker-autoheal/)          | Monitor and restart unhealthy docker containers                                                                                                      | [willfarrell/autoheal](https://hub.docker.com/r/willfarrell/autoheal)                    |              |
| [SABnzbd](https://sabnzbd.org/)                                      | Free and easy binary newsreader                                                                                                                      | [linuxserver/sabnzbd](https://hub.docker.com/r/linuxserver/sabnzbd)                      | /sabnzbd     |
| [FlareSolverr](https://github.com/FlareSolverr/FlareSolverr)         | Optional - Proxy server to bypass Cloudflare protection in Prowlarr                                                                                  | [flaresolverr/flaresolverr](https://hub.docker.com/r/flaresolverr/flaresolverr)          |              |

Optional containers are not run by default, they need to be enabled, 
see [Optional Services](#optional-services) for more information.

## Quick Start

`cp .env.example .env`, edit to your needs then `sudo docker compose up -d`.

For the first time, run `./update-config.sh` to update the applications base URLs and set the API keys in `.env`.

If you want to show Jellyfin information in the homepage, create it in Jellyfin settings and fill `JELLYFIN_API_KEY`.

## Environment Variables

| Variable                       | Description                                                                                                                                                                                            | Default                                          |
|--------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------|
| `COMPOSE_FILE`                 | Docker compose files to load                                                                                                                                                                           | `docker-compose.yml`                             |
| `COMPOSE_PATH_SEPARATOR`       | Path separator between compose files to load                                                                                                                                                           | `:`                                              |
| `USER_ID`                      | ID of the user to use in Docker containers                                                                                                                                                             | `1000`                                           |
| `GROUP_ID`                     | ID of the user group to use in Docker containers                                                                                                                                                       | `1000`                                           |
| `TIMEZONE`                     | TimeZone used by the container.                                                                                                                                                                        | `America/New_York`                               |
| `DATA_ROOT`                    | Host location of the data files                                                                                                                                                                        | `/mnt/data`                                      |
| `DOWNLOAD_ROOT`                | Host download location for qBittorrent, should be a subfolder of `DATA_ROOT`                                                                                                                           | `/mnt/data/torrents`                             |
| `PIA_LOCATION`                 | Servers to use for PIA. [see list here](https://serverlist.piaservers.net/vpninfo/servers/v6)                                                                                                          | `ca` (Montreal, Canada)                          |
| `PIA_USER`                     | PIA username                                                                                                                                                                                           |                                                  |
| `PIA_PASS`                     | PIA password                                                                                                                                                                                           |                                                  |
| `PIA_LOCAL_NETWORK`            | PIA local network                                                                                                                                                                                      | `192.168.0.0/16`                                 |
| `HOSTNAME`                     | Hostname of the NAS, could be a local IP or a domain name                                                                                                                                              | `localhost`                                      |
| `QBITTORRENT_USERNAME`         | qBittorrent username to access the web UI                                                                                                                                                              | `admin`                                          |
| `QBITTORRENT_PASSWORD`         | qBittorrent password to access the web UI                                                                                                                                                              | `adminadmin`                                     |
| `DNS_CHALLENGE`                | Enable/Disable DNS01 challenge, set to `false` to disable.                                                                                                                                             | `true`                                           |
| `DNS_CHALLENGE_PROVIDER`       | Provider for DNS01 challenge, [see list here](https://doc.traefik.io/traefik/https/acme/#providers).                                                                                                   | `cloudflare`                                     |
| `LETS_ENCRYPT_CA_SERVER`       | Let's Encrypt CA Server used to generate certificates, set to production by default.<br/>Set to `https://acme-staging-v02.api.letsencrypt.org/directory` to test your changes with the staging server. | `https://acme-v02.api.letsencrypt.org/directory` |
| `LETS_ENCRYPT_EMAIL`           | E-mail address used to send expiration notifications                                                                                                                                                   |                                                  |
| `CLOUDFLARE_EMAIL`             | CloudFlare Account email                                                                                                                                                                               |                                                  |
| `CLOUDFLARE_DNS_API_TOKEN`     | API token with `DNS:Edit` permission                                                                                                                                                                   |                                                  |
| `CLOUDFLARE_ZONE_API_TOKEN`    | API token with `Zone:Read` permission                                                                                                                                                                  |                                                  |
| `SONARR_API_KEY`               | Sonarr API key to show information in the homepage                                                                                                                                                     |                                                  |
| `RADARR_API_KEY`               | Radarr API key to show information in the homepage                                                                                                                                                     |                                                  |
| `LIDARR_API_KEY`               | Lidarr API key to show information in the homepage                                                                                                                                                     |                                                  |
| `PROWLARR_API_KEY`             | Prowlarr API key to show information in the homepage                                                                                                                                                   |                                                  |
| `JELLYFIN_API_KEY`             | Jellyfin API key to show information in the homepage                                                                                                                                                   |                                                  |
| `JELLYSEERR_API_KEY`           | Jellyseer API key to show information in the homepage                                                                                                                                                  |                                                  |
| `HOMEPAGE_VAR_TITLE`           | Title of the homepage                                                                                                                                                                                  | `Docker-Compose NAS`                             |
| `HOMEPAGE_VAR_SEARCH_PROVIDER` | Homepage search provider, [see list here](https://gethomepage.dev/en/widgets/search/)                                                                                                                  | `google`                                         |
| `HOMEPAGE_VAR_HEADER_STYLE`    | Homepage header style, [see list here](https://gethomepage.dev/en/configs/settings/#header-style)                                                                                                      | `boxed`                                          |
| `HOMEPAGE_VAR_WEATHER_CITY`    | Homepage weather city name                                                                                                                                                                             |                                                  |
| `HOMEPAGE_VAR_WEATHER_LAT`     | Homepage weather city latitude                                                                                                                                                                         |                                                  |
| `HOMEPAGE_VAR_WEATHER_LONG`    | Homepage weather city longitude                                                                                                                                                                        |                                                  |
| `HOMEPAGE_VAR_WEATHER_UNIT`    | Homepage weather unit, either `metric` or `imperial`                                                                                                                                                   | `metric`                                         |

## PIA WireGuard VPN

I chose PIA since it supports WireGuard and [port forwarding](https://github.com/thrnz/docker-wireguard-pia/issues/26#issuecomment-868165281),
but you could use other providers:

- OpenVPN: [linuxserver/openvpn-as](https://hub.docker.com/r/linuxserver/openvpn-as)
- WireGuard: [linuxserver/wireguard](https://hub.docker.com/r/linuxserver/wireguard)
- NordVPN + OpenVPN: [bubuntux/nordvpn](https://hub.docker.com/r/bubuntux/nordvpn/dockerfile)
- NordVPN + WireGuard (NordLynx): [bubuntux/nordlynx](https://hub.docker.com/r/bubuntux/nordlynx)

For PIA + WireGuard, fill `.env` and fill it with your PIA credentials.

The location of the server it will connect to is set by `LOC=ca`, defaulting to Montreal - Canada.

You need to fill the credentials in the `PIA_*` environment variable, 
otherwise the VPN container will exit and qBittorrent will not start.

## Sonarr, Radarr & Lidarr

### File Structure

Sonarr, Radarr, and Lidarr must be configured to support hardlinks, to allow instant moves and prevent using twice the storage
(Bittorrent downloads and final file). The trick is to use a single volume shared by the Bittorrent client and the *arrs.
Subfolders are used to separate the TV shows from the movies.

The configuration is well explained by [this guide](https://trash-guides.info/Hardlinks/How-to-setup-for/Docker/).

In summary, the final structure of the shared volume will be as follows:

```
data
├── torrents = shared folder qBittorrent downloads
│  ├── movies = movies downloads tagged by Radarr
│  └── tv = movies downloads tagged by Sonarr
└── media = shared folder for Sonarr and Radarr files
   ├── movies = Radarr
   └── tv = Sonarr
   └── music = Lidarr
```

Go to Settings > Management.
In Sonarr, set the Root folder to `/data/media/tv`.
In Radarr, set the Root folder to `/data/media/movies`.
In Lidarr, set the Root folder to `/data/media/music`.

### Download Client

Then qBittorrent can be configured at Settings > Download Clients. Because all the networking for qBittorrent takes
place in the VPN container, the hostname for qBittorrent is the hostname of the VPN container, ie `vpn`, and the port is `8080`:

## Prowlarr

The indexers are configured through Prowlarr. They synchronize automatically to Radarr and Sonarr.

Radarr and Sonarr may then be added via Settings > Apps. The Prowlarr server is `http://prowlarr:9696/prowlarr`, the Radarr server
is `http://radarr:7878/radarr` Sonarr `http://sonarr:8989/sonarr`, and Lidarr `http://lidarr:8686/lidarr`:

Their API keys can be found in Settings > Security > API Key.

## qBittorrent

Set the default save path to `/data/torrents` in Settings, and restrict the network interface to WireGuard (`wg0`).

The web UI login page can be disabled on for the local network in Settings > Web UI > Bypass authentication for clients

```
192.168.0.0/16
127.0.0.0/8
172.17.0.0/16
```

## Plex

TODO: Update with information on gathering Plex API information

Generally, running Docker on Linux you will want to use VA-API, but the exact mount paths may differ depending on your
hardware.

## Homepage

The homepage comes with sensible defaults; some settings can ben controlled via environment variables in `.env`.

If you to customize further, you can modify the files in `/homepage/*.yaml` according to the [documentation](https://gethomepage.dev). 
Due to how the Docker socket is configured for the Docker integration, files must be edited as root.

The files in `/homepage/tpl/*.yaml` only serve as a base to set up the homepage configuration on first run.

## Overseerr

Overseerr gives you content recommendations, allows others to make requests to you, and allows logging in with Jellyfin credentials.

TODO: Update these instructions for Overseerr

To setup, go to https://hostname/jellyseerr/setup, and set the URLs as follows:
- Jellyfin: http://jellyfin:8096/jellyfin
- Radarr:
  - Hostname: radarr
  - Port: 7878
  - URL Base: /radarr
- Sonarr
  - Hostname: sonarr
  - Port: 8989
  - URL Base: /sonarr

## Traefik and SSL Certificates

While you can use the private IP to access your NAS, how cool would it be for it to be accessible through a subdomain
with a valid SSL certificate?

Traefik makes this trivial by using Let's Encrypt and one of its
[supported ACME challenge providers](https://doc.traefik.io/traefik/https/acme).

Let's assume we are using `nas.domain.com` as custom subdomain.

The idea is to create an A record pointing to the private IP of the NAS, `192.168.0.10` for example:
```
nas.domain.com.	1	IN	A	192.168.0.10
```

The record will be publicly exposed but not resolve given this is a private IP.

Given the NAS is not accessible from the internet, we need to do a dnsChallenge.
Here we will be using CloudFlare, but the mechanism will be the same for all DNS providers
baring environment variable changes, see the Traefik documentation above and [Lego's documentation](https://go-acme.github.io/lego/dns).

Then, fill the CloudFlare `.env` entries.

If you want to test your configuration first, use the Let's Encrypt staging server by updating `LETS_ENCRYPT_CA_SERVER`'s
value in `.env`:
```
LETS_ENCRYPT_CA_SERVER=https://acme-staging-v02.api.letsencrypt.org/directory
```

If it worked, you will see the staging certificate at https://nas.domain.com.
You may remove the `./letsencrypt/acme.json` file and restart the services to issue the real certificate.

You are free to use any DNS01 provider. Simply replace `DNS_CHALLENGE_PROVIDER` with your own provider, 
[see complete list here](https://doc.traefik.io/traefik/https/acme/#providers). 
You will also need to inject the environments variables specific to your provider.

Certificate generation can be disabled by setting `DNS_CHALLENGE` to `false`.

## Optional Services

As their name would suggest, optional services are not launched by default. They have their own `docker-compose.yml` file
in their subfolders. To enable a service, append it to the `COMPOSE_FILE` environment variable.

Say you want to enable FlareSolverr, you should have `COMPOSE_FILE=docker-compose.yml:flaresolverr/docker-compose.yml`

### FlareSolverr

In Prowlarr, add the FlareSolverr indexer with the URL http://flaresolverr:8191/

### SABnzbd

Enable SABnzbd by setting `COMPOSE_FILE=docker-compose.yml:sabnzbd/docker-compose.yml`. It will be accessible at `/sabnzbd`.

If that is not the case, the `url_base` parameter in `sabnzbd.ini` should be set to `/sabnzbd`.

## Customization

You can override the configuration of a service or add new services by creating a new `docker-compose.override.yml` file,
then appending it to the `COMPOSE_FILE` environment variable: `COMPOSE_FILE=docker-compose.yml:docker-compose.override.yml`

[See official documentation](https://docs.docker.com/compose/extends).

For example, use a [different VPN provider](https://github.com/bubuntux/nordvpn):

```yml
version: '3.9'

services:
  vpn:
    image: ghcr.io/bubuntux/nordvpn
    cap_add:
      - NET_ADMIN               # Required
      - NET_RAW                 # Required
    environment:                # Review https://github.com/bubuntux/nordvpn#environment-variables
      - USER=user@email.com     # Required
      - "PASS=pas$word"         # Required
      - CONNECT=United_States
      - TECHNOLOGY=NordLynx
      - NETWORK=192.168.1.0/24  # So it can be accessed within the local network
```

### Optional: Using the VPN for *arr apps

If you want to use the VPN for Prowlarr and other *arr applications, add the following block to all the desired containers:
```yml
    network_mode: "service:vpn"
    depends_on:
      vpn:
        condition: service_healthy
```

Change the healthcheck to mark the containers as unhealthy when internet connection is not working by appending a URL
to the healthcheck, eg: `test: [ "CMD", "curl", "--fail", "http://127.0.0.1:7878/radarr/ping", "https://google.com" ]`

Then in Prowlarr, use `localhost` rather than `vpn` as the hostname, since they are on the same network.

## Use Separate Paths for Torrents and Storage

If you want to use separate paths for torrents download and long term storage, to use different disks for example,
set your `docker-compose.override.yml` to:

```yml
version: "3.9"
services:
  sonarr:
    volumes:
      - ./sonarr:/config
      - ${DATA_ROOT}/media/tv:/data/media/tv
      - ${DOWNLOAD_ROOT}/tv:/data/torrents/tv
  radarr:
    volumes:
      - ./radarr:/config
      - ${DATA_ROOT}/media/movies:/data/media/movies
      - ${DOWNLOAD_ROOT}/movies:/data/torrents/movies
```

Note you will lose the hard link ability, ie your files will be duplicated.

In Sonarr and Radarr, go to `Settings` > `Importing` > Untick `Use Hardlinks instead of Copy`
