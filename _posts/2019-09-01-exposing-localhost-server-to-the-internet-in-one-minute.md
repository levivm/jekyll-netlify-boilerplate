---
description: Learn how to use ngrok to expose your local server to the internet.
layout: post
author: levi_velazquez
date: '2019-09-01 13:35:52'
intro_paragraph: |+
  Lean how to use ngrok to expose your local server to the internet.

published: true
tags:
  - webdev
  - beginners
  - testing
  - devtips
title: Exposing localhost server to the internet in one minute
---


Sometimes, we found a situation due to multiple reasons, where we need to expose our local server to internet, maybe we need to show our website to a client and doesn't have enough time for deploying, others, we need to test it out a webhook from a third service and we don't want to create a new deployment just for testing purpose.


So, let's go for it.

## ngrok

 There services that offer you to set up a tunnel between their servers and your local machine. There are several of them, nevertheless, my recommendation is `ngrok`.


 It has a freemium model, it fits the most basic requirements.

### How it works

We download and run a program(__ngrok__), specifying which port is our local server listening to. __ngrok__ will connect to their external cloud service, creating a public endpoint redirecting all the traffic from it to your server. That's it, simple.

### Download and setup


1. Download it from [official site](https://ngrok.com/download)
2. Unzip `/path/to/ngrok.zip` into any folder
3. Start an HTTP tunnel in any port you want. Let's do it on 80

```bash
$ ./ngrok http 80 # this command should be run from where ngrok was unzipped
```

```bash
...........
Web Interface                 http://127.0.0.1:4040
Forwarding                    http://82330e5b.ngrok.io -> localhost:80
Forwarding                    https://82330e5b.ngrok.io -> localhost:80
...........

```

That's it. We just need to run our web server in port 80 and starting to receive traffic.

If you want to test it out. You may send a request using curl to our public ngrok domain, `http://82330e5b.ngrok.io` or just open it in a browser. Additionally, ngrok allows `https` connections, this is a great feature when we are testing external services needing a secure connection. 

Note: remember you need to have your local server running on port 80 (for this example).

```bash
$ curl -v http://82330e5b.ngrok.io
```


You can check all requests details using built-in dashboard for ngrok at `http://localhost:4040`

![Dashboard](https://thepracticaldev.s3.amazonaws.com/i/iesb6jqsmab4q5z1sor4.png)


That's all. For some people, it could look like a trivial task, but for beginners, this is a starting point.

If you like it, spread the word.

Disclosure: This is for testing purpose, if you are handling sensitive data, avoid using this method because all your traffic is going to pass through a third service.
