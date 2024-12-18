---
tags:
  - virtualisation
  - qemu
  - vm
---

With Libvirt we can convert our workstation into a hypervisor and run VMs on it.

## Installation

```bash
 ❯ sudo dnf install libvirt -y
 ❯ sudo dnf install virt-install -y
 ❯ sudo dnf install virt-viewer -y
```

## Creating a VM

* Download the ISO of your choice
* Start `virt-viewer`, select `New` > `Virtual Machine` and simply follow the wizard.