---
title: "Linux Kernel checkpatch.pl mentorship"
date: 2021-03-06T02:05:17+05:30
draft: false
---

### Checkpatch - What is it?
[checkpatch.pl](https://github.com/torvalds/linux/blob/master/scripts/checkpatch.pl) is a trivial style checker for patches sent to the linux kernel.

Why is it needed you say?</br>
I did an analysis on checkpatch warnings on about 50k commits from v5.4.</br>
(Find the full report {{% link checkpatch_out_50k.txt %}}here{{% /link %}})

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

> - Create cleanup patches for the existing code
> - Make sure new patches are stylistically consistent

Both of these are where checkpatch comes handy.

### Checkpatch in the patch submission process

Before sending out the patches to the mailing list, it is recommended to
run checkpatch over the patches.

In general, a Kernel developer would:

> - Create the patch with his proposed changes.
> - Build check the patch, run additional tools like checkstack, sparse etc.
> - Run checkpatch.pl over the patches to eliminate all trivial style violations.
> - Send out the patch to the maintainers and mailing list.


The only catch here is that checkpatch is not always right. It is just a dumb
perl script. If the code looks better with checkpatch errors, let it be.

### Drawbacks of checkpatch

Checkpatch is only a static trivial style checker. It does not have the
capabilities of a full c based parser like [clang-format](https://www.kernel.org/doc/html/latest/process/clang-format.html). In this sense it is limited in
what it can do. Still, it is highly valuable as a style checker tool and can
ease the work of maintainers and increase the chances of the patch being accepted
by the community.

### What did we do to improve it?

Over the first 3 months of my mentorship period, I primarily focussed on
eliminating false-positives from checkpatch as well as adding new rules.

Consider the following patch commit log:

```diff
#include asm/bitsperlong.h avoid further potential userspace pollution
by moving the #define of SHIFT_PER_LONG to bitops.h which is not
exported to userspace.
```

A kernel developer in 2020 would not see any checkpatch warning.

A kernel developer in 2021 would see the following warning:
```\bin\bash
WARNING: Commit log lines starting with '#' are dropped by git as comments
#82: 
#include asm/bitsperlong.h avoid further potential userspace pollution
```