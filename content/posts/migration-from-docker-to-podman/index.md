+++
title = "How To Migrate From Docker To Podman"
date = "2025-03-13T09:22:45+01:00"
author = "Felipe Torres González"
authorTwitter = "feliptg"
cover = ""
coverCaption = ""
tags = ["hugo", "deployment", "devops", "podman"]
keywords = ["devops"]
description = "A painless guide about how to migrate from a deployment based on Docker to Podman."
showFullContent = false
readingTime = true
hideComments = false
toc = true
+++

## Introduction

In this blog post, I'll explain the steps that I followed to migrate from **Docker** to **Podman**. Whether you are a sysadmin or a regular developer, I'm quite sure that you've heard of **Docker** before. It's a great piece of software, and it really helped to improve and simplify software delivering. Believe me, I was there before **Docker** exists, and it was quite chaotic.

Then, why to move away from **Docker**? Well, if you're after better integration with the system, like using **systemd**, or if you're after more security options, or if you prefer software that doesn't use daemons, then **Podman** is definitely worth a try!

**Podman** is an alternative container management tool developed as open source by Red Hat. It was developed after **Docker** and that meant **Podman**'s developers could learn a lot of lessons from it. Moreover, they dared to improve it, specially targeting the weakest point of **Docker**: security. They were very clever: they realised it would be hard to take market share of a well established software, so they made **Podman** fully compatible with **Docker**'s API. To me, this is a key to success because regardless on whether they offer a better piece of software or not, there's a lot of inertia towards **Docker** and reverting it would be almost impossible, unless you make an offer that you can't refuse: use **Podman** as you were using **Docker**. So simple, so brilliant.

I'm not going to get into the nitty-gritty or compare them side-by-side, as there are already some articles out there doing that. I'll just point you to the e-book [Podman in Action](https://developers.redhat.com/e-books/podman-action) by Daniel J. Walsh. As of today, it is free to download, and it is really well written. If you're interested in **Podman**, you should definitely check it out.

Today's blog post focuses on the migration from **Docker** to **Podman**. We'll take advantage of the compatible API, and we'll dare to seamless restart our containers using the same command that we were using with **Docker** with an single change: replace `docker` by `podman`.

Hence, no fancy stuff today, just a straightforward migration. I think it's best to keep it simple, as we're about to touch an important part of your system, and we need to feel confident enough before moving forward. What would be better than proving that we can run **Podman** as we were doing with **Docker**, let's get started!

## Save Your Containers

You might be reluctant to move because you have several containers running, and you can't afford losing the content of those. Many images are prepared to place modifiable content in a particular path, this way you could use the option `--volume` to specify a host folder in which data can outlive the container.

But, what if you forgot to use an external volume? Or, what if your particular image keeps many modifications within the running container? If you're in this situation, don't panic! There's a simple procedure to save all the content, and prepare the containers to be loaded into **Podman**.

### How To Save a Running Container

First, we'll get the hash identifier or the name of the containers that we aim to migrate. If there are stopped containers, we'll need `docker ps --all`. 

We'll use [`docker commit`](https://docs.docker.com/reference/cli/docker/container/commit/) for the purpose of saving a running container into an image that we can later import into **Podman**. The process is simple, just use:

```bash
$ docker commit <container id or name> <image name>
```

If the container is running, that command will pause it for a while, so keep that in mind in case you can't interrupt your underlying service.

Once we've made an image of a container, we need to export it this way:

```bash
$ docker save <image name> | gzip > <image name>.tar.gz
```

Repeat that process for each container that you need to migrate, and you'll be ready to start using **Podman**.

***Tip:*** *If you were running docker as superuser, the images that you'll export will be owned by `root`. As we'll run **Podman** as a regular user, remember to change the permissions of those images before proceeding with the following steps.*

## Import Your Saved Images Into Podman

At this stage, we're ready to start running our containers using **Podman** instead of **Docker**. Just a heads-up: you might want to stop your running containers under **Docker** now, otherwise ports being used by your running containers in **Docker** would avoid running your containers with **Podman** using exactly the same configuration.

First, we'll import all the images that from the previous step using:

```bash
$ podman load < <image name>.tar.gz
```

When you are done, go and check everything went OK using `podman image ls`. You shall see your images listed. Now, it's time to fire up a few containers using those images. One of the greatest things of **Podman** is that its command line API is 100% compatible with **Docker**'s. So take the command that you used before to run your containers with **Docker**: `docker run [...]`, and replace `docker` by `podman`, that's it!

## Final Steps

There's not much else to do. We saved all our beloved **Docker** containers and imported them into **Podman**. We made a big jump but we had a great safety net. Something didn't go as expected? Just start your stopped container in **Docker** and you lost nothing but time. However, I hope everything ran smoothly and you faced no big issues during the process.

Keeping **Docker** won't harm you, so don't be forced to clean it up before you feel comfortable using **Podman**. But if you reached this point, you are more than ready to free some space and remove **Docker** from your system.

So far, we made no use of any special feature, as we are using **Podman** as a 1:1 replacement for **Docker**. However, **Podman** offers many interesting features to improve security. I'll cover some interesting features that **Podman** includes in following blog posts, so keep an eye on my blog!
