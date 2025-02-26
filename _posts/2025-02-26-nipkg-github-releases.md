---
title: NIPKG feed with GitHub releases
date: 2025-02-26 12:00:00 +0100
categories: [DevOps, NIPKG]
tags: [labview, nipkg, gcd]     # TAG names should always be lowercase
---
## The Wait is Finally Over!

This is something I’ve been eagerly anticipating for a long time, and I missed the moment when it became possible.
In [Q3 2023](https://www.ni.com/docs/en-US/bundle/package-manager/page/new-behavior-changes.html), the NI Package Manager released a feature that had a rather cryptic description: “Incorporate support for URL redirection, commonly referred to as URL forwarding, in Package URLs.” This feature had been sitting on the [Idea Exchange](https://forums.ni.com/t5/NI-Package-Management-Idea/NIPM-Allow-feeds-to-use-nipkg-files-that-are-hosted-on-GitHub/idi-p/3946753) since 2019, and it’s so exciting that it’s finally been developed!

Initially, I thought it would be a breeze to add a package with this shiny new functionality, but it turned out to be a little more complex than expected. The feature was there, but the nipkg command line did not change as much as I thought it would.

For example, the regular `nipkg feed-add-pkg` command does not allow you to add a package from a path that is not relative to the feed path. The `nipkg feed-add-absolute-package` command? Well... it was just weird!

What I thought would work was a command where you pass in the feed directory and the absolute path (or URL) to a package, and voilà, it works! Unfortunately, to use `nipkg feed-add-absolute-package`, the package needs to already be in *another* feed. It’s like a chicken and egg situation. 🐣🥚

Remember trying that couple years ago but this time was really determined to make it work. All I say won't make sense without a context. Let's start with NIPKG feed. 

## Nipkg feed structure
NIPKG feed, regardless of where it is hosted, is composed of three files: Packages, Packages.gz and Packages.stamps.
* Packages (no extensions) is the main file where all the information regarding included packages is stored.
* Packages.gz is simply Packages file compressed with [gzip](https://en.wikipedia.org/wiki/Gzip).
* Packages.stamps is a lookup table of package name and [epoch timestamp](https://www.epochconverter.com/).
<br>
Sample content of these files for one package can be seen below.
<br>
![file structure](/assets/img/2025-02/nipkg-feed-file-structure.png)
![packages file content](/assets/img/2025-02/packages.png)
![packages.stamps file content](/assets/img/2025-02/packages-stamps.png)

## Manual solution
So the functionality was there but there was no way to add absolute package reference easily. I have worked with NIPKG for couple years now so was not scared of modyfing feed files manually.  First, I have replaced the name with link to a pakcage stored on [GitHub Release](https://github.com/zoryatec/gcd/releases) but it didn't - Packages.gz file was required. I had allready planed to automate the process, so developed little console application to compress Packages content into Packages.gz. It worked. 
![packages file with link](/assets/img/2025-02/packages-link.png)

## Automated solution
The main drive for that functionality was a tool called [GCD - G (LabVIEW) CI/CD](https://github.com/zoryatec/gcd) that I had planed to host on [GitHub Release](https://github.com/zoryatec/gcd/releases) and this functionality was perfect addition. Now I can just run the command below which will
* download a package from a source
* add a package with usuall `nipkg feed-add-pkg`
* replace package name with package url
* generage Packages.gz file

``` powershell
gcd nipkg feed-local add-http-package `
  --package-http-path 'https://github.com/zoryatec/gcd/releases/download/0.23.11/gcd_0.23.11_windows_x64.nipkg' `
  --feed-local-path 'test-feed' `
  --use-absolute-path `
```

If you want to try you can add feed `https://raw.githubusercontent.com/zoryatec/gcd/refs/heads/main/feed` to your NIPKG either through GUI or command line. Provided that your NIPKG version is >= 23.5.0.49296-0+f144 then you should be able to just download a tool which is hosted on GitHub Releases.

## Disclaimer
* ⚠ The tool is not production ready yet.
* ⚠ Don't know if it will always work but it works for my use cases.
* ⚠ I hope NI will finally develop similar command.
* ⚠ The feed (not packages) is hosted in source code repo. I had serious dilema here about braking separations of concerns principle but wanted that to be self-contained. 

## Limitations
This process currently works only for publicly availible feeds. The big part of what I am currently working on with [GCD](https://github.com/zoryatec/gcd) is making sharing NIPKG feeds privately through internet (not only localy).
