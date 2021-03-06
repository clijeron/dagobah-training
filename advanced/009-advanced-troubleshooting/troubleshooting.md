layout: true
class: inverse, top, large

---
class: special, middle
# When things go REALLY wrong: Advanced Troubleshooting

slides by @natefoo

.footnote[\#usegalaxy / @galaxyproject]

---
class: special, middle
# Everything always goes wrong

---
# Galaxy UI is slow

- Investigate load
  - Web server(s)
  - Database server
- Investigate memory usage, swapping
- Investigate iowait
  - Use iostat
- Use `gdb` (demo!)

---
# Galaxy UI is slow

- Use heartbeat (demo!)

---
# Local or Network FS slow/down

- Use `iostat` (demo!)
- Use `time dd` (demo!)

Solutions:
- Don't put the Galaxy server in that FS. Local distribution, CVMFS, ??
- Get a better network FS

---
# NFS caching errors

Galaxy gets "No such file or directory" for files that exist. NFS attribute caching is to blame. Set:

```ini
retry_job_output_collection = 5
```

[retry_job_output_collection](https://github.com/galaxyproject/galaxy/blob/dev/lib/galaxy/jobs/__init__.py#L1229)

---
# Hung server processes

- Use `uwsgitop` (demo!)

- `uwsgitop` state busy?
  -  5      12.7    3453    59962   1       13      0       **busy**    89ms    0       0       536.0M  1       0       450m    09:35:36
- Process state D?
  - g2main    3440 17.2  5.2 1696480 863536 ?      **D**    Nov10 167:46 /cvmfs/main.galaxyproject.org/venv/bin/uwsgi --ini /srv/galaxy/main/config/uwsgi.ini

Kernel uninterruptable sleep - normal unless stuck. Probably IO. Check filesystems, disks.

---
# Hung server processes

- Investigate `/proc/pid` especially `fd`
- Use `strace` (demo!)

---
# Undead processes

```
bind(): Address already in use [core/socket.c line 769]
```

Use `lsof -i :<port>` (demo!)

---
# pkill considered useful

```console
$ sudo pkill -INT -u galaxy uwsgi
```

---
# Remember ALL the log files

All may be relevant:
- uWSGI log
- handler logs
- Pulsar logs
- nginx access, error logs
- syslog/messages
- authlog
- browser console log

---
# Data Manager loading failure

![dm404-1.png](images/dm404-1.png)

---
# Data manager loading failure

![dm404-2.png](images/dm404-2.png)

---
# Data manager loading failure

![dm404-3.png](images/dm404-3.png)

---
# Data manager loading failure

Solution: Add to `/etc/apache2/sites-available/000-galaxy.conf`:
```apache
AllowEncodedSlashes on
```

---
# Database problems

Slow queries, high load, etc.

- Use `EXPLAIN ANALYZE` (demo!)
- Use `VACUUM ANALYZE`

Increase `shared_buffers`. 2GB on Main (16GB of memory on VM)

---
# error: [Errno 32] Broken pipe

This error is not indicative of any kind of failure. It just means that the client closed a connection before the server finished sending a response.

---
# Installation failures - Tool dependencies

```
utils.c:33:18: fatal error: zlib.h: No such file or directory
compilation terminated.
make: *** [utils.o] Error 1
```

Find `zlib.h` on [packages.ubuntu.com](http://packages.ubuntu.com/)

---
# Installation failures - Python dependencies

```
psutil/_psutil_linux.c:12:20: fatal error: Python.h: No such file or directory
compilation terminated.
error: command 'x86_64-linux-gnu-gcc' failed with exit status 1
```

Find `Python.h` on [packages.ubuntu.com](http://packages.ubuntu.com/)

---
# uWSGI errors

```
uwsgi: unrecognized option '--ini-paste'
```

Make sure you know which uWSGI you're running.

`/usr/bin/python` needs the `uwsgi-plugins-python` package and requires:

```console
$ uwsgi --plugin python
```

---
# Job failures

"Unable to run job due to a misconfiguration of the Galaxy job running system.  Please contact a site administrator."

There is a traceback in the Galaxy log. Go find it.

---
# Node failures

```console
$ sinfo -Nel
Fri Nov 11 09:50:01 2016
NODELIST   NODES PARTITION       STATE CPUS    S:C:T MEMORY TMP_DISK WEIGHT FEATURES REASON
localhost      1    debug*        down    2    2:1:1      1        0      1   (null) Node unexpectedly rebooted
```

Figure out why it rebooted and:

```console
$ sudo scontrol update nodename=localhost state=resume
```

---
class: special, middle
# Any troubling Galaxy situations you have?
