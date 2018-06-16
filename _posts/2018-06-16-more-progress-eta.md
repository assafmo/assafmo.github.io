---
layout: post
title: Some more progress and ETA tricks
date: 2018-06-16
---

In an [earlier post](https://assafmo.github.io/2017/08/02/pv-eta.html) I wrote about `pv -d`. Here are some more tricks that can be even easier to use.

### Reading to stdout with pv instead of cat

Reading files to stdout can be done with pv instead of cat. pv will also print transfer rate, ETA and a progress bar to stderr.

For example, counting packets in a pcap file:

```bash
➜  ls -lh
total 1.0G
-rw-rw-r-- 1 assafmo assafmo 1.0G Jun 10 20:51 my.pcap
➜  pv my.pcap | tcpdump -r /dev/stdin -qn | wc -l
reading from file /dev/stdin, link-type EN10MB (Ethernet)
512MiB 0:00:10 [26.3MiB/s] [================>                 ] 50% ETA 0:00:09
```

Result:

```bash
➜  pv my.pcap | tcpdump -r /dev/stdin -qn | wc -l
reading from file /dev/stdin, link-type EN10MB (Ethernet)
1023MiB 0:00:19 [44.8MiB/s] [================================>] 100%
8635943
```

### Using `pv -s` in between pipes

Placing pv in between pipes will make it read from stdin and print to stdout (just like cat), and also print the transfer rate to stderr.

By using the `-s` flag we can tell pv the total size of the data being transferred, thus pv will also print progress and ETA to stderr.

For example, merging multiple pcap files together:

```bash
➜  ls -lh
total 3.0G
-rw-rw-r-- 1 assafmo assafmo 1.0G Jun 10 20:51 1.pcap
-rw-rw-r-- 1 assafmo assafmo 1.0G Jun 10 20:51 2.pcap
-rw-rw-r-- 1 assafmo assafmo 1.0G Jun 10 20:51 my.pcap
➜  joincap *.pcap | pv -s $(du -bc *pcap | tail -1 | cut -f 1) > merged.pcap
1.16GiB 0:00:04 [ 275MiB/s] [============>                    ] 38% ETA 0:00:06
```

### Using `parallel --bar` to count done jobs

Finally, GNU parallel can print how many jobs are done and an ETA with the `--bar` flag.

For example, counting packets in multiple pcaps:

```bash
➜  ls *pcap | parallel --bar 'tcpdump -r "{}" -qn | wc -l > "{}".count'
66% 2:1=1s my.pcap
```
