#linux #security 

SELinux is **a set of kernel modifications and user-space tools that have been added to various Linux distributions**. Its architecture strives to separate enforcement of security decisions from the security policy, and streamlines the amount of software involved with security policy enforcement.

# Troubleshooting

Show recent blocked actions by SELinux.

```bash
ausearch -m AVC,USER_AVC,SELINUX_ERR,USER_SELINUX_ERR -ts recent
journalctl -t setroubleshoot
```

Temporary switch to permissive until next boot.

```bash
setenforce 0
```

Enable full path logging. 
This will make your journalctl logs a lot more readable. However you probably only want to enable this to reproduce the problem and disable it afterwards.

```bash
# Enable full path logging
auditctl -w /etc/shadow -p w -k shadow-write

# Disable the cache
rm -f /var/lib/setroubleshoot/setroubleshoot.xml

# Reproduce your problem

# Disable full path logging
auditctl -W /etc/shadow -p w -k shadow-write
```

Show more details of SELinux denials.

```bash
sealert -l "*
```

References: [Troubleshooting Problems Related to SELinux :: Fedora Docs](https://docs.fedoraproject.org/en-US/quick-docs/selinux-troubleshooting/)

