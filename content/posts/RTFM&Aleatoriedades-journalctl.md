---
title: "RTFM & Aleatoriedades para SysAdmin - journalctl "
date: 2024-01-05T17:43:55-03:00
layout: "simple"
# weight: 1
# aliases: ["/first"]
tags: ["SystemD","Linux"]
author: "Faioli a.k.a 0xttfx"
toc: true
hideSummary: false
comments: true
#showComments: true
showDate: true
showTitle: true
showShare: true
#disableShare: false
norss: false
nosearch: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
#UseHugoToc: true
---

 Para quando a lei de Murphy exerce a sua força. É nessa hora, que do seu kit MacGyver, você tira o **journalctl**  para identificar o que pode estar errado!
 
 Das muitas formas possíveis. Tem mais essa:

```bash
$ journalctl --no-pager --since today --grep 'fail|error|fatal' --output json|jq
```
- Com a opção "--grep" filtramos palavras chave
- Com a opção "--since" melhoramos nossa assertividade. 
	- Ex:
		- --since "1 hour ago"
		- --since "1 minutes ago"
		- --since "5 seconds ago"
		- --since 07:00 --until "1 hour ago"
- Com o "--output" usando o formato *json*, fazemos um parser amigável pra organizarmos com **jq**.

```bash
$ journalctl --no-pager --since today --grep 'fail|error|fatal' --output json|jq

{
  "_PID": "5605",
  "_SYSTEMD_SLICE": "user-1000.slice",
  "_SYSTEMD_USER_UNIT": "org.gnome.Shell@wayland.service",
  "_RUNTIME_SCOPE": "system",
  "_SYSTEMD_UNIT": "user@1000.service",
  "__SEQNUM": "7540847",
  ...
  ...
  "_SYSTEMD_OWNER_UID": "1000",
  "SYSLOG_IDENTIFIER": "google-chrome.desktop",
  "_COMM": "cat",
  "__MONOTONIC_TIMESTAMP": "285995139402",
  "_CMDLINE": "cat",
  "_BOOT_ID": "da45ccab9c404e9444444a9c51300cae",
  "_EXE": "/usr/bin/cat"
}
{
...
...
```

Ainda é possível filtrar por algum objeto com o *jq* e quantificar as ocorrências com sort e uniq
- Talvez, alguns objetos interessantes seriam _EXE, _CMDLINE, _PID, SYSLOG_IDENTIFIER, MESSAGE...    
```bash
$ journalctl --no-pager --since "60 minutes ago" --grep 'fail|error|fatal' --output json|jq '.SYSLOG_IDENTIFIER' | sort | uniq -c
      1 "audit"
      1 "cupsd"
     19 "discord.desktop"
      2 "fprintd"
      7 "gnome-shell"
  10518 "google-chrome.desktop"
     18 null
      4 "org.gnome.Software.desktop"
    173 "slack.desktop"
``` 

---
<script src="https://giscus.app/client.js"
        data-repo="0xttfx/0xttfx.github.io"
        data-repo-id="R_kgDOK3wAHw"
        data-category="BlogPostComments"
        data-category-id="DIC_kwDOK3wAH84Cnmtb"
        data-mapping="pathname"
        data-strict="1"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="top"
        data-theme="preferred_color_scheme"
        data-lang="en"
        data-loading="lazy"
        crossorigin="anonymous"
        async>
</script>

