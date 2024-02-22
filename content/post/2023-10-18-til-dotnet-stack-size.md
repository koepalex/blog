---
title: "Dotnet default stack size"
date: "2023-10-18"
categories: 
  - "TIL"
tags: 
  - "dotnet"
  - "threads"
  - "unix"
---

**T**oday **I** **L**earned that the default stack size for threads in dotnet e.g. the ThreadPool threads is OS dependent. On Windows it is _1.5 MiB_, on Linux, MacOs it is dependent on the concrete OS version. To determine the actual default thread size you have to run `ulimit -s`. For Ubuntu 22.04 it is _8192 bytes_ and for macOS 14.2 it is _8176 bytes_.

It is possible to configure the default stack size via environment variable e.g. to set the stack to 1.5Mib set `DOTNET_DefaultStackSize=180000` (the value is interpreted as hex).