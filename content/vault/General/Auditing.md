#linux #security

- You can use `auditctl` and `ausearch` to figure out which process (and when) changed a file
- Create a test file

```sh
$ echo "test1" >> /tmp/testfile.txt
```

- Use auditctl to monitor that file

```sh
$ /sbin/auditctl -w /tmp/testfile.txt -p war
```

- Alter the file

```sh
$ echo "test2" >> /tmp/testfile.txt
```

- Use ausearch to view what happened after file has been changed

```sh
$ /sbin/ausearch -f /tmp/testfile.txt

----
time->Wed Sep 21 17:17:41 2022
type=PROCTITLE msg=audit(1663773461.058:158563): proctitle="-bash"
type=PATH msg=audit(1663773461.058:158563): item=1 name="/tmp/testfile.txt" inode=737307 dev=00:26 mode=0100644 ouid=0 ogid=0 rdev=00:00 obj=unconfined_u:object_r:user_tmp_t:s0 objtype=NORMAL cap_fp=0000000000000000 cap_fi=0000000000000000 cap_fe=0 cap_fver=0
type=PATH msg=audit(1663773461.058:158563): item=0 name="/tmp/" inode=9994 dev=00:26 mode=041777 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:tmp_t:s0 objtype=PARENT cap_fp=0000000000000000 cap_fi=0000000000000000 cap_fe=0 cap_fver=0
type=CWD msg=audit(1663773461.058:158563):  cwd="/root"
type=SYSCALL msg=audit(1663773461.058:158563): arch=c000003e syscall=2 success=yes exit=3 a0=df7ed0 a1=441 a2=1b6 a3=0 items=2 ppid=13642 pid=13646 auid=2046 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=36 comm="bash" exe="/usr/bin/bash" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="hosts-file"
----
time->Wed Sep 21 17:18:03 2022
type=PROCTITLE msg=audit(1663773483.783:158573): proctitle="-bash"
type=PATH msg=audit(1663773483.783:158573): item=1 name="/tmp/testfile.txt" inode=737307 dev=00:26 mode=0100644 ouid=0 ogid=0 rdev=00:00 obj=unconfined_u:object_r:user_tmp_t:s0 objtype=NORMAL cap_fp=0000000000000000 cap_fi=0000000000000000 cap_fe=0 cap_fver=0
type=PATH msg=audit(1663773483.783:158573): item=0 name="/tmp/" inode=9994 dev=00:26 mode=041777 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:tmp_t:s0 objtype=PARENT cap_fp=0000000000000000 cap_fi=0000000000000000 cap_fe=0 cap_fver=0
type=CWD msg=audit(1663773483.783:158573):  cwd="/root"
type=SYSCALL msg=audit(1663773483.783:158573): arch=c000003e syscall=2 success=yes exit=3 a0=dfece0 a1=441 a2=1b6 a3=0 items=2 ppid=13642 pid=13646 auid=2046 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=36 comm="bash" exe="/usr/bin/bash" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="hosts-file"
```

- This gives information about which process (pid) and parent process (ppid) was used. So as soon as you know the pid/ppid, you can look up which process that is.

```sh
$ ps -p 13646

  PID TTY          TIME CMD
13646 pts/0    00:00:02 bash
```

- In this case you can see the the file is altered using `bash`
- [Reference](https://serverfault.com/questions/320716/find-out-which-process-is-changing-a-file "https://serverfault.com/questions/320716/find-out-which-process-is-changing-a-file")