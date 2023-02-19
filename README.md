# [bradpeczka/docker-nzbget-ng](https://github.com/bradpeczka/docker-nzbget-ng/)

This is a fork of the original [LSIO docker-nzbget repository](https://github.com/linuxserver/docker-nzbget), that has been updated to use the [nzbget-ng](https://github.com/nzbget-ng/nzbget) along with a variety of improvements and fixes.

[nzbget-ng](https://github.com/nzbget-ng/nzbget) is a usenet downloader, written in C++ and designed with performance in mind to achieve maximum download speed by using very little system resources.

[![nzbget](https://raw.githubusercontent.com/linuxserver/docker-templates/master/linuxserver.io/img/nzbget-banner.png)](http://nzbget.net/)

## Application Setup

Webui can be found at  `<your-ip>:6789` and the default login details (change ASAP) are

`login:nzbget, password:tegbzn6789`

To allow scheduling, from the webui set the time correction value in settings/logging.

To change umask settings.

![](http://i.imgur.com/A4VMbwE.png)

scroll to bottom, set umask like this (example shown for unraid)

![](http://i.imgur.com/mIqDEJJ.png)

You can add an additional mount point for intermediate unpacking folder with:-

`-v </path/to/intermedia_unpacking_folder>:/intermediate`

for example, and changing the setting for InterDir in the PATHS tab of settings to `/intermediate`

### Media folders

We have set `/downloads` as a ***optional path***, this is because it is the easiest way to get started. While easy to use, it has some drawbacks. Mainly losing the ability to atomic move (TL;DR instant file moves, rather than copy+delete) files while processing content.

Use the optional paths if you dont understand, or dont want hardlinks/atomic moves.

The folks over at servarr.com wrote a good [write-up](https://wiki.servarr.com/docker-guide#consistent-and-well-planned-paths) on how to get started with this.

## Usage

Here are some example snippets to help you get started creating a container.

### docker-compose (recommended, [click here for more info](https://docs.linuxserver.io/general/docker-compose))

```yaml
---
version: "2.1"
services:
  nzbget:
    image: ghcr.io/bradpeczka/docker-nzbget-ng:latest
    container_name: nzbget
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - NZBGET_USER=nzbget #optional
      - NZBGET_PASS=tegbzn6789 #optional
    volumes:
      - /path/to/data:/config
      - /path/to/downloads:/downloads #optional
    ports:
      - 6789:6789
    restart: unless-stopped
```

### docker cli ([click here for more info](https://docs.docker.com/engine/reference/commandline/cli/))

```bash
docker run -d \
  --name=nzbget \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e NZBGET_USER=nzbget `#optional` \
  -e NZBGET_PASS=tegbzn6789 `#optional` \
  -p 6789:6789 \
  -v /path/to/data:/config \
  -v /path/to/downloads:/downloads `#optional` \
  --restart unless-stopped \
  ghcr.io/bradpeczka/docker-nzbget-ng:latest
```

## Parameters

Container images are configured using parameters passed at runtime (such as those above). These parameters are separated by a colon and indicate `<external>:<internal>` respectively. For example, `-p 8080:80` would expose port `80` from inside the container to be accessible from the host's IP on port `8080` outside the container.

| Parameter | Function |
| :----: | --- |
| `-p 6789` | WebUI |
| `-e PUID=1000` | for UserID - see below for explanation |
| `-e PGID=1000` | for GroupID - see below for explanation |
| `-e TZ=Europe/London` | Specify a timezone to use EG Europe/London. |
| `-e NZBGET_USER=nzbget` | Specify the user for web authentication. |
| `-e NZBGET_PASS=tegbzn6789` | Specify the password for web authentication. |
| `-v /config` | NZBGet App data. |
| `-v /downloads` | Location of downloads on disk. |

## Environment variables from files (Docker secrets)

You can set any environment variable from a file by using a special prepend `FILE__`.

As an example:

```bash
-e FILE__PASSWORD=/run/secrets/mysecretpassword
```

Will set the environment variable `PASSWORD` based on the contents of the `/run/secrets/mysecretpassword` file.

## Umask for running applications

For all of our images we provide the ability to override the default umask settings for services started within the containers using the optional `-e UMASK=022` setting.
Keep in mind umask is not chmod it subtracts from permissions based on it's value it does not add. Please read up [here](https://en.wikipedia.org/wiki/Umask) before asking for support.

## User / Group Identifiers

When using volumes (`-v` flags) permissions issues can arise between the host OS and the container, we avoid this issue by allowing you to specify the user `PUID` and group `PGID`.

Ensure any volume directories on the host are owned by the same user you specify and any permissions issues will vanish like magic.

In this instance `PUID=1000` and `PGID=1000`, to find yours use `id user` as below:

```bash
  $ id username
    uid=1000(dockeruser) gid=1000(dockergroup) groups=1000(dockergroup)
```

## Support Info

* Shell access whilst the container is running: `docker exec -it nzbget /bin/bash`
* To monitor the logs of the container in realtime: `docker logs -f nzbget`
* container version number
  * `docker inspect -f '{{ index .Config.Labels "build_version" }}' nzbget`
* image version number
  * `docker inspect -f '{{ index .Config.Labels "build_version" }}' lscr.io/linuxserver/nzbget:latest`

## Updating Info

Most of our images are static, versioned, and require an image update and container recreation to update the app inside. With some exceptions (ie. nextcloud, plex), we do not recommend or support updating apps inside the container. Please consult the [Application Setup](#application-setup) section above to see if it is recommended for the image.

Below are the instructions for updating containers:

### Via Docker Compose

* Update all images: `docker-compose pull`
  * or update a single image: `docker-compose pull nzbget`
* Let compose update all containers as necessary: `docker-compose up -d`
  * or update a single container: `docker-compose up -d nzbget`
* You can also remove the old dangling images: `docker image prune`

### Via Docker Run

* Update the image: `docker pull lscr.io/linuxserver/nzbget:latest`
* Stop the running container: `docker stop nzbget`
* Delete the container: `docker rm nzbget`
* Recreate a new container with the same docker run parameters as instructed above (if mapped correctly to a host folder, your `/config` folder and settings will be preserved)
* You can also remove the old dangling images: `docker image prune`

### Via Watchtower auto-updater (only use if you don't remember the original parameters)

* Pull the latest image at its tag and replace it with the same env variables in one run:

  ```bash
  docker run --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  containrrr/watchtower \
  --run-once nzbget
  ```

* You can also remove the old dangling images: `docker image prune`

**Note:** We do not endorse the use of Watchtower as a solution to automated updates of existing Docker containers. In fact we generally discourage automated updates. However, this is a useful tool for one-time manual updates of containers where you have forgotten the original parameters. In the long term, we highly recommend using [Docker Compose](https://docs.linuxserver.io/general/docker-compose).

### Image Update Notifications - Diun (Docker Image Update Notifier)

* We recommend [Diun](https://crazymax.dev/diun/) for update notifications. Other tools that automatically update containers unattended are not recommended or supported.

## Building locally

If you want to make local modifications to these images for development purposes or just to customize the logic:

```bash
git clone https://github.com/linuxserver/docker-nzbget.git
cd docker-nzbget
docker build \
  --no-cache \
  --pull \
  -t lscr.io/linuxserver/nzbget:latest .
```

The ARM variants can be built on x86_64 hardware using `multiarch/qemu-user-static`

```bash
docker run --rm --privileged multiarch/qemu-user-static:register --reset
```

Once registered you can define the dockerfile to use with `-f Dockerfile.aarch64`.

## Versions

* **19.02.23:** - Forked. Dockerfiles updated to use nzbget-ng. Rebase master to 3.17.
* **31.12.22:** - Deprecate image.  Please consider switching to SABnzbd https://github.com/linuxserver/docker-sabnzbd
* **27.11.22:** - Advanced notice: This image will be deprecated on 2022-12-31. Please consider switching to SABnzbd https://github.com/linuxserver/docker-sabnzbd
* **13.11.22:** - Rebase master to 3.16, migrate to s6v3.
* **12.08.22:** - Bump unrar to 6.1.7.
* **22.02.22:** - Rebase to alpine 3.15, add six and python 7zip tools, allow env variables for credentials.
* **04.07.21:** - Rebase to alpine 3.14.
* **28.05.21:** - Add linuxserver wheel index.
* **23.01.21:** - Rebasing to alpine 3.13.
* **26.10.20:** - Fix python dependencies.
* **24.08.20:** - Fix ignored umask environment variable.
* **08.06.20:** - Symlink python3 bin to python.
* **01.06.20:** - Rebasing to alpine 3.12. Removing python2.
* **13.05.20:** - Add rarfile python package (for DeepUnrar).
* **01.01.20:** - Add python3 alongside python2 during transition.
* **19.12.19:** - Rebasing to alpine 3.11.
* **28.06.19:** - Rebasing to alpine 3.10.
* **13.06.19:** - Add apprise, chardet & pynzbget packages.
* **23.03.19:** - Switching to new Base images, shift to arm32v7 tag.
* **25.02.19:** - Rebasing to alpine 3.9.
* **20.01.19:** - Add pipeline logic and multi arch, build from source.
* **21.08.18:** - Rebase to alpine 3.8.
* **20.02.18:** - Add note about supplemental mount point for intermediate unpacking.
* **13.12.17:** - Rebase to alpine 3.7.
* **02.09.17:** - Place app in subfolder rather than /app.
* **12.07.17:** - Add inspect commands to README, move to jenkins build and push.
* **28.05.17:** - Rebase to alpine 3.6.
* **20.04.17:** - Add testing branch.
* **06.02.17:** - Rebase to alpine 3.5.
* **30.09.16:** - Fix umask.
* **09.09.16:** - Add layer badges to README.
* **27.08.16:** - Add badges to README, perms fix on /app to allow updates.
* **19.08.16:** - Rebase to alpine linux.
* **18.08.15:** - Now useing latest version of unrar beta and implements the universal installer method.
