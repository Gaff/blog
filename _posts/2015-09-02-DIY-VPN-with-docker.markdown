---
layout: post
title:  "Become your own VPN provider in 15 minutes with Docker"
date:   2015-09-02 13:44:47
categories: VPN docker
---

It's VERY easy to setup your own VPN with docker. This guide assumes you're ok with operating a linux box but know nothing about docker. I'll break it down into 4 easy steps:

## 1. Setup a docker host

I'm using bithost, so the steps here are:

* Sign up and pay them some bitcoins :)
* Generate a pgp key (digitalocean have detailed instructions on this step)
* Upload the key
* Launch a new host - I used ubuntu 14.04 image, but any platform that support docker will work.
* Install docker on the host - For ubuntu this is as simple as logging in and running ```curl -sSL https://get.docker.com/ | sh```

## 2. Run the docker image

Here is where the magic happens. Simply run this:

    docker run -d --cap-add=NET_ADMIN -p 1194:1194/udp -p 443:443/tcp jpetazzo/dockvpn

Docker will take about a minute to do its magic and then return you to the command line. To see that it is working do this:

    # docker ps -a
    CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS                                                    NAMES
    29fdb2af5932        jpetazzo/dockvpn    "/bin/sh -c run"    About a minute ago   Up About a minute   0.0.0.0:1194->1194/udp, 8080/tcp, 0.0.0.0:443->443/tcp   sharp_mestorf

You can see the docker image running. Docker has assigned me the name "sharp_mestorf" for this image. You can think of docker processes as services, they persist forever, but you can use ```docker stop``` and ```docker start``` to control them. Use ```docker rm``` to completely kill a stopped process. 


## 3. configure openVPN

Before we can connect to this VPN we first need the config file. To do this we need to temporarily setup a web server on our docker box to give us the config file. This is built into the docker image so we don't have to think about it:

    docker run --rm -t -i -p 8080:8080 --volumes-from sharp_mestorf jpetazzo/dockvpn serveconfig

Note that "sharp_mestorf" should be replaced by whatever name docker gave your container!

This time I'm running docker in interactive mode, and I've added "--rm" so the container is destroyed as soon as we stop the job. I onlu need to run the server for long enough to download the config file. After that I want to stop the server so that nobody else can get the config.

To fetch the config open up a webbrowser and connect to your docker box's IP on port 8080. You'll likely get some security errors because this is being served over HTTPS without the correct certificates. It's safe to ignore these errors. You probably want to save it as "myvpn.ovpn" or similar.

If you want to be a boss use wget :)

    wget --no-check-certificate https://188.226.132.215:8080 -o myvpn.ovpn

After you download the file you can stop the docker server process with control+C

## 4. Connect to your VPN

You need to tell your openvpn client to use the config file you just downloaded. On windows this means placing the config file in the appropriate directory. On linux just run:

```
sudo openvpn myvpn.ovpn
```

Congratulations! You now have your own personal VPN!!

## Foot notes


A prudent user should also be concerned about security. The security of this setup is pretty tight since only the holder of the SSH key can connect to your docker box, and only the holder of the vpn config can connect to your vpn. Still there are some notes on the key setup here: (https://hub.docker.com/r/jpetazzo/dockvpn/). It's also worth noting that the server does keep some logs inside the docker image in /etc/openvpn - limited to information about who is connecting to the VPN, but still. It should be easy to tone down the logging with a little tweaking.

Another thing to note is that this is a slightly hacky setup that's fine for a single user. If you wanted to support multiple users then you should check out [this docker image](https://github.com/kylemanna/docker-openvpn). It's setup to allow you to generate a separate key for each user etc etc but its slightly more complex to use. 

A nice extension project would be to create similar docker images that also include a proxy, or allow you to chain VPNs. Perhaps if I'll give it a go at some point.

If you found this article helpful please let me know :)

