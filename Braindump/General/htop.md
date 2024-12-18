---
tags:
  - linux
---

You can use `htop` to see memory and cpu usage of processes.

## Disable threads

By default every thread is listed in the `htop` output. You can disable this by doing the following:

* Press `F2` to enter the setup.
* Go to the **Display options**.
* Look for **Hide userland threads** and **Hide kernel threads**, and toggle them
* Press `F10` to apply the changes.

This will hide individual threads and show just the main process.