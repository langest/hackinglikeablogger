---
layout: post
title:  "Gentoo kernel update notes"
date:   2022-10-29 15:02:00 +0200
categories: gentoo kernel notes
---
# Gentoo kernel update notes

Build and install kernel.
```
# make && make install
```

Configure grub to use the new kernel.
```
# grub-mkconfig -o /boot/grub/grub.cfg
```

Build modules and packages depending on modules.
```
# make modules_prepare
# make modules
# make modules_install
# emerge --ask @module-rebuild
```
