+++
title = 'Hosting My Own Personal Site - Hugo Static Website Generator'
date = 2023-09-25T19:00:00Z
draft = true
+++

Having a personal website with a custom domain to write my thoughts, document my learnings, and put up my projects have always been a long-time goal of mine. I recently decided to follow up through that goal and bought a domain name along with a VPS (Virtual Private Server) subcription to host my website. Why VPS when another cheaper and easier solutions exist? Well because I want to learn on how to host my own website from scratch.

To create this website I researched a lot of options I could use, from the cloud provider to the tech stacks to use. For cloud provider, I wanted to use digitalocean at first due to their "free trial" with limited credits, but unfortunately they suspended my account right after I put my credit card information. From all the infos I gathered in the internet, they have a tendency to block accounts that use new credit card as a fraud prevention. Decided that contacting them would require too much efforts on my part, I chose a local cloud provider in my country, idcloudhost. For the website tech stacks, I got a few options from the recommendation of others on the internet and decided to use Hugo as I only need a static website for this project of mine.

In my first post, I want to document the steps I did to host my website. These are the tools I'm using below:
1. Linux based OS for the server
2. Hugo as a site generator
3. PaperMod - Hugo Theme
4. Apache 2 as web server
5. Git

## A little about Hugo
Hugo is a static-website generator. It's fast, simple, free, open source, and support multi-language website. Hugo also have pre-built theme that will help me to create my website faster without the need to delve much into the technical side of web development. But the good thing is, if I want more costumization, I can also create my own template in the future. Overall, it is not too hard to setup.

## Installing Hugo
I installed Hugo on my Linux VPS (as a production) and on my local PC (as development). On the VPS, I installed it using the prebuilt binaries while I used docker on my local PC for development. Before installing anything, I created a folder to store the downloaded file in my home folder.

```bash
# Command (Run in Terminal):
mkdir ~/installer
```

### Installing Prerequisites
While these softwares are not required for all case. I decided to install all of them for future usage:
1. Git
2. Go
3. Dart Sass

#### Installing Git
For git, I just install it using apt install. Then validate the installation by running `git version` command.

```bash
# Install Git
sudo apt install git-all

# Validate Installation by Checking Git Version
git version
# Output
git version 2.25.1
```

#### Installing Go
I went to the official Go website https://go.dev/doc/install to get the installer download link.

```bash
# Command to download with wget (run in Terminal):
wget <download_link> -P ~/installer/

# Example:
wget https://go.dev/dl/go1.21.1.linux-amd64.tar.gz -P ~/installer/
```

rm -rf /usr/local/go && tar -C /usr/local -xzf go1.21.4.linux-amd64.tar.gz

Add this line to `$HOME/.profile` or `/etc/profile` (for a system-wide installation). I used vi as my text editor. Here how it's look like:

```bash
# /etc/profile: system-wide .profile file for the Bourne shell (sh(1))
# and Bourne compatible shells (bash(1), ksh(1), ash(1), ...).

.
.
.

# Applications Path    # <-- Add it here at the bottom
GO="/usr/local/go/bin" # <-- Add it here at the bottom
PATH="$PATH:$GO"       # <-- Add it here at the bottom
```

Verify the installation by running this command:
```bash
# Validate Installation by Checking Go Version
go version
# Output
go version go1.21.1 linux/amd64
```

#### Installing Dart SASS
Go to the Dart SASS release page and choose the version (I used version 1.66.1).

```bash
# Command to download with wget (run in Terminal):
wget <download_link> -P ~/installer/

# Example:
wget https://github.com/sass/dart-sass/releases/download/1.66.1/dart-sass-1.66.1-linux-x64.tar.gz -P ~/installer/
```

Then extract the archive file to `~/installer/` folder.

```bash
# Command (Run in Terminal):
tar -xvf <archive_file_name> -C ~/installer/

# Example:
tar -xvf ~/installer/dart-sass-1.66.1-linux-x64.tar.gz -C ~/installer/
```

Move the extracted "dart-sass" folder to `/user/local/` folder.

```bash
# Example:
mv ~/installer/dart-sass /user/local/
```

Add this line to `$HOME/.profile` or `/etc/profile` (for a system-wide installation). You can use any text editor. Here how it's look like:

```bash
# /etc/profile: system-wide .profile file for the Bourne shell (sh(1))
# and Bourne compatible shells (bash(1), ksh(1), ash(1), ...).

...
...
...

# Applications Path
DART_SASS="/usr/local/dart-sass" # <-- Add it here
GO="/usr/local/go/bin"
PATH="$PATH:$DART_SASS:$GO"      # <--Edit this line
```

Verify the installation by running this command:
```bash
# Validate Installation by Checking SASS Version
sass --version
# Output
1.66.1
```

