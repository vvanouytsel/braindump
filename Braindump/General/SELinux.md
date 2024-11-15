#linux #security 

SELinux is **a set of kernel modifications and user-space tools that have been added to various Linux distributions**. Its architecture strives to separate enforcement of security decisions from the security policy, and streamlines the amount of software involved with security policy enforcement.

A fantastic blogpost can be read [here](https://github.blog/developer-skills/programming-languages-and-frameworks/introduction-to-selinux/).

# Working with SELinux

List all the SELinux users.

```bash
# seinfo -u

Users: 8
   guest_u
   root
   staff_u
   sysadm_u
   system_u
   unconfined_u
   user_u
   xguest_u
```

List a mapping between Linux users and SELinux users.

```bash
# semanage login -l

Login Name           SELinux User         MLS/MCS Range        Service

__default__          unconfined_u         s0-s0:c0.c1023       *
root                 unconfined_u         s0-s0:c0.c1023       *
```

If SELinux prevents access to a resource, you can find it in the `/var/log/audit` or `/var/log/audit/audit.log` file. Alternatively you can use tools like `sealert` and `ausearch` to make it easier.

These entries are called 'AVC' and show the relevant subject context, target context, class permissions and type. They look like the following.

```ini
comm=72733A6D61696E20513A526567 name="messages" dev="sda3" ino=255765 scontext=system_u:system_r:syslogd_t:s0 tcontext=unconfined_u:object_r:named_conf_t:s0 tclass=file permissive=0
```

Alternatively you can also use `ausearch` to list AVC denials.

# A Practical Example

Via `ausearch` you get the following output.

```bash
 # ausearch -m AVC,USER_AVC,SELINUX_ERR,USER_SELINUX_ERR -ts recent

----
time->Fri Nov 15 09:16:49 2024
type=PROCTITLE msg=audit(1731658609.669:9412): proctitle="(agent.sh)"
type=PATH msg=audit(1731658609.669:9412): item=0 name="/proc/self/fd/3" inode=9699340 dev=fd:01 mode=0100755 ouid=992 ogid=989 rdev=00:00 obj=unconfined_u:object_r:mnt_t:s0 nametype=NORMAL cap_fp=0 cap_fi=0 cap_fe=0 cap_fver=0 cap_frootid=0
type=CWD msg=audit(1731658609.669:9412): cwd="/"
type=SYSCALL msg=audit(1731658609.669:9412): arch=c000003e syscall=21 success=no exit=-13 a0=7ffcb685a2c0 a1=1 a2=0 a3=3 items=1 ppid=1 pid=5329 auid=4294967295 uid=0 gid=989 euid=0 suid=0 fsuid=0 egid=989 sgid=989 fsgid=989 tty=(none) ses=4294967295 comm="(agent.sh)" exe="/usr/lib/systemd/systemd" subj=system_u:system_r:init_t:s0 key=(null)
type=AVC msg=audit(1731658609.669:9412): avc:  denied  { execute } for  pid=5329 comm="(agent.sh)" name="bamboo-agent.sh" dev="dm-1" ino=9699340 scontext=system_u:system_r:init_t:s0 tcontext=unconfined_u:object_r:mnt_t:s0 tclass=file permissive=0
```

This SELinux audit log entry describes a denied execution attempt. Let's break it down:

1. Timestamp: Fri Nov 15 09:16:49 2024
2. Process Information:
	- Process title: "(agent.sh)"
	- Process ID (pid): 5329
	- Parent Process ID (ppid): 1 (likely systemd)
	- Executable: "/usr/lib/systemd/systemd"
3. Attempted Action:	
	- Syscall: 21 (execve - execute program)
	- Result: Failed (success=no, exit=-13 which means EACCES: Permission denied)
4. File Information:
	- Attempted to execute: "bamboo-agent.sh"
	- Located on device: dm-1
	- Inode: 9699340
5. SELinux Contexts:
	- Source context (scontext): system_u:system_r:init_t:s0  
		(The process running under the init_t domain)
	- Target context (tcontext): unconfined_u:object_r:mnt_t:s0  
		(The file with mnt_t type, typically used for mounted filesystems)
6. Denial Details:
	- Access Vector Cache (AVC) denied the "execute" permission
	- SELinux is in enforcing mode (permissive=0)
7. User/Group Information:
	- UID: 0 (root)
	- GID: 989

SELinux prevented a process (a systemd service running as root) from executing a file named "bamboo-agent.sh". The denial occurred because the file has the mnt_t type, which is not typically to be executed by processes in the init_t domain.

There are a couple ways that we can deal with this:

*  Relabel the file with the correct context (if it's misplaced).
*  Create a custom SELinux policy to allow this specific execution.
*  Move the file to a location where execution is allowed by default.

You can list all the existing contexts via the `fcontext` parameter.

```bash
semanage fcontext -l
```

In our case we will relabel the file/directory.

```bash
semanage fcontext -a -t usr_t "/mnt/data/bamboo-agent-home(/.*)?" 
restorecon -R -v /mnt/data/bamboo-agent-home
```

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
