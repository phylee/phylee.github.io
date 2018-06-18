---
title: DNF将取代YUM
date: 2017-05-03 23:57:01
tags:
- Linux
---

```text
The reason of initiating DNF project was because of
the biggest three pitfalls of Yum: undocumented API,
broken dependency solving algorithm and inability to
refactor internal functions. The last mentioned issue
is connected with the lack of documentation. Yum plugins
are using any method from Yum code base thus any change
there would cause the sudden crash of the Yum utility.
The DNF aim was to avoid mistakes made in Yum. From the
start all exposed API functions were properly documented.
The tests were included with almost every new commit. No quick
and dirty hacks are allowed. The project is directed by agile
development – the features that have the greatest impact
on users are operatively implemented with higher priority.
```

[原文链接](http://dnf.baseurl.org/2015/05/11/yum-is-dead-long-live-dnf/)