#linux 

# Working with Processes

Linux stores details about processes in `/proc/$ID/`. 

You can follow the output of a process by tailing the file descriptor. 0 is the standard input, 1 is the standard output and 2 is standard error.

If you want to follow along and see the STDOUT messages:

```bash
# 340978 is the PID, which you can find using 'ps -ef'
tail -f /proc/340978/fd/1
```
