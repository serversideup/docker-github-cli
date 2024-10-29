<p align="center">
		<img src="https://raw.githubusercontent.com/serversideup/docker-github-cli/main/.github/img/header.png" width="1280" alt="Docker Images Logo"></a>
</p>
<p align="center">
	<a href="https://github.com/serversideup/docker-github-cli/actions/workflows/action_publish-images-production.yml"><img alt="Build Status" src="https://img.shields.io/github/actions/workflow/status/serversideup/docker-github-cli/.github%2Fworkflows%2Faction_publish-images-production.yml"></a>
	<a href="https://github.com/serversideup/docker-github-cli/blob/main/LICENSE" target="_blank"><img src="https://badgen.net/github/license/serversideup/docker-github-cli" alt="License"></a>
	<a href="https://github.com/sponsors/serversideup"><img src="https://badgen.net/badge/icon/Support%20Us?label=GitHub%20Sponsors&color=orange" alt="Support us"></a>
  <br />
  <a href="https://hub.docker.com/r/serversideup/github-cli/"><img alt="serversideup/github-cli pulls" src="https://img.shields.io/docker/pulls/serversideup/github-cli?label=serversideup%2Fgithub-cli%20pulls"></a>
  <a href="https://serversideup.net/discord"><img alt="Discord" src="https://img.shields.io/discord/910287105714954251?color=blueviolet"></a>
</p>

## Introduction
`serversideup/github-cli` is a lightweight solution for running GitHub CLI via Docker.

## Features
- 🐧 **Debian and Alpine** - Choose your OS
- 🔒 **Unprivileged user** - Choose to run as root or an unprivileged user
- 📌 **Pinned GitHub CLI Version** - Set your GitHub CLI version down to the patch version
- 🔧 **Customize your "run as" user** - Customize the username to run as
- 🔑 **Set your own PUID and PGID** - Have the PUID and PGID match your host user
- 📦 **DockerHub and GitHub Container Registry** - Choose where you'd like to pull your image from
- 🤖 **Multi-architecture** - Every image ships with x86_64 and arm64 architectures

## Usage
Getting started is easy. Here's a few tips on how to use this image.

### Choose your registry

Our images are available on Docker Hub and GitHub Container Registry 🥳