### Installing Hugo - Prebuilt Binaries Installation in Linux
1. Go to this website: https://github.com/gohugoio/hugo/tags
2. Download the version that is going to be used. For my website I use the version 0.119.0.

    ```bash
    # Command (Run in Terminal):
    wget <download_link> -P ~/installer/
    
    # Example:
    wget https://github.com/gohugoio/hugo/releases/download/v0.119.0/hugo_extended_0.119.0_Linux-64bit.tar.gz -P ~/installer/
    ```

3. Extract the archive file

    ```bash
    # Command (Run in Terminal):
    tar -xvf <archive_file_name> -C ~/installer/

    # Example:
    tar -xvf ~/installer/hugo_extended_0.119.0_Linux-64bit.tar.gz -C ~/installer/
    ```

4. Move the "hugo" executable file to `/usr/local/bin/`. This directory already exists in Linux PATH environment variable, so there is no need to add it again later.

    ```bash
    # Command (Run in Terminal):
    mv hugo /user/local/bin/
    ```

5. Run this command to confirm that hugo already installed correctly.

    ```bash
    # Command (Run in Terminal):
    hugo env
    # Output:
    hugo v0.119.0-b84644c008e0dc2c4b67bd69cccf87a41a03937e+extended linux/amd64 BuildDate=2023-09-24T15:20:17Z VendorInfo=gohugoio
    GOOS="linux"
    GOARCH="amd64"
    GOVERSION="go1.21.1"
    github.com/sass/libsass="3.6.5"
    github.com/webmproject/libwebp="v1.2.4"
    github.com/sass/dart-sass/protocol="2.1.0"
    github.com/sass/dart-sass/compiler="1.66.1"
    github.com/sass/dart-sass/implementation="1.66.1"
    ```

### Installing Hugo - Docker Container Configuration in Windows
For development I use a docker container in my windows machine. First install docker desktop in Windows, I followed [Docker Official Documentation](https://docs.docker.com/desktop/install/windows-install/) for this. Then I created a custom docker image using this dockerfile:

#### Dockerfile

```dockerfile
# Use a base image
FROM ubuntu:20.04 AS base

WORKDIR /tmp

# Update and install required packages
RUN apt-get update && apt-get install -y git wget

# Install Go
RUN wget https://go.dev/dl/go1.21.1.linux-amd64.tar.gz -P /tmp/ && \
    tar -C /usr/local -xzf /tmp/go1.21.1.linux-amd64.tar.gz && \
    rm /tmp/go1.21.1.linux-amd64.tar.gz

# Install Hugo
RUN wget https://github.com/gohugoio/hugo/releases/download/v0.118.2/hugo_extended_0.118.2_linux-amd64.tar.gz -P /tmp/ && \
    tar -C /usr/local/bin -xzf /tmp/hugo_extended_0.118.2_linux-amd64.tar.gz && \
    rm /tmp/hugo_extended_0.118.2_linux-amd64.tar.gz

# Install Hugo
RUN wget https://github.com/sass/dart-sass/releases/download/1.66.1/dart-sass-1.66.1-linux-x64.tar.gz -P /tmp/ && \
    tar -C /usr/local -xzf /tmp/dart-sass-1.66.1-linux-x64.tar.gz && \
    rm /tmp/dart-sass-1.66.1-linux-x64.tar.gz

# Set the PATH for Go & Dart SASS
ENV PATH="$PATH:/usr/local/go/bin:/usr/local/dart-sass"

# Expose the port for Hugo server
EXPOSE 1313
```

The dockerfile above will build docker image with this specification:
- Ubuntu 20.04 as operating system
- go 1.21.1
- hugo extended 0.118.2
- dart-sass 1.66.1

The version of all the listed softwares can be changed based on needs by modifying the software download link in the dockerfile.

<!-- I also made an ARG site_option for my docker image. The 0 value mean that I want to use an existing hugo project inside src folder on my host, while 1 value mean I want to make a new hugo project. This ARG site_option can be used in docker compose file later. -->

To create the container from the image, I run this command:
```bash
docker run --rm --name de-hugo-dev -p 1313:1313 -v "./srv/http/src:/srv/http/src" --entrypoint "hugo" de-hugo-dev new site . --force
```

The command above will create a new hugo project (if I need it) and put it inside `./srv/http/src/` on my host. The container will immediately stopped and deleted. Then I can use a docker compose run my existing project in `./srv/http/src`.

#### Docker Compose File
Then I use this docker compose file to run the development website.

```yaml
services:
  hugo-server:
    image: "de-hugo-dev"
    ports:
      - "1313:1313"
    container_name: "de-hugo-dev"
    entrypoint: hugo
    command: "server -D --bind 0.0.0.0 --port 1313 --poll 700ms"
    working_dir: /srv/http/src
    volumes:
      - "./srv/http/src:/srv/http/src"
```