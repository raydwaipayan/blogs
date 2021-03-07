---
title: "Linux Kernel checkpatch.pl mentorship"
date: 2021-03-06T02:05:17+05:30
draft: false
---

### Checkpatch - What is it?
[checkpatch.pl](https://github.com/torvalds/linux/blob/master/scripts/checkpatch.pl) is a trivial style checker for patches sent to the linux kernel.

Why is it needed you say?</br>
I did an analysis on checkpatch warnings on about 50k commits from v5.4.</br>
(Find the full error dump {{% link checkpatch_out_50k.txt %}}here{{% /link %}})

The top errors were then generated via the following shell command:
```/bin/bash
cat checkpatch_out_50k.txt | grep -oP "(?:WARNING|ERROR|CHECK):(\w+)" \
| sort | uniq -c | sort -nr  | head -n6
```

Count of errors | Error Types | Error
----------------|-------------|------
41999 | <span style="color:green">CHECK</span> | CAMELCASE
13629 | <span style="color:green">CHECK</span> | PARENTHESIS_ALIGNMENT
11491 | <span style="color:orange">WARNING</span> | LONG_LINE
8106 | <span style="color:green">CHECK</span> | SPACING
6825 | <span style="color:orange">WARNING</span> | LEADING_SPACE
4394 | <span style="color:green">CHECK</span> | OPEN_ENDED_LINE

Hmm...The no. of stylistically incorrect bits still continue to exist and
are still merged into the kernel code. To make the kernel code more
stylistically coherent, we can:

- Create cleanup patches for the existing code
- Make sure new patches are stylistically consistent

Both of these are where checkpatch comes handy.