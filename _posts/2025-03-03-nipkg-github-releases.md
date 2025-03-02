---
title: NIPKG feed with GitHub releases
date: 2025-03-02 12:00:00 +0100
categories: [DevOps, NIPKG]
tags: [labview, nipkg, gcd]     # TAG names should always be lowercase
---
## The Wait is Finally Over!

This is something I’ve been waiting for a long time, and I missed the moment when it became possible.
In [Q3 2023](https://www.ni.com/docs/en-US/bundle/package-manager/page/new-behavior-changes.html), the NI Package Manager released a feature that had a rather cryptic description: “Incorporate support for URL redirection, commonly referred to as URL forwarding, in Package URLs.” This feature had been sitting on the [Idea Exchange](https://forums.ni.com/t5/NI-Package-Management-Idea/NIPM-Allow-feeds-to-use-nipkg-files-that-are-hosted-on-GitHub/idi-p/3946753) since 2019, and it’s so exciting that it’s finally been developed!

Initially, I thought it would be a breeze to add a package with this shiny new functionality, but it turned out to be a little more complex than expected. The feature was there, but the nipkg command line did not change as much as I thought it would. NIPKG was able to handle URL redirects, but the problem is how to include a URL to a package in the first place.

For example, the regular `nipkg feed-add-pkg` command does not allow you to add a package from a path that is not relative to the feed path. The `nipkg feed-add-absolute-package` command? Well... it was just weird!

What I thought would work was a command where you pass in the feed directory and the absolute path (or URL) to a package, and voilà, it works! Unfortunately, to use `nipkg feed-add-absolute-package`, the package needs to already be in *another* feed. It’s like a chicken and egg situation. 🐣🥚

I gave it a shot a couple of years ago, but this time, I was truly determined to make it work! Without context, none of this will make much sense—so let’s kick things off with the NIPKG feed!

## Nipkg feed structure
NIPKG feed, regardless of where it is hosted, is composed of three files: Packages, Packages.gz and Packages.stamps.
* Packages (no extensions) is the main file where all the information regarding included packages is stored.
* Packages.gz is simply Packages file compressed with [gzip](https://en.wikipedia.org/wiki/Gzip).
* Packages.stamps is a lookup table of package name and [epoch timestamp](https://www.epochconverter.com/).


**Structure and sample content of the files:**
<br>
![file structure](/assets/img/2025-03/nipkg-feed-file-structure.png)
*File structure*
![packages file content](/assets/img/2025-03/packages.png)
*Packages content*
![packages.stamps file content](/assets/img/2025-03/packages-stamps.png)
*Packages.stamps content*

## Manual solution
Going back to the problem, how can I  add an absolute package reference to a feed? Using `feed-add-absolute-package` was no go. I knew the file structure but didn’t want to start everything from scratch. So, first, I added the package using the regular `nipkg feed-add-pkg` command. Then, I swapped the name with a link [https://github.com/zoryatec/gcd/releases/download/0.23.11/gcd_0.23.11_windows_x64.nipkg](https://github.com/zoryatec/gcd/releases) to a package stored on GitHub Releases.

It wasn’t that simple, though—turns out, a Packages.gz file was required! Since I had already planned to automate this process (if it worked), I built a small console application to compress the Packages content into Packages.gz. And this time… it worked! 

![packages file with link](/assets/img/2025-03/packages-link.png)
<br>
*Packages file wher package name was replaced with url*
<br>


## Automated Solution 🚀  

The driving force behind this functionality was a tool called [GCD - G (LabVIEW) CI/CD](https://github.com/zoryatec/gcd), which I had planned to host on [GitHub Release](https://github.com/zoryatec/gcd/releases). This feature turned out to be the perfect addition!  

Now, with a single command, I can:  
✅ Download a package from a source  
✅ Add it using the regular `nipkg feed-add-pkg`  
✅ Replace the package name with the package URL  
✅ Generate the `Packages.gz` file automatically  

Here’s how it works:  

``` powershell
gcd nipkg feed-local add-http-package `
  --package-http-path 'https://github.com/zoryatec/gcd/releases/download/0.23.11/gcd_0.23.11_windows_x64.nipkg' `
  --feed-local-path 'test-feed' `
  --use-absolute-path `
```

Want to try it out? Just add the feed `https://raw.githubusercontent.com/zoryatec/gcd/refs/heads/main/feed` to your NIPKG — either through the GUI or via the command line.

If your NIPKG version is >= 23.5.0.49296-0+f144, you should be able to download the tool straight from GitHub Releases.

## Disclaimer
⚠️ The tool itself is not production-ready yet!  
⚠️ Unsure why NI does not give the ability to add an absolute package in their command line, and if there is a reason, I don't know!  
⚠️ The feed (not packages) is hosted in the source code repo! I had a serious dilemma here about breaking the separation of concerns principle but wanted it to be self-contained!  


## Further limitations
This process currently works only for publicly availible feeds. The big part of what I am currently working on with [GCD](https://github.com/zoryatec/gcd) is making sharing NIPKG feeds privately through internet (not only localy).
