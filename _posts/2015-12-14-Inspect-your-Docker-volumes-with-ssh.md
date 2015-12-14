---
layout: post
title:  "Inspect your Docker volumes with ssh"
date:   2015-12-14 13:44:47
categories: Docker sshd alpine
---

## Want to know what's in your Docker volumes?

Ever wanted to peek inside your docker volumes to see what's going on? If your docker machine is local this is pretty simple of course, but if it's remote it become harder!

What you can do is setup a sshd server that shares the volumes. This should be simple, but often requires *another* volume with the SSH key, which is a pain to setup.

To make life simpler I've created an image that allows you to set the ssh key from the envornment, this making it dead simple to run:

    docker run --rm -it -p 2222:22 --volumes-from my_other_image -e AUTHORIZED_KEY="`cat id_rsa.pub`" gaff/alpine-sshd

Now you should be able to ssh in and look at your volumes. Simple :)

Let me know if this has been useful :)
