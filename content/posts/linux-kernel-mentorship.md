---
title: "Linux Kernel checkpatch.pl mentorship"
date: 2021-03-06T02:05:17+05:30
draft: false
---

## Checkpatch - What is it?
[checkpatch.pl](https://github.com/torvalds/linux/blob/master/scripts/checkpatch.pl)
is a trivial style checker for patches sent to the linux kernel.

Why is it needed you say?</br>
I did an analysis on checkpatch warnings on about 50k commits from v5.4.</br>
(Find the full report {{% link "checkpatch_out_50k.txt" %}}here{{% /link %}})

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

As we can see, the number of stylistically incorrect bits still continue
to exist and are still merged into the kernel code. To make the kernel
code more stylistically coherent, we can:

> - Create cleanup patches for the existing code
> - Make sure new patches are stylistically consistent

Both of these are where checkpatch comes handy.

## Checkpatch in the patch submission process

Before sending out the patches to the mailing list, it is recommended to
run checkpatch over the patches.

In general, a Kernel developer would:

> - Create the patch with his proposed changes.
> - Build check the patch and lookout for compiler warnings.
> - Run additional tools such as sparse, KASAN, etc.
> - Run checkpatch.pl over the patches to eliminate all trivial style violations.
> - Send out the patch to the maintainers and mailing list.


It is to be noted that checkpatch is not always right. And at times the
author's judgement should take precedence over checkpatch.

## Drawbacks of checkpatch

Checkpatch is only a static trivial style checker. It does not have the
capabilities of a tool using a full-fledged C parser, such as
[clang-format](https://www.kernel.org/doc/html/latest/process/clang-format.html)
. In this sense it is limited in what it can do.

Still, it is highly valuable as a style checker tool and can ease the work of maintainers and increase the chances of the patch being accepted by the community.

## What did we do to improve it?

Over the first three months of my mentorship period, I primarily focused on
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

During the last three months I worked on two major features:

> - Addition of a verbose mode to checkpatch
> - Addition of an external documentation to checkpatch

For the documentation we decided to document all checkpatch message types,
their meanings and how to resolve them. This is still a work in progress.
Contributions are definitely welcome.

### Verbose Mode

This was one of the new additions we did to checkpatch.

One can run checkpatch like this:

```/bin/bash
./scripts/checkpatch.pl -v test.patch
```

All verbose descriptions can be seen by pairing the verbose option
with --list-types:

```/bin/bash
./scripts/checkpatch.pl --list-types -v
```

### Checkpatch Documentation

The checkpatch documentation is at `Documentation/dev-tools/checkpatch.rst`
in the kernel repository.

To build the documentation make sure you have configured sphynx
either in the system or in a python virtualenv. Next the kernel
doc can be built using:

```/bin/bash
make htmldocs
```

## How to contribute to the documentation?

The documentation is still a work in progress. Feel free to send
the patches to the kernel-doc mailing list, and add me
(Dwaipayan Ray <dwaipayanray1@gmail.com>) and Lukas (Lukas Bulwahn <lukas.bulwahn@gmail.com>)
for reviewing it.

The documentation is written in rst syntax. Refer
[here](http://openalea.gforge.inria.fr/doc/openalea/doc/_build/html/source/sphinx/rest_syntax.html) for a quick guide.

Before sending out the patch, please always:

> - checkpatch.pl your changes
> - build check your patch (in this case check the generated html docs)
> - spell check your patch (this can be easily done through checkpatch
>   using --codespell flag if you already have the codespell dictionary)

## Acknowledgements

During my mentorship period I learned a lot from my mentors. 
I would like to thank Lukas for his constant support as my mentor,
Joe for his guidance with checkpatch and everyone in the community for being
such amazing people.

The kernel contribution process may look old but it works. At the beginning
when I started sending my initial patches, it looked like a tiring and
unnecessary process. I was more accustomed to creating pull requests rather
than sending patches over mail.

Slowly I found out that scripts like get_maintainer.pl actually lets you easily
find the people associated with that particular code. After finding those
people it's very easy to send the patches to the list and the associated
people. This follows a period of extensive review which is very useful for
having good code. I have got lots of reviews from people in the community
and those patches are successfully merged into the mainline.

So I was really amazed by the process. What is even more amazing is that the
newcomers won't get lost in this extensive process. There are always
people who help and encourage others to create good patches. And that
makes me glad to say that I too am a part of this community.
