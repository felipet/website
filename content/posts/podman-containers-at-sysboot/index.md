+++
title = "How To Start Containers with Podman At System's Boot"
date = "2025-03-28T12:29:13+01:00"
author = "Felipe Torres González"
authorTwitter = "feliptg"
cover = ""
coverCaption = ""
tags = ["hugo", "deployment", "devops", "podman"]
keywords = ["devops"]
description = "A short guide about how to run containers using Podman at system's boot."
showFullContent = false
readingTime = true
hideComments = false
toc = true

+++

## Introduction

Let's continue with the series of blog posts about moving from **Docker** to **Podman**. 

Today I'll cover a short issue which might help you with your transition:

**Why `--restart unless-stopped` doesn't work??**

Let's say you read that **Podman**'s command line API is 1:1 compatible with **Docker**'s  and you ran your container as always, but you realised that it didn't boot up automatically after a reboot of the system as it was happening when using **Docker**. Well, this was something to be expected as we moved from a container manager that runs as a daemon  to another that runs as a regular process.

### **Docker** vs **Podman** at System's Boot

When our system boots up, we have a **systemd** service that handles starting **Docker**. Then, **Docker** itself is in charge of starting all those containers marked to be started automatically. Kind of magic to us, as we only need to specify the argument `--restart unless-stopped` when running a container.

But **Podman** doesn't work that way. There's no service running **Podman** in background. A disadvantage? I don't think so. Why to place **Docker**'s  service in the middle if we can write our own **systemd** service to trigger our target container. That would be cleaner, wouldn't it?

## Our New Best Friend: **systemd**

**Podman** doesn't reinvent the wheel, there's an excellent piece of SW that already handles services in our OS: **systemd**. So instead of duplicating features, it relies on **systemd** to handle the life-cycle of our containers.

If you're not very familiar with **systemd**, I'd suggest you to have a look at this great book: [Linux Service Management Made Easy with systemd by _Donald Al Tevault_](https://www.oreilly.com/library/view/linux-service-management/9781801811644/).

This section covers how to achieve the same feature we had when using `--restart unless-stopped` while running our containers.

### Understanding The New Deployment Schema

This diagram from the book Podman in Action by _Daniel J. Walsh_ depicts how all the pieces will be put together:

![Diagram from the book Podman in Action](/img/diagram_from_podman_in_action.png)

We'll need to define a **systemd** unit file that describes how do we expect to run our container. Then **systemd** will use it to trigger our container when required. What if you don't know how to write a unit file? No worries, as **Podman** devs have been really kind providing a tool that generates such file on behalf of us. But a brief understanding of unit files would be good anyway. Remember that you have `man systemd.unit` at hand if you need to clarify any option from the future unit file.

Let's get our hands dirty and make an example of it.

## Booting a Container at Boot

For the sake of the tutorial, I'll use one image of my own: [La Coctelera Backend](https://github.com/felipet/lacoctelera_backend/pkgs/container/lacoctelera_backend), this image is the one that I use to deploy the backend service of [_La Coctelera_](https://felipe.nubecita.eu/projects/lacoctelera/).

First, we need either to create a container or use a running one. This will be used by **Podman** to detect what settings are needed when the unit file gets made.

```bash
$ podman create \
	--restart unless-stopped \
	--network host \
	--env-file /home/user/.local/share/coctelera/coctelera.env \
	-v coctelera:/app/config \
	ghcr.io/felipet/lacoctelera_backend:main
```

Remember to include the restart policy!

Now, we can use `podman generate systemd` to generate a **systemd** unit file. I usually don't provide names to the containers as I enjoy using the automatic funny names that are generated. If you are like me, you'll need to get the name that was assigned to your container (`podman ps -a`).

```bash
podman generate systemd --new amazing_feistel > ~/.config/systemd/user/lacoctelera_backend.service
```

I intend to run my service as a regular user, that's why I place the unit file in the user's local storage. Let's take a look at the file that was generated:

```bash
[Unit]
Description=La Coctelera backend service
Wants=network-online.target
After=network-online.target
RequiresMountsFor=%t/containers

[Service]
Environment=PODMAN_SYSTEMD_UNIT=%n
Restart=always
TimeoutStopSec=70
ExecStart=/usr/bin/podman run \
        --cidfile=%t/%n.ctr-id \
        --cgroups=no-conmon \
        --rm \
        --sdnotify=conmon \
        -d \
        --network host \
        --env-file /home/user/.local/share/coctelera/coctelera.env \
        -v coctelera:/app/config ghcr.io/felipet/lacoctelera_backend:main
ExecStop=/usr/bin/podman stop \
        --ignore -t 10 \
        --cidfile=%t/%n.ctr-id
ExecStopPost=/usr/bin/podman rm \
        -f \
        --ignore -t 10 \
        --cidfile=%t/%n.ctr-id
KillSignal=SIGINT
Type=notify
NotifyAccess=all

[Install]
WantedBy=default.target
```

I only made a few minor changes to it: I modified the description, and added `KillSignal` as my application expects `Ctrl + c` as the signal to stop the service gracefully. Pretty painless, wasn't it?

We only need a few more shell commands before we call it a day. We can test running our container using **systemctl**:

```bash
$ systemctl --user start lacoctelera_backend.service
$ systemctl --user status lacoctelera_backend.service
```

If everything went fine, we shall see that our service is `active (running)`. Now, it's time to enable our service at boot using this command:

```bash
$ systemctl --user enable lacoctelera_backend.service
```

And that's it! Our service will be trigger at boot the next time.

## Final Thoughts

At this point you might think: such a hassle, I have to write a unit file and learn about how to use **systemd** while I only needed a simple option when I was using **Docker**. And this is sort of true. But the bottom line is that we can benefit from the other interesting features that are provided by **systemd**. 

The combination of **systemd** + **Podman** leads to a more secure and reliable deployment. We'll delve deeper into the security options that we can use from now in future posts.

Also, learning how to use **systemd** is never a waste of time. We can use this knowledge for many other things, as **systemd** is a core component of our OS.

And a final remark, did you like how **Docker** handled logs? I personally don't. The command `docker logs <container>` simply sucks. Then you have to write your logs to a file, and place it in a volume for example, if you aim to access them easily from the host. OK, but what happens when that log grows? And it will, believe me.

By using **systemd** we can make use of **journald** to handle our log messages. One of the greatest advantages is that **journald** makes an automatic log rotation of our logs, and it also keeps logs in binary which saves space. And stores our logs independently of our container, so even if our container is down, we can check the logs. That's a nice feature.

That's all for today, stay tuned for more posts of the series!
