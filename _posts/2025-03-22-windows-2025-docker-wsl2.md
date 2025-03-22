---
title: Docker on Windows 2025 with WSL2 using Github Actions aka How to eat an apple and have an apple.
date: 2025-03-16 12:00:00 +0100
categories: [C#, Patterns]
tags: [C#, patterns, builder]     # TAG names should always be lowercase
mermaid: true
---

#Intor

## Weirdo
I hope nobody will get ofended but I think 'we' developers are weird group of people. We are excited about something most of the people does not care about. TODO


### Windows Server 2025
Windows Server 2025 has been generaly availible since [4th of November 2024](https://www.microsoft.com/en-us/windows-server/blog/2024/11/04/windows-server-2025-now-generally-available-with-advanced-security-improved-performance-and-cloud-agility/) and it is being shiped with a few features I am actually excited about.

#### WSL2
**WSL** generally allows you to run a Linux environment on Windows and **WSL2** has significant performance improvemts in relation to WSL1. What is most important, it allows you to run Linux docker containers on Windows machine without the need for Hyper-V.

WSL2 was not included in Windows Server 2022 LTSC and only added to Windows Server 2022 some time after the release. This was quite important obstacle. When  using **Windows Containers** you need to match the version of the operating system and container and that is most of the time LTSC. At least how it was so far. Did not investigated [Portability for containers](https://learn.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/portability) enough to say that changed but it seems it changed.

So in theory I could have WSL2 on Windows Server 2022 but without [Process Isolation](https://learn.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/hyperv-container) for containers, so no go. 

With Windows Server 2025 relase you can run WSL2. 

#### Winget preinstalled
For those who does not know that, Microsoft fianlly created a package manager called Winget which allows you to easily autmoate process instalation. It has been introduced in 2020, roughly after 15 years after dpkg. What a jurney! It was possible to add it to your system before but now it is delivered with the Windows Server 2025 base instalation. 

#### Containers Portability
This is something I have not investigated thoroughly but looks promissing. 
