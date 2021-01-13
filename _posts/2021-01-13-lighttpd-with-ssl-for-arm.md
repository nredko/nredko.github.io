---
title: Cross-compiling lighttpd with SSL for Android ARM
tags: [cross-compilation, adndroid, lighttpd]
style: fill
color: light
description: How to compile lighttpd with SSL support for Android
---

## Problem

During my collaboration with (Grey-Box)[https://www.grey-box.ca/] project we encountered a problem that we need fast and lightweigth http server with SSL support and PHP support.

We decided to use [lighttpd](http://lighttpd.net) since it doesn't require PHP daemon to run permanently.

Existing builds had not included SSL support, and compilation using [native ARM toolchains](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-a/downloads)  also didn't work.

## Solution

Using [Android NDK](https://developer.android.com/ndk) I managed to build lighttpd for Android as well as its requirements: [zlib](https://zlib.net/), [pcre](https://www.pcre.org/) and [openssl](https://www.openssl.org/).

The only module excluded from compilation (so not supported) is [mod_webdav](https://redmine.lighttpd.net/projects/1/wiki/Docs_ModWebDAV), since implementation of this module in `lighttpd` uses features of Linux kernel which are not supported in Android OS.

## Github

Here is the [source project](https://github.com/nredko/lighttpd-arm)
