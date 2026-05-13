+++
date = '2025-01-10T14:07:23+01:00'
update = '2025-01-15T10:37:00+01:00'
title = 'Automated Website Deployment'
author = "Felipe Torres González"
description = "This blog post is a recap of all the steps needed to set up a deployment pipeline for a Hugo-based website hosted in GitHub."
categories = ['nubecita']
tags = ['hugo', 'apache', 'reverse-proxy', 'deployment', 'github', 'webhook']
toc = true

+++

## Overview

In this entry, I'll explain how to achieve an automated deployment after a change in a website built with Hugo, and deployed on Ubuntu Server with Apache thanks to [Webhook](https://github.com/adnanh/webhook).

There are many hosting services in which a page built with Hugo can be deployed for free. This is already covered by the [official docs](https://gohugo.io/hosting-and-deployment/). Using an external hosting service is the way to go for most people, but what if you already have a server?

My plan was to host my new personal site on my server: [Nubecita](https://nubecita.eu). It runs [Ubuntu Server 24.04](https://ubuntu.com/download/server) and Apache as web server. One of the most common combinations for hosting stuff on the Internet. Aside from that, I push my website to a repository hosted in [GitHub](https://github.com/felipet/personal-site). My goal was to automate the process of deployment, so whenever I push a new change to the repository, it gets deployed into the production web server.

This guide is tailored for such a setup. I hope someone will find it useful.

## First Things First

Before we start configuring services and writing scripts, we'll set up our system. We'll need to install Webhook. I'll opt for the packaged version from the repositories, rather than the one from the Snap Store:

```bash
sudo apt install webhook
```

**Update**: You'll need Webhook >= 2.8.2, as that version includes all the changes to run under Systemd.

Also, I wouldn't recommend using Hugo from the Snap Store, as it would restrict you to having your deployment scripts in a system directory rather than your home directory. But that's up to you, so bear in mind that you can't use Hugo from the Snap Store to follow this guide.

## Write a Deployment Script

Once we've got our system ready, we can start with the interesting part. We're going to need a deployment script, which is nothing more that a script that automates all the steps that you'd do if you were doing a manual deployment. For the sake of this guide, I'll include the script that I use, but you'll need to modify it to suit your needs.

```bash
#!/bin/bash

echo "Retrieve new content from the repo..."
git clone https://github.com/felipet/personal-site.git --depth 1
cd personal-site
echo "Build the static page"
hugo
cd public
echo "Deploy the new content"
rsync -r ./ /var/www/html/felipe-site
chown -R www-data:www-data /var/www/html/felipe-site
systemctl reload apache2
echo "Wiping temporal data..."
cd ../../
rm -rf personal-site
```

Remarkable items from the script:

- Add `--depth 1` to the clone command, this would save time in case your repository contains a significant amount of images, and other heavy items.
- After synchronising with `rsync`, I change the owner of the new data. The user `www-data` is often used in Apache deployments. Consider whether that user and group are the best option in your deployment.
- I reload the `apache2` service after the deployment, thus this script needs to be run by a privileged user.

Place that script wherever you'd like. I chose the path `/var/scripts` for which I assigned as owner of the directory the `www-data` user & group. Also, give it a try, and make sure it works as expected.

## Set Up Webhook

Our deployment script is ready now, but how do we bring it to life? Here it is where Webhook comes into scene. We'll add a new Systemd service to manage Webhook, and we'll write the configuration file that tells Webhook what hook shall be deployed for our purpose.

### Running Webhook As a Systemd Service

We'll configure Webhook to run as a Systemd service. This is well explained in [the official documentation](https://github.com/adnanh/webhook/blob/master/docs/Systemd-Activation.md), so I'll just mention the minor modifications that I made.

In brief, this is the socket descriptor:

```bash
[Unit]
Description=Webhook server socket

[Socket]
## Listen on one specific interface only
ListenStream=127.0.0.1:9191

[Install]
WantedBy=multi-user.target
```

As you can see, I wrote that the socket will only listen to requests coming from the local address. I'm planning to use a reverse proxy, which adds an extra layer of security because I don't want to expose it to the public. If you're planning to skip the reverse proxy, you'll have to listen to `0.0.0.0:9191` but remember that you'll need to set up some extra security layer.

And the service descriptor:

```bash
[Unit]
Description=Webhook server

[Service]
Type=exec
ExecStart=/usr/bin/webhook -nopanic -hooks /var/webhooks/hooks.json -verbose -port 9191
StandardOutput=append:/var/log/webhook
StandardError=append:/var/log/webhook
User=root
Group=root
```

**Very important:** the listening port and the port that is passed as an argument to Webhook are the same. It might seem like basic advice, but when you don't use the standard ports, you might accidentally miss a port specification somewhere, which can waste time.

As I said, you need to run this service as `root` in order to have enough privileges to run `systemctl reload apache2`.

### Set Up a New Webhook

You might have noticed that we passed as argument to Webhook the file `/var/webhooks/hooks.json`. Now, we'll pay some attention to such file.

I chose the path `/var/webhooks` but you can put it anywhere you like. I also gave `www-data` permissions to read and write on that path, as I aim to deploy my repository right there every time the service is called.

This is the content of the file `hooks.json`:

```json
[
    {
        "id": "deploy_felipe_site",
        "execute-command": "/var/scripts/deploy_felipe-site.bash",
        "command-working-directory": "/var/webhooks",
        "trigger-rule":
        {
            "match":
            {
                "type": "payload-hmac-sha256",
                "secret": "SOME_SECRET_STRING!!!",
                "parameter":
                {
                    "source": "header",
                    "name": "X-Hub-Signature-256"
                }
            }
        }
    }
]
```

The value for `execute-command` shall be the path to your deployment script. The values for the `match` field are explained in the official [GitHub's](https://docs.github.com/en/webhooks/webhook-events-and-payloads#delivery-headers) documentation. Don't forget to replace the value for `secret`. Generate some random strong string and put it there. Keep it for a while, as you'll need to introduce the secret string while configuring our Webhook in GitHub.

### Final Steps

Ready to go? Then run these commands:

```bash
sudo systemctl enable webhook.socket
sudo systemctl start webhook.socket
```

And check that everything is working fine:

```bash
systemctl status webhook.socket
```

You should get something alike to this output:

```bash
● webhook.socket - Webhook server socket
     Loaded: loaded (/etc/systemd/system/webhook.socket; enabled; preset: enabled)
     Active: active (listening) since Tue 2025-01-14 10:39:37 UTC; 6s ago
   Triggers: ● webhook.service
     Listen: 127.0.0.1:9191 (Stream)
      Tasks: 0 (limit: 37781)
     Memory: 8.0K (peak: 256.0K)
        CPU: 820us
     CGroup: /system.slice/webhook.socket

Jan 14 10:39:37 nubecita.eu systemd[1]: Listening on webhook.socket - Webhook server socket.
```

## Using Apache As Reverse Proxy

Here, you might diverge from the path I followed. If you are not fond of using a reverse proxy, go straight to open a port in your firewall and offer your service straight to the the public. In that case, I'd recommend you to follow Webhook's docs to enable HTTPS.

I use Apache to serve several websites, and as a reverse proxy for some web services that I offer. For me, it was an obvious choice. I avoid opening another port in my firewall, and the SSL stuff is managed by Apache, a _win-win_.

To forward requests to Webhook's socket, add a new virtual host using the `ProxyPass` command:

```
ProxyPass /webhook http://127.0.0.1:9191/hooks
ProxyPassReverse /webhook http://127.0.0.1:9191/hooks
```

This will redirect a request to `https://nubecita.eu/webhook/deploy_felipe_site` to the local Webhook socket. If you made a new virtual host, enable it. In any case, reload the Apache service:

```bash
sudo systemctl reload apache2
```

Our backend is ready at this point, let's make a simple request to check that it responds, fingers crossed!

We can use Curl to send a POST request to the endpoint in which we installed our webhook:

```bash
curl -X 'POST' \
  'https://nubecita.eu/webhook/deploy_felipe_site'
Hook rules were not satisfied.%
```

As we didn't send our credentials nor any further information along the request, our service tells us that we didn't satisfy the rules that we defined in `hooks.json`. That's great, our backend responds, and blocks dummy requests. Let's complete our deployment process with the last step.

## Create a Webhook in GitHub

We reached the last needed step. We need to configure a webhook in our repository in GitHub.

The process is explained in this [page](https://docs.github.com/en/webhooks/using-webhooks/creating-webhooks). However, let's summarise the steps:

1. Go the settings section of the repository that keeps your web site content.
2. Open Webhooks from the left panel, and click on _Add webhook_.
3. Input the URL of your endpoint in the _Payload URL_, and choose _application/json_ as _Content type_.
4. Do you remember the secret string? Now it's time to use it again. Add it in the _Secret_ box.
5. Of course, set _Enable SSL verification_.
6. And finally, choose when do you aim to run the webhook. I selected _Pushes_ and _Releases_.

Save it and navigate to the _Recent deliveries_ tab. You shall see that a ping job was launched. If that succeeded, congrats! Your deployment process is ready.

Make any change to the content of the repository, and push the changes. You shall see that the changes get deployed in your production web site.
