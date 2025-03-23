---
title: Windows + Linux containers on Windows 2025 with WSL2 and Github Actions runner
date: 2025-03-22 12:00:00 +0100
categories: [DevOps, Docker]
tags: [devops, docker, windows, linux]     # TAG names should always be lowercase
---
![Windows + Linux containers on Windows 2025 with WSL2 and Github Actions runner](/assets/img/2025-03/windows-2025.png)


# Intro
This post will cover several side topics, so I'll start by listing the main points and then dive deeper into a few key areas. Here's what you can learn:
* how to run **Windows Server 2022 LTSC** container on Windows Server 2025 LTSC with process isolation
* how to run **Windows Server 2025 LTSC** container on Windows Server 2025 LTSC with process isolation
* how to run **MSSQL** Linux container with **WSL2**
* do all above on the same machine simultaneously
* do it on GitHub action hosted agent **windows-2025**

If you are not interested in reading, you can just check the pipline [here](https://github.com/dawidwozny/ms-hosted-windows-2025). You can fork it, if you like, and try it yourself. It works on mine 😆

## Motivation
### Weirdo
I hope nobody will get ofended but I think 'we' developers are a weird group of people. We get genuinely excited about things that most people find absolutely boring. Like, who cares about  composition, dependency inversion or functional programing? Normal person thinks about inheritance at most! We spend days writing software for free on GitHub, while other people are out there enjoying life on other *Hub platforms... like Snapchat or TikTok 😆 And recently started to wonder if being weird in a group of weird people makes me O(n^2) weird or maybe normal? Why can I be consider weird among developers? I use Windows Containers. Yeah, and apart of people I showed this painfull path I don't know anyone in person using it. Even Microsoft publish official image for MSSQL on Linux, not on Windows.😆 It's like a secret club… with a really buggy entrance.

### Being serious
Ok, as much as I could make fun of Windows Containers I use them. Why? Because as any container technology, they solve real problems. There are corner cases where you can't use Linux and still want to benefit from containerisation. Is this post for somebody who want to learn Windows Containers? No. This post is for somebody who wan't to get the most of Linux containers but still need to build on Windows.

When you are working with Linux containers, services like MSSQL, MongoDB, RabbitMQ, Nginx, actually any software you can imagine, can be up and running on your machine within a few seconds and I also include time to read sparse documentation.

Windows containers ecosystem is not so rich. When I begun my jurney with containers, Microsoft was still supporting developement images for MS SQL on Windows. You can check that Windows folder of this [repo](https://github.com/microsoft/mssql-docker) was not updated for the last 4 years. After that, started to develop own images for MSSQL and actually any infrastructure I required in test/dev environments.

Going forward I hope, I will not need to do it anymore. The changes to Window Server should allow to run Linux containers on Windows without need for custom setup. That's why I present this on GitHub action hosted agent. Which is still in [beta](https://github.com/actions/runner-images?tab=readme-ov-file) but [availible](https://github.com/actions/runner-images/issues/11228) for anyone to use.

### Windows Server 2025 LTSC
Windows Server 2025 has been generaly availible since [4th of November 2024](https://www.microsoft.com/en-us/windows-server/blog/2024/11/04/windows-server-2025-now-generally-available-with-advanced-security-improved-performance-and-cloud-agility/) and it is being shiped with a few features I am actually excited about (and related to this post).

#### WSL2
**WSL** in general allows you to run a Linux environment on Windows and **WSL2** has significant performance improvemts in relation to WSL1. What is most important, it allows you to run Linux docker containers on Windows machine without the need for Hyper-V.

WSL2 was not included in Windows Server 2022 LTSC and only added to Windows Server 2022 SAC some time after the release. This was quite important obstacle. When  using Windows Containers you need to match the version of the operating system and the container and that is most of the time LTSC. At least how it was so far, although changed with [Container Portability](https://learn.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/portability) implementation.

So in theory I could have WSL2 on Windows Server 2022 but without [Process Isolation](https://learn.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/hyperv-container) for windows containers, no go.

With Windows Server 2025 relase you can run WSL2 off the shelf.

#### Winget preinstalled
For those who does not know that, Microsoft fianlly created a package manager called Winget which allows you to easily autmoate process instalation. It has been introduced in 2020, roughly after 15 years after dpkg. What a jurney! It was possible to add it to your system before but now it is delivered with the Windows Server 2025 base instalation. 

#### Containers Portability
[Containers Portability](https://learn.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/portability) was something highly anticipated and makes Windows Containers working more like Linux ones where you don't need to match exact version of container and host. It has actually been added to version 23H2 in [July 2023](https://techcommunity.microsoft.com/blog/containers/portability-with-windows-server-annual-channel-for-containers/3885911) but it is first time being part of LTSC version.

#### Github Action windows-2025
As I mentioned earlier the setup was done on new GitHub Action runner [windows-2025](https://github.com/actions/runner-images/issues/11228) as for now 2025-03-22 is still in beta.

## Soulution
I have build a [pipline](https://github.com/dawidwozny/ms-hosted-windows-2025/blob/main/.github/workflows/main.yml) in dedicated [repo](https://github.com/dawidwozny/ms-hosted-windows-2025) which you can fork and play yourself.

Here I will give some comments on the steps in pipeline:
### Checkout with LFS
Wanted to make it a bit more usefull therfore added fake test database backup which is stored through Git LFS.
``` yml
- uses: actions/checkout@v4
    with:
        lfs: true
```
### Install Linux
Instaling Linux distribution. It takes around 40 seconds.
``` powershell
wsl --install Ubuntu
```

### Prevent WSL shutdonw
Little trick to prevent WSL to shutdown automatically. By default when using `wsl -- <command>` syntax, WSL will shutdown automatically after a few seconds along with all containers.
``` powershell
wsl -d Ubuntu  --exec dbus-launch true   
```
### Install docker on WSL
I had some issues with bash script created on windows. `dos2unix` did a trick.
It takes around 40 seconds.
```powershell
wsl -- sudo apt-get update
wsl -- sudo apt-get install dos2unix
wsl -- dos2unix ./docker-install.sh
wsl -- ./docker-install.sh
```

#### docker-install.sh
It is copy paste of official docker instalation [instruction](https://docs.docker.com/engine/install/ubuntu/).
``` bash
#!/bin/sh
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
# install docker
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

### Compose Up
I could just run `docker run` but wanted to make it a bit more interesting and restore db from backup. With compose it is just simpler. Prempting criticism: this setup meant to be simple not secure.
Pulling and running takes below 60 seconds. Then I wait for database server to start being operational.

``` powershell
wsl -- sudo docker compose pull -q  
wsl -- sudo docker compose -p test-setup up -d
```

``` yaml
services:
  db:
    image: mcr.microsoft.com/mssql/server:2022-latest
    ports:
      - "1433:1433"
    environment:
     - MSSQL_SA_PASSWORD=SecretPassword9!
     - ACCEPT_EULA=Y
    volumes:
      - ./database/data:/var/opt/mssql/data
      - ./database/log:/var/opt/mssql/log
      - ./database/backup:/var/opt/mssql/backup
  web:
    image: nginx
    ports:
      - "8080:80"
```
### Test curl inside WSL
Returns startup nginx content.
```powershell
wsl -- curl http://localhost:8080
```

### Test request from Windows host
Returns the same output as curl.
```powershell
Invoke-WebRequest -Uri http://localhost:8080
```

### Run initial test query
>Note: Used address is 127.0.0.1 and not localhost. Had some issues with networking which I will be still investigating so it is not plain sailing.
```powershell
Invoke-Sqlcmd `
    -ServerInstance "127.0.0.1,1433" `
    -Database master `
    -Username "sa" `
    -Password "SecretPassword9!" `
    -TrustServer `
    -Query "SELECT name FROM sys.databases" 
```

### Restore database from backup 
Backup folder is mounted during docker compose then need to run the following:
``` powershell
Invoke-Sqlcmd `
    -ServerInstance "127.0.0.1,1433" `
    -Database master `
    -Username "sa" `
    -Password "SecretPassword9!" `
    -TrustServer `
    -Query "RESTORE DATABASE test FROM DISK = '/var/opt/mssql/backup/test.bak' WITH REPLACE;"
```

### Run sample query on resored database
```powershell
Invoke-Sqlcmd `
    -ServerInstance "127.0.0.1,1433" `
    -Database test `
    -Username "sa" `
    -Password "SecretPassword9!" `
    -TrustServer `
    -Query "SELECT * FROM Person;" 
```

### Run Windows 2022 container
This is proof that you can run Windows Server 2022 image on Windows Server 2025 host.
I have added flag `--isolation=process` just to be explicit about it but this command would normally fail anyway since GitHub runner does not have nested virutalisation and Hyper-V installed.

```powershell
docker run -d -p 8081:8080 --name aspnetcore_sample mcr.microsoft.com/dotnet/samples:aspnetapp-8.0-nanoserver-ltsc2022 --isolation=process
```

### Run Windows 2025 container
There is no ASP.NET sample availible for 2025 therfore just run powershell command. Did not want to overcomplicate it with my own application.

```powershell
docker run --rm --isolation=process mcr.microsoft.com/windows/servercore:ltsc2025-amd64 powershell -Command "Write-Host 'Hello from the other side!'" 
```

### Test linux container still works
``` powershell
Invoke-WebRequest -Uri http://localhost:8080
```

### Test ASP.NET LTSC 2022 container works
``` powershell
Invoke-WebRequest -Uri http://localhost:8081
```

## Summary
That's it. It is possible. GitHub runner is still in beta so this approach is not ready yet but looking forward using it soon. Don't get me wrong here. I would not run Linux container on Windows in production unless Microsoft says explicitly it is supported. For building test/dev environments I am willing to give it a go.