**DockerHub:**
- [serversideup/github-cli](https://hub.docker.com/r/serversideup/github-cli)

**GitHub Container Registry:**
- [ghcr.io/serversideup/github-cli](https://github.com/serversideup/docker-github-cli/pkgs/container/github-cli)

| Variation | Image Size |
| --------- | -------------------- |
| `serversideup/github-cli` | [![DockerHub serversideup/github-cli:alpine](https://img.shields.io/docker/image-size/serversideup/github-cli/alpine?label=alpine)](https://hub.docker.com/r/serversideup/github-cli/tags?name=alpine)<br>[![DockerHub serversideup/github-cli](https://img.shields.io/docker/image-size/serversideup/github-cli/latest?label=debian)](https://hub.docker.com/r/serversideup/github-cli)

### Mounting directories
There are a few directories that we need to mount to ensure everything works correctly.

#### Mounting the `/config` directory
In order to authenticate with GitHub, we need to mount your `~/.config` directory from your host into the container. We have a special `/config` directory that is symbolically linked from the unprivileged user's home directory. We use this root directory so you can change the `RUN_AS_USER` environment variable and not have to worry about having to predict what the absolute path to the configuration directory is. 

For example, if you rename the `RUN_AS_USER` to something like `bob`, `/home/bob/.config` will be symbolically linked to `/config`, giving you a predictable path to mount your configuration files.

#### Working with SSH
SSH also behaves in a similar way. Keep reading below for more details.

#### Mounting the `/workdir` directory
We also have a `/workdir` directory that is intended for you your workspace.

### Change the "run as" user, PUID and PGID

```bash
docker run --rm -it \
  -v "$HOME/.ssh:/ssh:ro" \
  -v "$HOME/.ssh/known_hosts:/ssh/known_hosts:rw" \
  -v "$(pwd):/workdir" \
  -v "$HOME/.config/gh:/config/gh:rw" \
  -e "PUID=9999" \
  -e "PGID=9999" \
  -e "RUN_AS_USER=bob" \
  serversideup/github-cli:latest gh --version
```

### Run a shell
```bash
docker run --rm -it \
  -v "$HOME/.ssh:/ssh" \
  -v "$(pwd):/workdir" \
  serversideup/github-cli:latest /bin/sh
```

### Working with SSH
> [!NOTE]  
> Working with SSH keys can be tricky, especially if we're setting a `RUN_AS_USER` dynamically. We created a few things to help reduce the headache of getting this configured.

#### The `/ssh` directory
By default, we have a `/ssh` directory that is symbolically linked from `~/.ssh`. The `/ssh` directory is used as our single source of truth for SSH keys and configurations.

If you set `RUN_AS_USER`, the entrypoint will create a home directory at `/home/${RUN_AS_USER}`, then create a symbolic link from `/home/${RUN_AS_USER}/.ssh` to `/ssh`. This gives you the power to set your `RUN_AS_USER` to anything you want without us needing to predict what user you want to run as.

#### Mounting the SSH auth socket
The SSH auth socket is a Unix socket used by the SSH agent to communicate with other processes, allowing for secure key management. To use it with Docker, you can mount it as follows:

**macOS:**
```bash
docker run --rm -it \
  -v "$HOME/.ssh:/ssh:ro" \
  -v "$HOME/.ssh/known_hosts:/ssh/known_hosts:rw" \
  -v "$(pwd):/workdir" \
  -v "/run/host-services/ssh-auth.sock:/run/host-services/ssh-auth.sock" \
  -e SSH_AUTH_SOCK="/run/host-services/ssh-auth.sock" \
  serversideup/github-cli:latest gh --version
```

Notice how we're matching the `SSH_AUTH_SOCK` to the host's socket. This is necessary for the SSH agent to communicate with the container.

**Linux:**
```bash
docker run --rm -it \
  -v "$HOME/.ssh:/ssh:ro" \
 -v "$HOME/.ssh/known_hosts:/ssh/known_hosts:rw" \
  -v "$(pwd):/workdir" \
  -v "$SSH_AUTH_SOCK:$SSH_AUTH_SOCK" \
  -e SSH_AUTH_SOCK=$SSH_AUTH_SOCK" \
  serversideup/github-cli:latest gh --version
```

### Environment Variables
You can customize the image easily with the following environment variables:

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `PUID` | `1000` | Set the User ID that you'd like to run GitHub CLI as |
| `PGID` | `1000` | Set the Group ID that you'd like to run GitHub CLI as |
| `RUN_AS_USER` | `github` | The username you'd like to run GitHub CLI as (this will be created for you and will default to an **unprivileged** user) |
| `DEBUG` | `false` | Enable debug output of container startup |


## Resources
- **[DockerHub](https://hub.docker.com/r/serversideup/github-cli)** to browse the images.
- **[Discord](https://serversideup.net/discord)** for friendly support from the community and the team.
- **[GitHub](https://github.com/serversideup/docker-github-cli)** for source code, bug reports, and project management.
- **[Get Professional Help](https://serversideup.net/professional-support)** - Get video + screen-sharing help directly from the core contributors.

## Contributing
As an open-source project, we strive for transparency and collaboration in our development process. We greatly appreciate any contributions members of our community can provide. Whether you're fixing bugs, proposing features, improving documentation, or spreading awareness - your involvement strengthens the project. Please review our [code of conduct](./.github/code_of_conduct.md) to understand how we work together respectfully.

- **Bug Report**: If you're experiencing an issue while using these images, please [create an issue](https://github.com/serversideup/docker-github-cli/issues/new/choose).
- **Feature Request**: Make this project better by [submitting a feature request](https://github.com/serversideup/docker-github-cli/discussions/).
- **Documentation**: Improve our documentation by [submitting a documentation change](./README.md).
- **Community Support**: Help others on [GitHub Discussions](https://github.com/serversideup/docker-github-cli/discussions) or [Discord](https://serversideup.net/discord).
- **Security Report**: Report critical security issues via [our responsible disclosure policy](https://www.notion.so/Responsible-Disclosure-Policy-421a6a3be1714d388ebbadba7eebbdc8).

Need help getting started? Join our Discord community and we'll help you out!

<a href="https://serversideup.net/discord"><img src="https://serversideup.net/wp-content/themes/serversideup/images/open-source/join-discord.svg" title="Join Discord"></a>

## Our Sponsors
All of our software is free an open to the world. None of this can be brought to you without the financial backing of our sponsors.

<p align="center"><a href="https://github.com/sponsors/serversideup"><img src="https://521public.s3.amazonaws.com/serversideup/sponsors/sponsor-box.png" alt="Sponsors"></a></p>

### Black Level Sponsors
<a href="https://sevalla.com"><img src="https://serversideup.net/wp-content/uploads/2024/10/sponsor-image.png" alt="Sevalla" width="546px"></a>

#### Bronze Sponsors
<!-- bronze -->No bronze sponsors yet. <a href="https://github.com/sponsors/serversideup">Become a sponsor →</a><!-- bronze -->

#### Individual Supporters
<!-- supporters --><a href="https://github.com/GeekDougle"><img src="https://github.com/GeekDougle.png" width="40px" alt="GeekDougle" /></a>&nbsp;&nbsp;<a href="https://github.com/JQuilty"><img src="https://github.com/JQuilty.png" width="40px" alt="JQuilty" /></a>&nbsp;&nbsp;<a href="https://github.com/MaltMethodDev"><img src="https://github.com/MaltMethodDev.png" width="40px" alt="MaltMethodDev" /></a>&nbsp;&nbsp;<!-- supporters -->

## About Us
We're [Dan](https://twitter.com/danpastori) and [Jay](https://twitter.com/jaydrogers) - a two person team with a passion for open source products. We created [Server Side Up](https://serversideup.net) to help share what we learn.

<div align="center">

| <div align="center">Dan Pastori</div>                  | <div align="center">Jay Rogers</div>                                 |
| ----------------------------- | ------------------------------------------ |
| <div align="center"><a href="https://twitter.com/danpastori"><img src="https://serversideup.net/wp-content/uploads/2023/08/dan.jpg" title="Dan Pastori" width="150px"></a><br /><a href="https://twitter.com/danpastori"><img src="https://serversideup.net/wp-content/themes/serversideup/images/open-source/twitter.svg" title="Twitter" width="24px"></a><a href="https://github.com/danpastori"><img src="https://serversideup.net/wp-content/themes/serversideup/images/open-source/github.svg" title="GitHub" width="24px"></a></div>                        | <div align="center"><a href="https://twitter.com/jaydrogers"><img src="https://serversideup.net/wp-content/uploads/2023/08/jay.jpg" title="Jay Rogers" width="150px"></a><br /><a href="https://twitter.com/jaydrogers"><img src="https://serversideup.net/wp-content/themes/serversideup/images/open-source/twitter.svg" title="Twitter" width="24px"></a><a href="https://github.com/jaydrogers"><img src="https://serversideup.net/wp-content/themes/serversideup/images/open-source/github.svg" title="GitHub" width="24px"></a></div>                                       |

</div>

### Find us at:

* **📖 [Blog](https://serversideup.net)** - Get the latest guides and free courses on all things web/mobile development.
* **🙋 [Community](https://community.serversideup.net)** - Get friendly help from our community members.
* **🤵‍♂️ [Get Professional Help](https://serversideup.net/professional-support)** - Get video + screen-sharing support from the core contributors.
* **💻 [GitHub](https://github.com/serversideup)** - Check out our other open source projects.
* **📫 [Newsletter](https://serversideup.net/subscribe)** - Skip the algorithms and get quality content right to your inbox.
* **🐥 [Twitter](https://twitter.com/serversideup)** - You can also follow [Dan](https://twitter.com/danpastori) and [Jay](https://twitter.com/jaydrogers).
* **❤️ [Sponsor Us](https://github.com/sponsors/serversideup)** - Please consider sponsoring us so we can create more helpful resources.

## Our products
If you appreciate this project, be sure to check out our other projects.

### 📚 Books
- **[The Ultimate Guide to Building APIs & SPAs](https://serversideup.net/ultimate-guide-to-building-apis-and-spas-with-laravel-and-nuxt3/)**: Build web & mobile apps from the same codebase.
- **[Building Multi-Platform Browser Extensions](https://serversideup.net/building-multi-platform-browser-extensions/)**: Ship extensions to all browsers from the same codebase.

### 🛠️ Software-as-a-Service
- **[Bugflow](https://bugflow.io/)**: Get visual bug reports directly in GitHub, GitLab, and more.
- **[SelfHost Pro](https://selfhostpro.com/)**: Connect Stripe or Lemonsqueezy to a private docker registry for self-hosted apps.

### 🌍 Open Source
- **[AmplitudeJS](https://521dimensions.com/open-source/amplitudejs)**: Open-source HTML5 & JavaScript Web Audio Library.
- **[Spin](https://serversideup.net/open-source/spin/)**: Laravel Sail alternative for running Docker from development → production.
- **[Financial Freedom](https://github.com/serversideup/financial-freedom)**: Open source alternative to Mint, YNAB, & Monarch Money.