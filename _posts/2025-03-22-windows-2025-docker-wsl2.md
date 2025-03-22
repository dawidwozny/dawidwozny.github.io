---
title: Docker on Windows 2025 with WSL2 using Github Actions aka How to eat an apple and have an apple.
date: 2025-03-16 12:00:00 +0100
categories: [C#, Patterns]
tags: [C#, patterns, builder]     # TAG names should always be lowercase
mermaid: true
---

# Intro
There will be quite a few side topics in this post therefore will get straigh to the point and then elaborate on couple of things after. What you could learn here is:
* how to run windows docker container with process isolation
* how to run linux docker container with WSL2
* do all above on the same machine simultaneously
* do it on GitHub action hosted agent windows-2025

If you are not interested in reading you can just check the pipline [here](https://github.com/dawidwozny/ms-hosted-windows-2025). You can fork it, if you like, and try it yourself. It works on mine 😆

## Motivation
### Weirdo
I hope nobody will get ofended but I think 'we' developers are a weird group of people. We get genuinely excited about things that most people find absolutely boring. Like, who cares about  composition, dependency inversion or functional programing? Normal person thinks about inheritance at most! We spend days writing software for free on GitHub, while other people are out there enjoying life on other *Hub platforms... like Snapchat or TikTok 😆 And recently started to wonder if being weird in a group of weird people makes me O(n^2) weird or maybe normal? Why I can be consider weird among developers? I use windows docker containers. Yeah, and apart of people I showed this painfull path I don't know anyone in person using it. Even Microsoft publish official image for MSSQL on Linux not on Windows.😆 It's like a secret club… with a really buggy entrance.

### Now serious
Ok, as much as I could make fun of windows containers I use them. Why? Because as any container technology, they solve real problems. There are corner cases where you can't use Linux and still want to benefit from contenerisation. So is this post for somebody who want to learn windows containers? No. This post is for somebody who wan't to get the most of Linux containers but still need to build on Windows.

When you are working with Linux containers, services like MSSQL, MongoDB, RabbitMQ, Nginx, actually any software you can imagine, can be up and running on your machine within a few seconds and I include time to read sparse documentation.

Windows containers ecosystem is not so rich. When I begun my jurney with containers, Microsoft was still supporting developement images for MS SQL on windows. You can check that Windows folder of this [repo](https://github.com/microsoft/mssql-docker) was not updated for the last 4 years. After that, started to develop own images for MSSQL and actually any infrastructure I required in test/dev environments.

Going forward I hope, I will not need to do it anymore. The changes to Window Server should allow to run Linux containers on Windows without need for custom setup. That's why I present this on GitHub action hosted agent. Which is still in [beta](https://github.com/actions/runner-images?tab=readme-ov-file) but [availible](https://github.com/actions/runner-images/issues/11228) for anyone to use.


### Windows Server 2025
Windows Server 2025 has been generaly availible since [4th of November 2024](https://www.microsoft.com/en-us/windows-server/blog/2024/11/04/windows-server-2025-now-generally-available-with-advanced-security-improved-performance-and-cloud-agility/) and it is being shiped with a few features I am actually excited about (and related to this post).

#### WSL2
**WSL** in general allows you to run a Linux environment on Windows and **WSL2** has significant performance improvemts in relation to WSL1. What is most important, it allows you to run Linux docker containers on Windows machine without the need for Hyper-V.

WSL2 was not included in Windows Server 2022 LTSC and only added to Windows Server 2022 some time after the release. This was quite important obstacle. When  using **Windows Containers** you need to match the version of the operating system and the container and that is most of the time LTSC. At least how it was so far. Did not investigated [Portability for containers](https://learn.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/portability) enough to say that changed but it seems it changed.

So in theory I could have WSL2 on Windows Server 2022 but without [Process Isolation](https://learn.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/hyperv-container) for containers, so no go. 

With Windows Server 2025 relase you can run WSL2. 

#### Winget preinstalled
For those who does not know that, Microsoft fianlly created a package manager called Winget which allows you to easily autmoate process instalation. It has been introduced in 2020, roughly after 15 years after dpkg. What a jurney! It was possible to add it to your system before but now it is delivered with the Windows Server 2025 base instalation. 

#### Containers Portability
This is something I have not investigated thoroughly but looks promissing. Loking at this [matrix](https://learn.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility?tabs=windows-server-2025%2Cwindows-11) I can conlude it is possible to run Windows 2022 containers on  Windows Server 2022, 2025 and Windows 11 using process isolation. Thanks to **Containers Portability** it will be possible to use windows containers as containers and not lightweight VMs ;)

## Eat an apple and have an apple
With Windows Server 2025 it should be possible to:
- Run and build as on any windows os
- Run and build windows containers with process isolation
- Run and build linux containers with WSL2

I would not use this approach in production but for testing where I can sacrifice a bit of performance for a bit of convinience why not.

#### Github Action windows-2025

