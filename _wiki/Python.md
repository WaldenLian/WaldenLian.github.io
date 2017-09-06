---
layout: wiki
title: Python
categories: Python
description: Common issues in Python
keywords: Python
---

### Common issues with Python

* Sometimes the `pip` install will fail due to the poor network state. One solution is to set a longer timeout:  `pip --default-timeout=100 install -U pip` (-U here is to upgrade the target package to the latest version if already installed)

