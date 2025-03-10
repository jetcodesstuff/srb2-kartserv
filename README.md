# Sonic Robo Blast 2 Kart Server

[![Docker Image Version](https://img.shields.io/docker/v/ellite/srb2kart-server)](https://hub.docker.com/r/ellite/srb2kart-server)
[![Docker Image Size](https://img.shields.io/docker/image-size/ellite/srb2kart-server)](https://hub.docker.com/r/ellite/srb2kart-server)

> Containerized version of SRB2Kart. This is a fork of [rwanyoikes Dockerfile](https://github.com/rwanyoike/srb2kart-server-docker)

<p align="center">
  <img src="https://cdn.discordapp.com/attachments/298839130144505858/512450353124343808/unknown.png" width="100%" alt="SRB2Kart">
</p>

Containerized version of [SRB2Kart](https://mb.srb2.org/showthread.php?t=43708), a kart racing mod based on the 3D Sonic the Hedgehog fangame [Sonic Robo Blast 2](https://srb2.org/), based on a modified version of [Doom Legacy](http://doomlegacy.sourceforge.net/). You can use SRB2Kart to run a SRB2Kart dedicated netgame server given the proper config.

## Usage

This will pull an image with SRB2Kart and start a dedicated netgame server on port `5029/udp`:

```bash
docker run -it --name srb2kart -p 5029:5029/udp jetcodesstuff/srb2-kartserv:latest
```

### Data Volume

The `~/.srb2kart` directory is symlinked to `/kart` in the container. You can bind-mount a SRB2Kart directory (with configuration files, mods, etc.) on the host machine to the `/kart` directory inside the container. For example:

#### Addons
  In order to load addons, bind the `/addons` volume to a host directory and copy them there.
  ```bash
  docker run -it --name srb2kart -p 5029:5029/udp -v /path/on/host/addons:/addons ellite/srb2kart-server:latest
  ```

```bash
$ tree srb2kart-myserver/
srb2kart-myserver
├── mods
│   ├── kl_xxx.pk3
│   ├── kl_xxx.wad
│   └── kr_xxx.pk3
└── kartserv.cfg

1 directory, 4 files
```

> This directory must be accessible to the user account that is used to run SRB2Kart inside the container. If your host machine is run under *nix OS, SRB2Kart uses the non-root account `1000:1000` (`group:id`, respectively).

```bash
docker run --rm -it --name srb2kart \
    -v <path to data directory>:/kart \
    -p <port on host>:5029/udp \
    jetcodesstuff/srb2-kartserv:<version> -dedicated -file \
    mods/kl_xxx.pk3 \
    mods/kl_xxx.wad \
    mods/kr_xxx.pk3
```

### systemd

Here's an example of how to run the container as a service on Linux with the help of `systemd`.

1. Create a systemd service descriptor file:

  ```ini
  # /etc/systemd/system/docker.srb2kart.service

  [Unit]
  Description=SRB2Kart Server
  Requires=docker.service
  After=docker.service
  # Ref: https://www.freedesktop.org/software/systemd/man/systemd.unit.html#StartLimitIntervalSec=interval
  StartLimitIntervalSec=60s
  StartLimitBurst=2

  [Service]
  TimeoutStartSec=0
  Restart=on-failure
  RestartSec=5s
  ExecStartPre=/usr/bin/docker stop %n
  ExecStartPre=/usr/bin/docker rm %n
  ExecStartPre=/usr/bin/docker pull jetcodesstuff/srb2-kartserv:<version>
  ExecStart=/usr/bin/docker run --rm --name %n \
      -v <path to data directory>:/kart \
      -p <port on host>:5029/udp \
      jetcodesstuff/srb2-kartserv:<version>

  [Install]
  WantedBy=multi-user.target
  ```

2. Enable starting the service on system boot:

  ```bash
  systemctl enable docker.srb2kart
  ```

## Manual Build

```bash
git clone https://github.com/ellite/srb2kart-server-docker
cd srb2kart-server-docker/
# Ref for version numbers: https://github.com/STJr/Kart-Public/releases
docker build --build-arg "SRB2KART_VERSION=<version>" \
    -t srb2kart-server:<version> .
```

The build will download the Source Code and build the SRB2Kart executable, as well as download the data files (`/usr/share/games/SRB2Kart`) for SRB2Kart.

## License

This project is licensed under the [GPLv2 License](./LICENSE).
