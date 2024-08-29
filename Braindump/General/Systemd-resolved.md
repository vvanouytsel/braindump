#linux #sytemd

## Enable Systemd-resolved

- Use the local stub file used by systemd

```bash
# Symlink /etc/resolv.conf to the stub file
$ ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
```

- Configure the local stub file using systemd-resolved

```bash
# Configure DNS servers
$ vim /etc/systemd/resolved.conf.d/dns_servers.conf

[Resolve]
DNSSEC=no
Cache=yes
DNS=8.8.4.4 8.8.8.8
Domains=company.net
DNSStubListener=yes
```

- Restart systemd-resolved

```bash
$ systemctl restart systemd-resolved
```

- Check if the nameservers are used

```bash
$ systemd-resolved --status

Global
         DNS Servers: 8.8.4.4
                      8.8.8.8
```

## Disable Systemd-resolved

- Use resolv.conf, not managed by systemd-resolved

```bash
# Remove symlink to the stub file
$ rm -rf /run/systemd/resolve/stub-resolv.conf
```

References:  
[https://www.freedesktop.org/software/systemd/man/systemd-resolved.service.html](https://www.freedesktop.org/software/systemd/man/systemd-resolved.service.html "https://www.freedesktop.org/software/systemd/man/systemd-resolved.service.html") [https://wiki.archlinux.org/index.php/Systemd-resolved#Configuration](https://wiki.archlinux.org/index.php/Systemd-resolved#Configuration "https://wiki.archlinux.org/index.php/Systemd-resolved#Configuration")

## Resolvectl

* List domain settings

```bash
 ❯ resolvectl domain                          
Global:
Link 2 (wwan0):
Link 3 (enp0s31f6):
Link 4 (wlp0s20f3): company.net
```

* Set search domain for specific interface

```bash
❯ sudo resolvectl domain wlp0s20f3 company.net
```