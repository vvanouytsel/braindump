---
tags:
  - linux
---

## Dd

* Wipe the partition table, boot loader and other metadata from a disk

```bash
$ dd if=/dev/zero of=/dev/sdX bs=1M count=10
```