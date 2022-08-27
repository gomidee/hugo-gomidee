---
title: "How to Self-Host Vikunja"
date: 2022-04-25
tags: [Self-hosting]
---

Vikunja is a To-Do app licensed under AGPL-3.0, it's a great little project. I decided to self-host my own instance and here's how I did it.

I don't overly enjoy using Docker but I made an exception for Vikunja as it's extremely easy to set it up. Please note that I'm running **Ubuntu 20.04 LTS**, you can use a different distro instead. Now it's time for you to get your hands dirty!

I use Tailscale in order to connect my devices, it also helps to further harden my setup (you could use WireGuard instead, it should work just fine). Here's how I like to do it.

## Here's what you need

- A virtual private server (VPS): you can get the cheapest option, it will work just fine. I use [1984 Hosting](https://1984.hosting) as my hosting provider, but you can use another one if you want to.
- A domain name (optional)
- Tailscale (optional)

### Gettin' down to business
#### Hardening your server

Please refer to my hardening [guide](https://github.com/gomidee/Hardening), you'll need to follow some steps manually, after that you can run the script:

```
wget https://raw.githubusercontent.com/gomidee/Hardening/main/palmtree.sh
chmod +x palmtree.sh
sudo ./palmtree.sh
```
**Please enable port 22 (or your chosen ssh port) when prompted to select firewall ports, otherwise you'll be locked out of ssh!</strong>**

#### Hardening docker

Docker doesn't respect iptable rules by default, add the following line to **/etc/docker/daemon.json** and **/etc/default/docker**, respectively.

```
{ "iptables": false }
```
```
DOCKER_OPTS="--iptables=false"
```

### Setup Tailscale (optional)

When running the Palmtree script you were promted to install Tailscale, if you didn't and would like to use it on your setup you can run the following command:

```
curl -fsSL https://tailscale.com/install.sh | sh
```
Cool! Now you're good to create the **docker-compose.yml** file.

### Setting up Vikunja and Nginx

First, install nginx, we'll need it soon.

```
sudo apt-get update
sudo apt-get install nginx
```

Let's create a directory to keep things organised

```
mkdir vikunja && cd vikunja
```
Now download the docker-compose and the nginx template by running:

```
wget https://gist.githubusercontent.com/dnburgess/5c93209089ee80c13e2834664a4267dc/raw/5ed387bdb92bd153311ed5596c428a44ac2fe7e6/gistfile1.txt
wget https://gist.githubusercontent.com/dnburgess/5e09f5de59d5f0aa5ac908dfc2dadaca/raw/67aa9bb60f4254fb2e4e2b53c785b1bf70ce356f/gistfile1.txt
```

Go ahead and edit the docker-compose file and make any changes that you wish:

- Ports
- Front end url
- Email
- Password

After you finished editing the file, we can deploy the container! *YEEEWWW*

First enable the port that you chose
```
docker-compose up -d
```
We use the -d argument so that docker runs in detached mode... so it runs on the background.

Disable UFW, type your server's IP and the port... Like this: 20.43.111.112:8022
You could also type the machine's Tailscale address: 100.69.20.149.50:8022

You should see Vikunja!

### Hardening with Tailscale and setting up your domain name

Enable ufw again... this is the part were we harden our docker instance and create a reverse proxy using nginx. I'll explain how this will work and what the strategy is. I'll use images to illustrate this:

![tailscale](/images/tailscale.jpg)


You can see in the image above a bunch of IPs, these are the internal IPs, much like your home LAN. This means they are only accessible on the Tailscale network.. **cool right?**

Tailscale is a different interface so if you type <strong>ip addr</strong> on your Linux server you should be able to see an interfaced named **tailscale0**. Like this:

![](/images/example.jpg)

Our goal is to make port 8022 unreachable from the outside but reachable from inside our network. We can do this by running:
```
ufw allow in on tailscale0 to any port 8022 proto tcp
```

If you type **ufw status** you should see something like this:

![](/images/ports.jpg)

Awesome! Now we need this *thing* to be accessible in through our domain. Now... setup a subdomain so you can access vikunja. I chose vik, so I can access it at **vik.gomidee.com**

Tip: If you're new to all of this, go to [landchad.net](https://landchad.net). It will make much more sense.

After setting up the subdomain record you'll need to use nginx to proxy whatever is happening in our internal network accessible on the outside network. This is the reverse proxy.

Let's open our doors to the outside world!
```
ufw allow 80
ufw allow 443
```
### Let's show them what we got!
I use VIM as my text editor you can use nano or whatever. Do the following...
```
vim /etc/nginx/sites-enabled/vik
```
Copy and paste the following:
```
server {

         server_name vik.yourawesomedomain.com;

	location / {

	      proxy_pass http://TAILSCALEIP:8022;

	      }
}
```

Change the domain and the Tailscale IP (if you used a different port change 8022 to whatever port you're using)

**Last thing! CERTBOT**, let's encrypt this connection!
```
apt install python3-certbot-nginx
certbot --nginx
```
*BANG!* Go visit your Vikunja instance!

If you have any questions you can reach out to me on my [contact](https://gomidee.com/contact) page.

---
Tags:
