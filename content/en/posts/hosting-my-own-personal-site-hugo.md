---
title: 'Hosting My Own Personal Site - Hugo Static Website Generator'
date: '2024-04-09T16:28:00+07:00'
categories: ['Web Hosting']
tags: ['Hugo', 'Web Hosting', 'Docker']
series: ['Personal Site with Hugo']
draft: false
---

Having a personal website with a custom domain to write my thoughts, document my learnings, and showcase my projects has always been a long-time goal of mine. I recently decided to follow through on that goal and bought a domain name, along with a VPS (Virtual Private Server) subscription, to host my website. Why VPS when there are cheaper and easier solutions? Well, because I want to learn how to host my own website from scratch.

To create this website, I researched a lot of options I could use, from the cloud provider to the tech stacks to use. For the cloud provider, I initially wanted to use DigitalOcean due to their "free trial" with limited credits. Unfortunately, they suspended my account right after I entered my credit card information. From all the information I gathered on the internet, they have a tendency to block accounts that use a new credit card as a fraud prevention measure. Deciding that contacting them would require too much effort on my part, I chose a local cloud provider in my country, idcloudhost. As for the website tech stacks, I had a few options from the recommendations of others on the internet and decided to use Hugo since I only need a static website for this project of mine.

In my first post, I want to document the steps I took to host my website. Below are the tools I'm using:
1. Linux based OS for the server
2. Hugo as a site generator
3. Apache 2 as web server
4. Git

## A little about Hugo
Hugo is a static website generator. It's fast, simple, free, open-source, and supports multi-language websites. Hugo also has pre-built themes that will help me create my website faster without the need to delve too much into the technical side of web development. The good thing is, if I want more customization, I can also create my own template in the future. Overall, it is not too hard to set up.

## Installing Hugo
I installed Hugo on my Linux VPS for production and on my local PC for development. On the VPS, I installed it using the prebuilt binaries, while I used Docker on my local PC. Before installing anything, I created a folder to store the downloaded file in my `home` directory.

```bash
# Command (Run in Terminal):
mkdir ~/installer
```

### Installing Prerequisites
While these software are not required for all cases, I decided to install all of them for future usage:
1. Git
2. Go
3. Dart Sass

Before doing anything, I ran `sudo apt update` to update the available packages list information in my server.

#### Installing Git
For Git, I just installed it using `apt install`. Then, I validated the installation by running the `git version` command.

```bash
# Install Git
sudo apt install git-all

# Validate installation by checking Git version
git version
# Output
git version 2.43.0
```

#### Installing Go
I went to the official Go website (https://go.dev/doc/install) to get the installer download link. Then, I downloaded the package with `wget`.

```bash
# Command to download file with wget (run in Terminal):
wget <download_link> -P ~/installer/

# Example:
wget https://go.dev/dl/go1.21.1.linux-amd64.tar.gz -P ~/installer/
```

Next, install go with the command below:
```bash
# It will remove the exisiting installation and install the go package in /usr/local
sudo rm -rf /usr/local/go && tar -C ~/installer/ -xzf ~/installer/go1.21.1.linux-amd64.tar.gz && sudo mv ~/installer/go /usr/local/
```

Add these lines to `$HOME/.profile` or `/etc/profile` (for a system-wide installation). I used vi as my text editor. Here's how it looks:

```bash
# /etc/profile: system-wide .profile file for the Bourne shell (sh(1))
# and Bourne compatible shells (bash(1), ksh(1), ash(1), ...).
...
...
# Applications Path    # <-- Add it here at the bottom
GO="/usr/local/go/bin" # <-- Add it here at the bottom
PATH="$PATH:$GO"       # <-- Add it here at the bottom
```

Lastly, re-login to apply the changes of the profile file and verify the installation by running:
```bash
# Validate installation by checking Go version
go version
# Output
go version go1.21.1 linux/amd64
```

#### Installing Dart Sass
Go to the Dart Sass release page, choose the version (I used version 1.66.1), and copy the link. Download it with `wget`:

```bash
# Command to download with wget (run in Terminal):
wget <download_link> -P ~/installer/

# Example:
wget https://github.com/sass/dart-sass/releases/download/1.66.1/dart-sass-1.66.1-linux-x64.tar.gz -P ~/installer/
```

Next, extract the archive file to `~/installer/` directory.

```bash
# Command (run in Terminal):
tar -xvf <archive_file_name> -C ~/installer/

# Example:
tar -xvf ~/installer/dart-sass-1.66.1-linux-x64.tar.gz -C ~/installer/
```

Move the extracted `dart-sass` directory to the `/usr/local/` directory.

```bash
# Example:
sudo mv ~/installer/dart-sass /usr/local/
```

Add these lines to `$HOME/.profile` or `/etc/profile` (for a system-wide installation). Here's how it looks:

```bash
# /etc/profile: system-wide .profile file for the Bourne shell (sh(1))
# and Bourne compatible shells (bash(1), ksh(1), ash(1), ...).
...
...
# Applications Path
DART_SASS="/usr/local/dart-sass" # <-- Add it here
GO="/usr/local/go/bin"
PATH="$PATH:$DART_SASS:$GO"      # <--Edit this line
```

Finally, re-login to apply the changes of the profile file and verify the installation by running:
```bash
# Validate installation by checking Sass version
sass --version
# Output
1.66.1
```

### Installing Hugo - Prebuilt Binaries Installation in Linux
1. Go to this website: https://github.com/gohugoio/hugo/tags
2. Download the version that is going to be used. For my website, I'm using Hugo version 0.119.0.

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
    sudo mv hugo /usr/local/bin/
    ```

5. Run this command to confirm that Hugo is installed correctly.

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
For development, I use a Docker container on my Windows machine. First, I installed Docker Desktop on Windows, following the [Docker official documentation](https://docs.docker.com/desktop/install/windows-install/) for this. Then, I created a custom Docker image using this Dockerfile:

#### Setup Dockerfile

```dockerfile
# Use a base image
FROM ubuntu:24.04 AS base

WORKDIR /tmp

# Update and install required packages
RUN apt-get update && apt-get install -y git wget

# Install Go
RUN wget https://go.dev/dl/go1.21.1.linux-amd64.tar.gz -P /tmp/ && \
    tar -C /usr/local -xzf /tmp/go1.21.1.linux-amd64.tar.gz && \
    rm /tmp/go1.21.1.linux-amd64.tar.gz

# Install Hugo
RUN wget https://github.com/gohugoio/hugo/releases/download/v0.119.0/hugo_extended_0.119.0_linux-amd64.tar.gz -P /tmp/ && \
    tar -C /usr/local/bin -xzf /tmp/hugo_extended_0.119.0_linux-amd64.tar.gz && \
    rm /tmp/hugo_extended_0.119.0_linux-amd64.tar.gz

# Install Dart Sass
RUN wget https://github.com/sass/dart-sass/releases/download/1.66.1/dart-sass-1.66.1-linux-x64.tar.gz -P /tmp/ && \
    tar -C /usr/local -xzf /tmp/dart-sass-1.66.1-linux-x64.tar.gz && \
    rm /tmp/dart-sass-1.66.1-linux-x64.tar.gz

# Set the PATH for Go & Dart Sass
ENV PATH="$PATH:/usr/local/go/bin:/usr/local/dart-sass"

# Expose the port for Hugo server
EXPOSE 1313
```

The Dockerfile provided above will build a Docker image with the following specifications:
- Ubuntu 24.04 as operating system
- go 1.21.1
- hugo extended 0.119.0
- dart-sass 1.66.1

The version of all the listed software can be changed based on needs by modifying the software download links in the Dockerfile.

<!-- I also made an ARG site_option for my docker image. The 0 value mean that I want to use an existing hugo project inside src folder on my host, while 1 value mean I want to make a new hugo project. This ARG site_option can be used in docker compose file later. -->

Lastly, build the Docker image with this command:
```docker build -t de-hugo-dev .```

#### Setup Docker Compose File
This Docker compose file will be used to run a Docker container and serve the website with `hugo server`, using the existing project located at `./srv/http/src`.

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

Note:
- `--poll 700ms` was added to solve [**Issue 1**](#issue-1-file-changes-on-host-not-reflected-in-hugo-web-site)

## Running Hugo in Development
### Creating a New Project
To create a new Hugo project using Docker, I run this command:
```bash
docker run --rm --name de-hugo-dev -p 1313:1313 -v "./srv/http/src:/srv/http/src" --entrypoint "hugo" de-hugo-dev new site . --force
```

The command above will create a new Hugo project and put it inside `./srv/http/src/` on my host. The container will then be immediately stopped and deleted. This part will only be run once to create a new project. For existing project, I will use `docker compose` to create and run the container.

### Run the Docker Container and Serve the Website
To run the Docker container:
```bash
docker compose up -d
```

It will run the development site. Every edit made in the folder `./srv/http/src` will be reflected live on the site (http://localhost:1313).

## Publish the Site in Production
Edit the Hugo configuration file (`config.yml`, `hugo.yaml`, `hugo.toml`, or `hugo.json` - I prefer to use the `config.yml` file) and make the following changes:
1. Set the baseURL for the production site to the VPS domain name.
2. Set the title of the production site.
3. Set the preferred region and language.

```yaml
baseURL: "http://deacputri.com"
languageCode: "en-us"
title: De's Personal Site
```

To publish the site, run:
```hugo```

It will generate the ready-to-publish site in the `public` directory. Then, on the VPS, create the directory `/srv/http/your-website-name`. Copy the generated `public` folder from development to `/srv/http/your-website-name` in production.

Next, edit the Apache2 configuration file:
```bash
sudo vi /etc/apache2/apache2.conf
```

Comment out these lines:
```xml
# <Directory /var/www/>
#         Options Indexes FollowSymLinks
#         AllowOverride None
#         Require all granted
# </Directory>
```

and add the following lines:
```xml
<Directory /srv/http/your-website-name/public/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>
```

Save the file.

Then, edit the default virtual host configuration file:
```bash
sudo vi /etc/apache2/sites-available/000-default.conf
```

Change these lines from:
```xml
<VirtualHost *:80>
    ...
    ...
    DocumentRoot /var/www/html
    ...
    ...
</VirtualHost>
```

to:
```xml
<VirtualHost *:80>
    ...
    ...
    DocumentRoot /srv/http/your-website-name/public
    ...
    ...
</VirtualHost>
```

Finally, restart the web server:
```bash
sudo service apache2 restart
```

Access the website with the domain name using a browser. Note that, the web server will serve "Page Not Found" since the Hugo project is still empty.

## Issue Troubleshooting

### Issue 1 File Changes on Host Not Reflected in Hugo Web Site
<table>
    <tr>
        <td>Issue</td>
        <td>
            When editing development files on the host, the expected behavior was for changes to be immediately reflected at http://localhost:1313. However, although the file changes on the host were successfully reflected in the container, Hugo did not update the website when changes were made.
        </td>
    </tr>
    <tr>
        <td>Cause</td>
        <td>
            Common issue in windows host. Based on <a href="https://github.com/klakegg/docker-hugo/issues/61#issuecomment-1198441388">this source</a>:
            <blockquote cite="https://github.com/klakegg/docker-hugo/issues/61#issuecomment-1198441388">
            "In cases where Hugo CLI is running in a shared directory on linux VM on a windows host the dev server is not detecting file changes from the host environment. (ex: Whenever a docker dev environment is running on a windows host.). This is because by default most watchers will use native file system change detection. And it is not working well in this shared situation."
            </blockquote>
        </td>
    </tr>
    <tr>
        <td>Solution</td>
        <td>
            Added <code>--poll 700ms</code> to the command in the compose file when running "hugo server".
        </td>
    </tr>
</table>
