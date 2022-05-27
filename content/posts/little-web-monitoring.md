---
title: "Little Web Monitoring"
date: 2017-08-02T08:22:16Z
tags: [script, python, monitoring]
type: "post"
draft: false
---

Sometimes making small `OneJob` services is efficient in terms of time, organization, functionality.
In the example of this article, I will try to show how it can be relevant to write `microscripts' under this model.

Let's say we want to make sure, for a given list of sites, that they all return `200` codes in less than `2s`, in which case we'll get a notification on `Slack`.
The sources for these posts are available [here](https://github.com/Kaderovski/Slack-Bot-Web-Alert/blob/master/bot.py).

For the language, `python3` will be more than enough.

```
#!/bin/python3.6

import requests
import time
from slackclient import SlackClient

slack_token = "YOUR_TOKEN_HERE"
sc = SlackClient(slack_token)

urls=['site1', 'site2', 'site3']

while True :

    for i in urls :
        r = requests.head(i)
        t = requests.get(i, timeout=5).elapsed.total_seconds()
        # print(i, r.status_code, t)
        time.sleep(2)

        if r.status_code != 200 :
            sc.api_call(
            "chat.postMessage",
            channel="#infra",
            text=("ALERTE !", i, r.status_code)
            )

        if t > 2 :
            sc.api_call(
            "chat.postMessage",
            channel="#infra",
            text=("LoadSite too long : ", i, r.status_code)
            )

    time.sleep(10)
```

Let’s checkout what we have here :

We are using : 

* `requests` to be able to call sites/url
* `time` to pause code.
* `SlackClient` to be notified.

In the end, with only 30-35 lines of code, we are able to ensure **quickly** the proper functioning of a list of sites.
It takes less than 10 minutes to code, it's "fast", "light" and "portable" everywhere.

The advantage of this type of scripts is that they are very modular, adding features is just as fast as developing the code and you make sure you only get the information you need, which is a real advantage in terms of performance. 
