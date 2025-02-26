---
title: NIPKG feed with GitHub releases
date: 2025-02-26 12:00:00 +0100
categories: [DevOps, NIPKG]
tags: [labview, nipkg, gcd]     # TAG names should always be lowercase
---
## 🎉 The Wait is Finally Over! 🎉

This is something I’ve been eagerly anticipating for a long time, and I missed the moment when it became possible. 🙌
In Q3 2023, the NI Package Manager released a feature that had a rather cryptic description: “Incorporate support for URL redirection, commonly referred to as URL forwarding, in Package URLs.” This feature had been sitting on the Idea Exchange since 2019, and it’s so exciting that it’s finally been developed! 😍

Initially, I thought it would be a breeze to add a package with this shiny new functionality, but it turned out to be a little more complex than expected. 😅 The feature was there, but the nipkg command line did not change as much as I thought it would.

For example, the regular `nipkg feed-add-pkg` command does not allow you to add a package from a path that is not relative to the feed path. And the `nipkg feed-add-absolute-package` command? Well... it was just weird! 🤷‍♂️

What I thought would work was a command where you pass in the feed directory and the absolute path (or URL) to a package, and voilà, it works! Unfortunately, to use nipkg feed-add-absolute-package, the package needs to already be in another feed. It’s like a chicken and egg situation. 🐣🥚

## Nipkg feed structure

## Manual solution

## Automated solution

## Disclaimer
