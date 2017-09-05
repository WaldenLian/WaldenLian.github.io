---
layout: wiki
title: Linux
categories: Linux
description: Linux 常用操作。
keywords: Linux
---

### Common commands in daily usage

* Compare the content inside two directories:  `diff -ruNa s1 s2`
* Methods to exit the `ssh` while keeping the current process running:
  * `screen` command:
    * Enter a new screen session: `screen`
    * Detach the current session while keeping the current process running: `Ctrl+A d`
    * Resume the previous sessions: `screen -x <host_id>`